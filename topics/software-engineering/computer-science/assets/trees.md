# Trees

#deep-dive

A tree is non-linear and a hierarchical data structure consisting of a collection of nodes such that each node of the tree stores a value and a list of references to other nodes (the “children”).

---

## Binary Tree

A binary tree is a tree data structure in which each node has at most two children, which are referred to as the left child and the right child.

![binary_tree](binary_tree.png)


---
## Binary Search Tree

A binary search tree, also called an ordered or sorted binary tree, is a rooted binary tree data structure with the key of each internal node being greater than all the keys in the respective node’s left subtree and less than the ones in its right subtree.


![binary_search_tree](binary_search_tree.png)

### BST Java Implementation

```JAVA
class Node {
    int value;
    Node left;
    Node right;

    Node(int value) {
        this.value = value;
        left = null;
        right = null;
    }
}

public class BinaryTree {
    Node root;

    private Node addRecursive(Node current, int value) {
        if (current == null) {
            return new Node(value);
        }
        
        if (value < current.value) {
            current.left = addRecursive(current.left, value);
        } else if (value > current.value) {
            current.right = addRecursive(current.right, value);
        } else {
            // value already exists
            return current;
        }
        
        return current;
    }

	// start the addRecursive from the root.
    public void add(int value) {
        root = addRecursive(root, value);
    }

    private boolean containsNodeRecursive(Node current, int value) {
        if (current == null) {
            return false;
        }
        
        if (value == current.value) {
            return true;
        }
        
        return value < current.value
            ? containsNodeRecursive(current.left, value)
            : containsNodeRecursive(current.right, value);
    }

	// start the containsNodeRecursive from the root.
    public boolean containsNode(int value) {
        return containsNodeRecursive(root, value);
    }
}

```

---
## Full Binary Tree

A full Binary tree is a special type of binary tree in which every parent node/internal node has either two or no children. It is also known as a proper binary tree.

![full_binary_tree](full_binary_tree.webp)

---
## Complete Binary Tree

A complete binary tree is a special type of binary tree where all the levels of the tree are filled completely except the lowest level nodes which are filled from as left as possible.

![complete_binary_tree](complete_binary_tree.webp)

---
## Heap

Heap is a tree-based data structure that follows the properties of a complete binary tree and is either a Min Heap or a Max Heap.

### Min Heap

The root is the smallest node and the data contained in each node is less than (or equal to) the node's children.

[min-heap](min-heap.png)

### Max Heap

The root is the biggest node and the data contained in each node is more than (or equal to) the node's children.

![max-heap](max-heap.png)


---
## Balanced Tree

A balanced binary tree, also referred to as a height-balanced binary tree, is defined as a binary tree in which the height of the left and right subtree of any node differ by not more than 1.

![balanced_tree](balanced_tree.png)

---
## Unbalanced Tree

An unbalanced binary tree is one that is not balanced.