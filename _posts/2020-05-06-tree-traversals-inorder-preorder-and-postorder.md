---
layout: post
title: "Tree Traversals (Inorder, Preorder and Postorder)"
date: 2020-05-06 17:51:17 +0200
categories: ['Binary tree', 'Algorithm']
---

## Tree traversal

Because of the complexity of tree data structure, there are many ways to traverse a binary tree, such as pre-order, in-order and post-order.

![Tree traversal]({{ site.baseurl }}/assets/images/2020-05-06/Tree traversal.png)

### Pre-order

LeetCode: https://leetcode.com/problems/binary-tree-preorder-traversal/

#### Recursive solution

```javascript
var preorderTraversal = function(root) {
    let result = [];

    function preorderT(node) {
        if (node == null) return;

        result.push(node.val);
        preorderT(node.left);
        preorderT(node.right);
    }

    preorderT(root);
    return result;
};
```

#### Iterative solution

In iterative solution, we use stack to simulate the recursive solution. We push firstly the root node. As the stack is FILO, we push the right child of current node, then the left child. So when we pop the stack, we will get the left child before right child.

```javascript
var preorderTraversal = function(root) {
    if (root == null) return [];

    let result = [];
    let stack = [root];

    while(stack.length != 0) {
        let cur = stack.pop();
        result.push(cur.val);        
        if (cur.right != null) stack.push(cur.right);
        if (cur.left != null) stack.push(cur.left);
    }

    return result;
};
```



### In-order

LeetCode: https://leetcode.com/problems/binary-tree-inorder-traversal/

#### Recursive solution

```javascript
var inorderTraversal = function(root) {
    let result = [];

    function inorderT(node) {
        if (node == null) return;

        inorderT(node.left);
        result.push(node.val);
        inorderT(node.right);
    }

    inorderT(root);
    return result;
};
```

#### Iterative solution

Firstly, we traverse the left sub tree and push the nodes until the leftest node, then visit this node and travere its right sub tree.

```javascript
var inorderTraversal = function(root) {
    let result = [];
    let stack = [];
    let cur = root;

    while (stack.length != 0 || cur != null) {
        if (cur != null) {
            stack.push(cur);
            cur = cur.left;
        } else {
            cur = stack.pop();
            result.push(cur.val);
            cur = cur.right;
        }
    }

    return result;
};
```



### Post-order

#### Recursive solution

LeetCode: https://leetcode.com/problems/binary-tree-postorder-traversal/

```javascript
var postorderTraversal = function(root) {
    let result = [];

    function postorderT(node) {
        if (node == null) return;
        
        postorderT(node.left);
        postorderT(node.right);
        result.push(node.val);
    }

    postorderT(root);
    return result;
};
```

#### Iterative solution

Same as the in-order traversal, we firstly traverse the left sub tree until the leftest node, then we check if the right sub tree of the leftest node should be traversed.

If the right child is null or is already visited, then current node should visited. Otherwise, we should visit the right sub tree.

```javascript
var postorderTraversal = function(root) {
    let result = [];
    let stack = [];
    let cur = root;
    let lastVisited = null;

    while (stack.length != 0 || cur != null) {
        if (cur != null) {
            stack.push(cur);
            cur = cur.left;
        } else {
            let topStack = stack[stack.length - 1];
            if (topStack.right != null && topStack.right != lastVisited) {
                cur = topStack.right;
            } else {
                lastVisited = stack.pop();
                result.push(lastVisited.val);
            }
        }
    }

    return result;
};
```
