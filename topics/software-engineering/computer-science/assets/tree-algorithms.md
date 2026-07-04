# Tree Algorithms

#deep-dive

Tree algorithms operate on hierarchical, non-linear data structures. They enable efficient traversal, search, and manipulation of nodes using recursive or iterative strategies, forming the basis for many graph, parsing, and indexing problems.

---
### Pre-Order Traversal

Pre-order traversal is a tree traversal algorithm that visits the root node first, then recursively traverses the left subtree, followed by the right subtree.

```python
def preOrderTraversal(node):
    if node is None:
        return
    print(node.data, end=", ")
    preOrderTraversal(node.left)
    preOrderTraversal(node.right)
```

![preorder-traversal](preorder-traversal.gif)

---
### In-Order Traversal

In-order traversal is a tree traversal algorithm that visits the left subtree, the root, and then the right subtree. This is the most common way to traverse a binary search tree. It is also used to create a sorted list of nodes in a binary search tree.

```python
def inOrderTraversal(node):
    if node is None:
        return
    inOrderTraversal(node.left)
    print(node.data, end=", ")
    inOrderTraversal(node.right)
```

![inorder-traversal](inorder-traversal.gif)

---
### Post-Order Traversal

Post-order traversal is a type of tree traversal that visits the left subtree, then the right subtree, and finally the root node. 

This is the opposite of pre-order traversal, which visits the root node first, then the left subtree, and finally the right subtree.

```python
def postOrderTraversal(node):
    if node is None:
        return
    postOrderTraversal(node.left)
    postOrderTraversal(node.right)
    print(node.data, end=", ")
```

![postorder-traversal](postorder-traversal.gif)

---
### Breadth-First Search

Breadth first search is a graph traversal algorithm that starts at the root node and explores all of the neighbour nodes at the present depth prior to moving on to the nodes at the next depth level.


```JAVA
    void levelOrderRec(Node root, int level, List<List<Integer>> res) {
        // Base case: If node is null, return
        if (root == null) return;

        // Add a new level to the result if needed
        if (res.size() <= level)
            res.add(new ArrayList<>());

        // Add current node's data to its corresponding level
        res.get(level).add(root.data);

        // Recur for left and right children
        levelOrderRec(root.left, level + 1, res);
        levelOrderRec(root.right, level + 1, res);
    }
    
    // Function to perform level order traversal
    List<List<Integer>> levelOrder(Node root) {
        // Stores the result level by level
        List<List<Integer>> res = new ArrayList<>();
        levelOrderRec(root, 0, res);
        return res;
    }

	...
	levelOrder(root)
```

---
### Depth-First Search

Depth first search is a graph traversal algorithm that starts at a root node and explores as far as possible along each branch before backtracking.

DFS can be implemented using Pre-order, In-order and Post-order Traversal.

```JAVA
    public void dfs(TreeNode node) {
        if (node == null) return;

        // Process the current node (pre-order)
        System.out.print(node.val + " ");

        dfs(node.left);   // Visit left subtree
        dfs(node.right);  // Visit right subtree
    }
```