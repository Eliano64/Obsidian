---
title: Trie
tags:
  - DataStructureAndAlgorithm
  - String
  - Tree
categories:
  - Data Structure & Algorithm
date: 2025-08-24 00:00:00
katex: true
---

字典树（Trie）是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。
每个节点有26个子节点，子节点的序号分别代表26个字母。
因此，从根节点到某个节点的路径表示一个字符串。
有时候，每个节点会有一个标记，用于表示是否为一个字符串的结束。
如果一个节点的标记为true，那么从根节点到该节点的路径表示的字符串就是一个单词。

# C++示例

以下是一个简单的C++实现，支持插入、搜索和前缀搜索。

```C++
class Node {
public:
    int end;
    vector<unique_ptr<Node>> son;
    Node():end(0),son(26){};
};

class Trie {
public:
    unique_ptr<Node> root;
    Trie() {
        root=make_unique<Node>();
    }
    
    void insert(string word) {
        Node* cur=root.get();
        for(auto chr: word){
            if(cur->son[chr-'a']==nullptr)
                cur->son[chr-'a']=make_unique<Node>();
            cur=cur->son[chr-'a'].get();
        }
        cur->end=1;
    }
    
    bool search(string word) {
        Node* cur = root.get();
        for (char c : word) {
            c -= 'a';
            if (cur->son[c] == nullptr) { 
                return false;
            }
            cur = cur->son[c].get();
        }
        return cur->end==1;
    }
    
    bool startsWith(string prefix) {
        Node* cur = root.get();
        for (char c : prefix) {
            c -= 'a';
            if (cur->son[c] == nullptr) { 
                return false;
            }
            cur = cur->son[c].get();
        }
        return true;
    }
};
```
