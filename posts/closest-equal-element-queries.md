---
title: closest equal element queries
tags:
  - LeetCode
categories:
  - Record
date: 2025-11-01 00:00:00
katex: true
---

[题目](https://leetcode.cn/problems/closest-equal-element-queries)

```C++
vector<int> solveQueries(vector<int>& nums, vector<int>& queries) {
        map<int,vector<int>>mp;
        for(int i=0;i<nums.size();++i){
            mp[nums[i]].push_back(i);
        }
        int n = queries.size();
        vector<int>ans(n,-1);
        for(int j=0;j<n;++j){
            int q = queries[j];
            auto & vec = mp[nums[q]];
            if(vec.size()==1){
                continue;
            }
            int idx = ranges::lower_bound(vec,q)-vec.begin();
            
            ans[j] = min((vec[(idx+1)%vec.size()]-vec[idx]+nums.size())%nums.size(),(vec[idx] - vec[(idx-1+vec.size())%vec.size()]+nums.size())%nums.size());
        }
        return ans;
    }
```
跟它类似的还有比如[981. 基于时间的键值存储](https://leetcode.cn/problems/time-based-key-value-store/)

这类题型的本质是：**对于某个属性相同的元素，需要在它们的位置序列中进行高效查找。**

流程：

{% mermaid %}
flowchart TD
    A[开始：读取输入数据] --> B[建立值-vector映射表]
    B --> C{遍历原数组/序列}
    C --> D[将每个元素的值作为key<br/>将其序号加入对应的vector]
    D --> E{是否遍历完成?}
    E -->|否| C
    E -->|是| F[处理查询]
    
    F --> G{遍历每个查询}
    G --> H[获取查询的目标值]
    H --> I{映射表中是否存在该值?}
    I -->|否| J[返回-1或默认值]
    I -->|是| K[获取该值对应的位置序列]
    K --> L[在位置序列上进行二分查找]
    L --> M[使用lower_bound/upper_bound<br/>找到合适的位置]
    M --> N[检查前后相邻位置]
    N --> O[计算距离/时间差<br/>选择最优解]
    O --> P[记录查询结果]
    P --> Q{是否处理完所有查询?}
    Q -->|否| G
    Q -->|是| R[返回所有查询结果]
    
    J --> Q
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style L fill:#fff3e0
    style R fill:#e8f5e8
    
    classDef keyStep fill:#ffeb3b,stroke:#f57f17,stroke-width:2px
    class B,L keyStep
{% endmermaid %}
