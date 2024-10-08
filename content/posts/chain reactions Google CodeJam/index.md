+++
author = "Damian Kuraś"
title = "Chain Reactions Solution — Google Code Jam 2022 Qualification Round"
date = 2022-04-05
lastmod = 2024-10-08
description = "Complete solution for the Google Code Jam 2022 Qualification Round problem 'Chain Reactions'. Includes C++ code and explanation."
draft = false
image = "images/chain-reactions-cover.png"
keywords = "Google Code Jam, Chain Reactions, C++, Competitive Programming, Code Jam solution"
categories = ["Competetive Programing"]
tags = ["C++","BFS","Tree Traversal"]
slug = "chain-reactions-google-codejam"
math = false
+++

## Introduction

Are you looking for an optimal solution to the Chain Reactions problem from Google Code Jam 2022? In this post, I’ll walk you through the problem statement, the core concepts, and a detailed C++ solution using BFS.

## Problem statement

### Problem

Wile lives alone in the desert, so he entertains himself by building complicated machines that run on chain reactions. Each machine consists of **N** modules indexed 1,2,...,**N**. Each module may point at one other module with a lower index. If not, it points at the abyss.

Modules that are not pointed at by any others are called _initiators_. Wile can manually trigger initiators. When a module is triggered, it triggers the module it is pointing at (if any) which in turn may trigger a third module (if it points at one), and so on, until the chain would hit the abyss or an already triggered module. This is called a chain reaction.

Each of the **N** modules has a fun factor **F<sub>i</sub>**. The fun Wile gets from a chain reaction is the largest fun factor of all modules that triggered in that chain reaction. Wile is going to trigger each initiator module once, in some order. The overall fun Wile gets from the session is the sum of the fun he gets from each chain reaction.

For example, suppose Wile has 4 modules with fun factor **F<sub>i</sub>**= 60, **F<sub>2</sub>**=20,**F<sub>3</sub>**=40, and **F<sub>4</sub>**=50 and module 1 points at the abyss, modules 2 and 3 at module 1, and module 4 at module 2. There are two initiators(3 and 4) that Wile must trigger, in some order.

{{< animated-gif src="./chain-reactions-1.gif" alt="Animation showcasing the modules pointing at abyss and their scores. The animations show the order of triggering"  >}}

As seen above, if Wile manually triggers module 4 first, modules 4, 2 and 1 will get triggered in the same chain reaction, for a fun of max(50,20,60) = 60. Then, when Wile triggers module 3, module 3 will get triggered alone (module 1 cannot get triggered again), for a fun of 40, and an overall fun for the session of 60+40=100.

{{< animated-gif src="./chain-reactions-2.gif" alt="Second animation showcasing the modules pointing at abyss and their scores. The animations show the order of triggering"  >}}

However, if Wile manually triggers module 3 first, modules 3 and 1 will get triggered in the same chain reaction, for a fun of max(40,60) = 60. Then, when Wile triggers module 4, modules 4 and 2 will get triggered in the same chain reaction, for a fun of max(50,20) = 50 and an overall fun of the session of 60+50=110.

Given the fun factors and the setup of the modules compute the maximum fun Wile can get if he triggers the initiators in the best possible order.

### Input

The first line of the input gives the number of test cases **T**. **T** test cases follow, each described using 3 lines. Each test case starts with a line with a single integer **N**, the number of modules Wile has. The second line contains **N** integers **F<sub>i</sub>**,**F<sub>2</sub>**,**...**,**F<sub>N</sub>** where **F<sub>i</sub>** is the fun factor of the i-th module. The third line contains **N** integers **P<sub>1</sub>**,**P<sub>2</sub>**,**...**,**P<sub>N</sub>**. If **P<sub>i</sub>** = 0, that means module _i_ points at the abyss. Otherwise, module _i_ points at module **P<sub>i</sub>**.

### Output

For each test case, output one line containing Case #x: y where x is the test case number (starting from 1) and y is the maximum fun Wile can have by manually triggering the initiators in the best possible order.

### Limits

Memory limit: 1GB.s

1 ≤ **T** ≤ 100

0 ≤ **F<sub>i</sub>** ≤ **10<sup>9</sup>**.

0 ≤ **P<sub>i</sub>** ≤ _i_-1, for all _i_.

### Problem statement TLDR

In the Chain Reactions problem, we have a machine composed of **N** modules. Each module is connected to another module, or it points to the abyss. Each module has a score called the fun factor. _Initiators_ are modules that are not pointed at by any other module. We can only trigger initiators, and after we trigger them, they chain trigger all modules they point at and stop either at the abyss or another module that was triggered already. The score we get from the full chain is equal to the highest score of any module in the chain. The challenge is to calculate the maximum fun Wile can get from these chain reactions by triggering the initiators in the optimal order.

## Solution

### Observations

Let's start with a pretty simple observation: modules pointing only at one other module or abyss lead to tree-like data structure where abyss is root and initiators are leafs pointing on their parent node. Another observation I made during the competition was that we always get at least all the initiators scores in final answers. Remaining modules can only add points to total score when they are triggered by node with a lower score than they have.

### Example

Let's look again at the example:

{{< animated-gif src="./chain-reactions-2.gif" alt="Animation showcasing the modules pointing at abyss and their scores. The animations show the order of triggering" >}}

Since we can add all leaves to the final score(50 and 40), we can focus on adding non leaves. Once we reach module 1 in the example above all information needed, is how much extra points it can give us in best case scenario. We can calculate the minimal of the children's nodes chain values. For example the module 2 should provide the 50 as the chain fun score as it was already triggered by module 4 with score of 50. If it had more children's, we would use the minimal one instead. Knowing the minimal of the chain fun score leading to the currently traversed node we can calculate extra score. For example at module 1 we can get at best 60-min(50,40) which is 20 extra points. Initiators score plus this extra points calculated at all nodes gives us optimal answer. In example, it is equal to 110(50+40+20).

If the children's of node are all higher than module fun score, we don't add any extra points. Like in the case of calculating extra points for module 2.

### Implementation details

We can efficiently traverse the three using BFS. Before proceeding a node we first traverse their children's if they have any. If the node is initiator we add its value to the total sum, then we return its max chain value to its parent. Once traversing non-leaf node we need to add an extra score from the difference between the currently traversed node and the lowest child only if it is positive value. After we are done traversing node, we need to return its chain maximal value.

## Solution in c++

Here’s the full implementation of the solution in C++:

```cpp
#include <bits/stdc++.h>
using namespace std;

unsigned long long int sum; // Global sum to keep track of the total fun

int bfs(int node, vector<vector<int>> &adj, vector<int> &funFactors)
{
    // If the current node has no children, it's a leaf node (an initiator)
    if (adj[node].empty())
    {
        sum += funFactors[node];
        return funFactors[node];
    }
    int minChildFun = INT_MAX;
    // Process all children of the current node
    for (int child : adj[node])
    {
        int childFun = bfs(child, adj, funFactors);
        minChildFun = min(minChildFun, childFun);
    }
    // Calculate how much fun the current node adds skip the negative values
    sum += max(0, funFactors[node] - minChildFun);

    return max(minChildFun, funFactors[node]);
}

int main()
{
    int T;
    cin >> T;
    for (int c = 1; c <= T; c++)
    {
        int n;
        cin >> n;
        vector<int> funFactors(n + 1);
        vector<vector<int>> adj(n + 1); // Adjacency list representing the tree
        for (int i = 1; i <= n; i++)
        {
            cin >> funFactors[i];
        }
        int node;
        for (int i = 1; i <= n; i++)
        {
            cin >> node;
            adj[node].push_back(i);
        }
        for (auto child : adj[0]) // we need to bfs from every child of abyss (root)
        {
            bfs(child, adj, funFactors);
        }
        cout << "Case #" << c << ": " << sum << "\n";
        sum = 0;
    }
    return 0;
}

```

## Conclusion

The Chain Reactions problem tests your understanding of tree structures and BFS, as well as your ability to optimize the total fun score calculation without exhaustively considering all possible triggering sequences. The solution provided here should help you understand the core concepts and enable you to apply similar techniques in the future.
