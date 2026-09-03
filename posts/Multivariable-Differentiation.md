---
title: Multivariable Differentiation
tags:
  - Differentiation
categories:
  - Calculus
date: 2026-08-18 14:07:09
katex: true
---
# 隐函数

利用全微分形式不变性。例如$F(x,y,z(x,y))=0$，将$z,x,y$视为相等地位的变量，直接求导，写为$$F'_x \mathrm{d}x+F'_y\mathrm{d}{y}+F'_z\mathrm{d}{z}=0$$

然后再根据这个等式解出$\frac{\partial z}{\partial x}$和$\frac{\partial z}{\partial y}$。

# 链式求导

画出链式树。

注意层级（这里约定与根越接近层级越高，与叶越接近层级越低。约定根为第零层级）。

求导时求导函数比求导变量层级最多只高一层。不能跨层级直接求。
对与求导函数同层级的求导变量求导结果为 0 。

对哪一个层级的变量求导得到的导函数也属于那一个层级，且是那个层级的变量的函数。

低层级的函数不能对高层级的变量求导。

两个变量名可能一致，但层级不同。如$z=f(x,y(x))$，$\frac{\partial z}{\partial x}$是对第一层级的$x$求偏导数，在第一层级，但$\frac{\mathrm{d}z}{\mathrm{d}x}$是对第二层级（叶）的$x$求全导数，在第二层级。