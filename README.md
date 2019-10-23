# Lab3AVL-RB
"""
Elizabeth Rubio
Lab 3
"""

#This node is used in the AVL tree
class Node:
    def __init__(self,item):
        self.left = None
        self.right = None
        self.item = item
        self.parent = None
        self.height = 0
        
    #To keep track of the height and updated it to make sure that the rules of
    #an AVL are met. This is done by getting the left and the right subtree max
    def tree_updated_height(self):
        left_height = -1
        if self.left is not None:
            left_height = self.left.height
        right_height = -1
        if self.right is not None:
            right_height = self.right.height
        self.height = 1 + max(left_height, right_height)
     
    #Finding the balance by subtracting the left and right tree and the result
    #will determine if its balanced or not
    def tree_balance(self):
        left_height = -1
        if self.left is not None:
            left_height = self.left.height
        right_height = -1
        if self.right is not None:
            right_height = self.right.height
        return left_height - right_height
    
    #This will help the nodes become children when rotating 
    def replace_child(self, node, new_child):
        if self.left == node:
            return self.set_child("left", new_child)
        elif self.right == node:
             return self.set_child("right", new_child)
        return False
        
    #This aids replace_child to change the pointer of the nodes when rotating
    def set_child(self, option, child):
        if option != "left" and option != "right":
            return False
        if option == "left":
            self.left = child
        else:
            self.right = child
        if child is not None:
            child.parent = self
        self.tree_updated_height()
        return True

class AVL:
    def __init__(self):
        self.root = None
    
    #Basic BST insertion, however once the item has been inserted then the rebalance
    #method is called to rebalance the tree  
    def insert(self, node):
        if self.root is None:
            self.root = node
            node.parent = None
        else:
            current = self.root
            while current is not None:
                if node.item < current.item:
                    if current.left is None:
                        current.left = node
                        node.parent = current
                        current = None
                    else:
                        current = current.left
                else:
                    if current.right is None:
                        current.right = node
                        node.parent = current
                        current = None
                    else:
                        current = current.right
            node = node.parent
            while node is not None:
                self.tree_rebalance(node)
                node = node.parent
    
    # if rebalance is necessary then there are the left left, left right, right
    #left, and right right rotations.     
    def tree_rebalance(self, node):
        node.tree_updated_height()
        #rotate left
        if node.tree_balance() == -2:
            #rotate right
            if node.right.tree_balance() == 1:
                self.tree_right_rotation(node.right)
            #rotate left
            return self.tree_left_rotation(node)
        #rotate right
        elif node.tree_balance() == 2:
            #rotate left
            if node.left.tree_balance() == -1:
                self.tree_left_rotation(node.left)
            #rotate right
            return self.tree_right_rotation(node)
        return node
    
    def tree_right_rotation(self, node):
        left_right_child = node.left.right
        if node.parent is not None:
           node.parent.replace_child(node, node.left)
        else:
            self.root = node.left
            self.root.parent = None
        node.left.set_child("right", node)
        node.set_child("left", left_right_child)
        return node.parent
        
    def tree_left_rotation(self, node):
        right_left_child = node.right.left
        if node.parent is not None:
           node.parent.replace_child(node, node.right)
        else:
            self.root = node.right
            self.root.parent = None
        node.right.set_child("left", node)
        node.set_child("right", right_left_child)       
        return node.parent
    
    def search(self, word):
        current = self.root
        while current is not None:
            if word < current.item:
                current = current.left
            elif word > current.item:
                current = current.right 
            else:
                return True
        return False
    
 
#Red-black tree node
class RBNode:
    #color = red
    def __init__(self,item):
        self.color = False
        self.item = item
        self.right = None
        self.left = None
        self.parent = None
    
    #instead of calling the grandparent every time as self.parent.perent this method
    #returns the grandparent
    def rb_grandparent(self):
        if self.parent is None:
            return None
        return self.parent.parent
    
    #after defining grandparent, uncle uses it to define the sibling or the parent
    def rb_uncle(self):
        grandparent = None
        if grandparent is not None:
            return self.rb_grandparent()
        if grandparent is None:
            return None
        if grandparent.left == self.parent:
            return grandparent.right
        return grandparent.left  
    
    #used for rotation    
    def rb_replace_child(self, current, new_child):
        if self.left == current:
            return self.rb_set_child("left", new_child)
        elif self.right == current:
            return self.rb_set_child("right", new_child)
        return False
    
    #just like rb_replaced_child this function is als used for rotations
    def rb_set_child(self, option, child):
        if option != "left" and option != "right":
            return False
        if option == "left":
            self.left = child
        else:
            self.right = child
        if child is not None:
            child.parent = self
        return True
        
    
class RB:
    def __init__(self):
        self.root = None
    
    #The initial colors are established here and then it is balanced
    def rb_insert(self, node):
        self.bst_insert(node)
        #False = red and True = black
        node.color = False
        self.rb_balance(node)
        
    def bst_insert(self, node):
        if self.root is None:
            self.root = node
            node.parent = None
        else:
            current = self.root
            while current is not None:
                if node.item < current.item:
                    if current.left is None:
                        current.left = node
                        node.parent = current
                        current = None
                    else:
                        current = current.left
                else:
                    if current.right is None:
                        current.right = node
                        node.parent = current
                        current = None
                    else:
                        current = current.right
    
    #Like the AVL this function is used to determine if the rules of the Red-black
    #tree are met. The colors are also check, if necessary other functions will
    #be called to set the colors correctly
    def rb_balance(self, node):
        if node.parent is None:
            node.color = True
            return
        #node.parent.color == black
        if node.parent.color == True:
            return
        parent = node.parent
        grandparent = node.rb_grandparent()
        uncle = node.rb_uncle()
        if uncle is not None and uncle.color == False:
            parent.color = uncle.color = True
            grandparent.color = False
            self.rb_balance(grandparent)
            return
        if node == parent.right and parent == grandparent.left:
            self.rb_rotate_left(parent)
            node = parent
            parent = node.parent
        elif node == parent.left and parent == grandparent.right:
            self.rb_rotate_right(parent)
            node = parent
            parent = node.parent
        parent.color = True
        grandparent.color = False
        if node == parent.left:
            self.rb_rotate_right(grandparent)
        else:
            self.rb_rotate_left(grandparent)

    def rb_rotate_left(self, node):
        right_left_child = node.right.left
        if node.parent is not None:
            node.parent.rb_replace_child(node, node.right)
        else:
            self.root = node.right
            self.root.parent = None
        node.right.rb_set_child("left", node)
        node.rb_set_child("right", right_left_child)
            
    def rb_rotate_right(self, node):
        left_right_child = node.left.right
        if node.parent is not None:
            node.parent.rb_replace_child(node, node.left)
        else:
            self.root = node.left
            self.root.parent = None
        node.left.rb_set_child("right", node)
        node.rb_set_child("left", left_right_child)
        
    def search(self, word):
        current = self.root
        while current is not None:
            if word < current.item:
                current = current.left
            elif word > current.item:
                current = current.right 
            else:
                return True
        return False
 
#reads the file while creating the tree       
def file_reading(file, tree, number):
    text_opener = open(file, 'r')
    read_file = text_opener.read().splitlines()
    text_opener.close()
    if number == 1:
        for i in range(len(read_file)):
            node = Node(read_file[i])
            tree.insert(node)
    if number == 2:
        for i in range(len(read_file)):
            node = RBNode(read_file[i])
            tree.rb_insert(node)
    return tree

#This method will create all the premutations and will check in the process if
#the premutation is an anagram or not
def anagrams(word, tree, word_list, prefix=""):
    if len(word) <= 1:
        strg = prefix + word 
        if tree.search(strg):
            word_list.append(strg)
    else:
        for i in range(len(word)):
            cur = word[i: i + 1]
            before = word[0: i] # letters before cur
            after = word[i + 1:] # letters after cur
            
            if cur not in before: # Check if permutations of cur have not been generated.
                anagrams(before + after, tree, word_list, prefix + cur)
    return word_list

#This is method 3 which checks the max number of anagrams
def max_anagrams(file, tree, new_list):
    text_opener = open(file, 'r')
    read_file = text_opener.read().splitlines()
    text_opener.close()
    for i in range(len(read_file)):
        node = Node(read_file[i])
        tree.insert(node)
    
    text_opener2 = open(new_list, 'r')
    new_file = text_opener2.read().splitlines()
    text_opener.close()
    
    length_list = []
    w_list = []
    for word in range(len(new_file)):
        anagrams(new_file[word], tree, w_list, "")
        length_list.append(len(w_list))
    index_max = length_list.index(max(length_list))
    return new_file[index_max]
        
def main():
    main_tree1 = AVL()
    main_tree2 = RB()
    
    print("Enter 1 for AVL Tree, 2 for Red-Black Tree, or 3 for list", end = '')
    option = int(input())
    if option == 1:
        number = 1
        tree = file_reading('words.txt', main_tree1, number)
        w_list = []
        anagrams("spot", tree, w_list, "")
        print("Number of anagrams that are words: ",len(w_list))
        return
    
    if option == 2:
        number = 2
        tree = file_reading('words.txt', main_tree2, number)
        w_list = []
        anagrams("spot", tree, w_list, "")
        print("Number of anagrams that are words: ",len(w_list))
        return
    
    if option == 3:
        max_word = max_anagrams('words.txt', main_tree1, "max_list.txt")
        print(max_word)
        
    if option < 1 or option > 3:
        print("Invalid input")
        return

main()
