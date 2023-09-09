# Populating Next Right Pointers in Each Node II

## Description

Given a binary tree

```c
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to `NULL`.

Initially, all next pointers are set to `NULL`.

![img](https://assets.leetcode.com/uploads/2019/02/15/117_sample.png)

## Algorithm 1

One feasible algorithm is depth-first-search. Here, we need to the visit right child before visiting the left child. We can recode the rightmost node so far for each depth.

```c++
void connectHelper(Node *root, vector<Node*> & ptrs, unsigned depth)
{
    if (!root)
        return;
    if (depth >= ptrs.size())
    {
        ptrs.push_back(root);
    }
    else
    {
        root -> next = ptrs[depth];
        ptrs[depth] = root;
    }
    connectHelper(root -> right, ptrs, depth + 1);
    connectHelper(root -> left, ptrs, depth + 1);
}

Node* connect(Node* root)
{
    vector<Node*> ptrs;
    connectHelper(root, ptrs, 0);
    return root;    
}
```

## Algorithm 2

Previous algorithm needs O(depth) space to store rightmost node for each depth. However, this problem actually let us to divide nodes of different depths into levels. So breath-first-search could solve it by its nature. Beside, we can take advantage the field `Node *next` to form a queue.

```c
struct Node* connect(struct Node* root)
{
    struct Node *rightmost = root, *front = root, *tail = root, *current;
    while (current = front)
    {
        if (current -> left)
        {
            tail -> next = current -> left;
            tail = tail -> next;
        }
        if (current -> right)
        {
            tail -> next = current -> right;
            tail = tail -> next;
        }
        front = current -> next;
        if (rightmost == current)
        {
            current -> next = NULL;
            rightmost = tail;
        }
    }
    return root;
}
```

