---
title: Frontend Learning Index
tags:
  - Frontend
  - HTML
  - CSS
categories:
  - Frontend
date: 2026-06-26 00:00:00
katex: true
---

> [!TODO]
> [x] JS
> [ ] html
> [ ] css
> [ ] framework: React as example

# Frontend Learning Index

## JavaScript


- [[JavaScript-Basics]]：脚本引入、变量、基本类型、类型转换、控制流、函数基础、函数表达式、回调、箭头函数基础。
- [[JavaScript-Objects-and-Data-Structures]]：对象、拷贝、构造函数、可选链、Symbol、数组、迭代器、Map/Set、WeakMap/WeakSet、Date、JSON。
- [[JavaScript-Advanced-Functions]]：Rest、Spread、定时器、`bind`、`this` 与箭头函数。

## HTML

目标笔记：

- `HTML Semantic Structure Notes.md`

主题范围：

| 主题 | 内容 |
|---|---|
| 文档结构 | `<!doctype html>`、`html/head/body` |
| 常用标签 | 标题、段落、链接、图片、列表、表格 |
| 表单 | `form`、`input`、`label`、`button`、校验 |
| 语义化 | `header`、`main`、`section`、`article`、`nav`、`footer` |
| 资源引入 | `script`、`link`、图片与路径 |

## CSS

目标笔记：

- `CSS Layout and Styling Notes.md`

主题范围：

| 主题 | 内容 |
|---|---|
| 选择器 | 基础选择器、组合选择器、伪类/伪元素 |
| 盒模型 | `content/padding/border/margin`、`box-sizing` |
| 布局 | Flex、Grid、定位、浮动遗留知识 |
| 响应式 | 媒体查询、相对单位、移动端适配 |
| 样式组织 | 命名、层叠、优先级、变量 |

## React

目标笔记：

- `React Framework Notes.md`

主题范围：

| 主题 | 内容 |
|---|---|
| 组件 | 函数组件、组件拆分、组合 |
| JSX | 表达式、条件渲染、列表渲染 |
| Props / State | 数据传递、本地状态 |
| Hooks | `useState`、`useEffect`、`useMemo`、`useCallback`、`useRef` |
| 表单与事件 | 受控组件、事件处理 |
| 工程化 | Vite、目录结构、路由、请求、状态管理 |

## Expansion Rules

主题边界：

1. HTML、CSS、React 不直接混进 JavaScript 基础笔记；
2. 如果一个主题是跨领域的，例如“DOM 操作”，可以放在 `DOM and Browser API Notes.md`；
3. React 中只放框架相关内容，纯 JS 语法仍回链到 JavaScript 三篇；
4. 新增材料先作为独立草稿保存，再把可复用知识并入相应主题笔记。
