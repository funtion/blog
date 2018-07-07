---
layout: post
date: 2018-07-07 10:17
status: public
tags: [binary-tree, CSP, lambda, c++, tail-recursion]
title: '如何优雅的遍历一棵二叉树'
categories: [Programming]
---

## 初学 C++

```C++
void visit_recursive(shared_ptr<Node> node)
{
    if (node->left)
    {
        visit_recur(node->left);
    }

    cout << node->value << endl;

    if (node->right)
    {
        visit_recur(node->right);
    }
}
```

## 尾递归转化

```C++
// 由于 lambda 引用了调用栈上的元素，实际是无法进行尾递归优化的
void visit_tail(const shared_ptr<Node> node, const function<void()> continuation)
{
    auto next = [&]()
    {
        cout << node->value << endl;
        if (node->right)
        {
            visit_tail(node->right, continuation);
        }
        else
        {
            continuation();
        }
    };

    if (node->left)
    {
        visit_tail(node->left, next);
    }
    else
    {
        next();
    }
}
```

## 一行尾递归

```C++
void visit_tail(const shared_ptr<Node> node, const function<void()> continuation)
{
    node ? visit_tail(node->left, [&]() { cout << node->value << endl; visit_tail(node->right, continuation); }) : continuation();
}
```

## 强行循环

```C++
void visit_loop(const shared_ptr<Node> node)
{
    shared_ptr<Node> curNode = node;
    bool finsihed = false;
    function<void()> curContinuation = [&]()
    {
        finsihed = true;
    };

    while (!finsihed)
    {
        shared_ptr<Node> curNodeCopy = curNode;
        function<void()> curContinuationCopy = curContinuation;
        auto next = [&curNode, &curContinuation, curNodeCopy, curContinuationCopy]()
        {
            cout << curNodeCopy->value << endl;
            if (curNodeCopy->right)
            {
                curNode = curNodeCopy->right;
                curContinuation = curContinuationCopy;
            }
            else
            {
                curContinuationCopy();
            }
        };

        if (curNode->left)
        {
            curNode = curNode->left;
            curContinuation = next;
        }
        else
        {
            next();
        }
    }
}

```

## 纯手动压栈
```C++
void visit_stack(const shared_ptr<Node> node)
{
    stack<shared_ptr<Node>> nodes;
    stack<function<void()>> continuations;

    nodes.push(node);

    while (!nodes.empty())
    {
        function<void()> steps[3];
        steps[2] = [&nodes]()
        {
            nodes.pop();
        };

        steps[1] = [&]()
        {
            auto curNode = nodes.top();
            continuations.push(steps[2]);
            cout << curNode->value << endl;
            if (curNode->right)
            {
                nodes.push(curNode->right);
                continuations.push(steps[0]);
            }
        };

        steps[0] = [&]()
        {
            auto curNode = nodes.top();
            continuations.push(steps[1]);
            if (curNode->left)
            {
                nodes.push(curNode->left);
                continuations.push(steps[0]);
            }
        };

        if (continuations.empty())
        {
            steps[0]();
        }
        else
        {
            auto curCnt = continuations.top();
            continuations.pop();
            curCnt();
        }
    }
}
```
