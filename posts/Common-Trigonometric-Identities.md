---
title: Common Trigonometric Identities
tags:
  - Integral
categories:
  - Calculus
date: 2026-08-18 14:06:52
katex: true
---


> 善用巧合。

0. (定义)$$\sec x=\frac{1}{\cos x},\csc x=\frac{1}{\sin x}$$
1. （平方差、降次）
$$\sin^{2}x+\cos^{2}x=1\implies \sin^{2}x = (1+\cos x)(1-\cos x)$$
2. （完全平方、升次去根号）$$\begin{align}
(1-\sin x)^{m}&=(1-2\sin \frac{x}{2}\cos \frac{x}{2})^{m}=\left( \sin \frac{x}{2}-\cos \frac{x}{2} \right)^{2m} \\
&=2^{m}\sin^{2m}\left( \frac{x}{2}-\frac{\pi}{4} \right)
\end{align}$$$$\begin{align}
(1+\sin x)^{m}&=(1+2\sin \frac{x}{2}\cos \frac{x}{2})^{m}=\left( \sin \frac{x}{2}+\cos \frac{x}{2} \right)^{2m} \\
&=2^{m}\sin^{2m}\left( \frac{x}{2}+\frac{\pi}{4} \right)
\end{align}$$ ^24cf79
3. （等价）$$\begin{align}
\frac{\sin x}{\cos^2x}=\tan x\sec x=\mathrm{d}\sec x \\
\frac{1}{\cos^{2}x}=\sec^{2}x=\mathrm{d}\tan x
\end{align}$$
4. (等价)$$\arcsin x=\frac{\pi}{2}-\arccos x$$
5. （转化）
$$\tan^{2}x=\sec^{2}x-1= \frac{\mathrm{d}{\tan x}}{\mathrm{d}{x}}-1  $$
6. （倒代换区间再现）
$$\arctan\left( \frac{1}{x} \right)=\frac{\pi}{2}-\arctan(x)$$
7. （降次）
$$\sin^{2}x= \frac{1-\cos{2x}}{2},\cos^{2}x= \frac{1+\cos2x}{2},\sin x\cos x=\frac{1}{2}\sin{2}x$$
8. （积化和差）
$$
\begin{align}
\sin\alpha\sin\beta &= \frac12\left[\cos(\alpha-\beta)-\cos(\alpha+\beta)\right]  \\
\cos\alpha\cos\beta &= \frac12\left[\cos(\alpha-\beta)+\cos(\alpha+\beta)\right]  \\ \\

\sin\alpha\cos\beta &= \frac12\left[\sin(\alpha+\beta)+\sin(\alpha-\beta)\right]  \\
\end{align}
$$
6. （半角公式）$$\begin{align}
\sin^2\frac{\alpha}{2}=\frac{1-\cos\alpha}{2}  \\
\cos^2\frac{\alpha}{2}=\frac{1+\cos\alpha}{2}  \\
\tan^2\frac{\alpha}{2}=\frac{1-\cos\alpha}{1+\cos\alpha}
\end{align}$$
7. (万能公式)$$\begin{align}
\sin x=\frac{2\tan\frac{x}{2}}{1+\tan^2\frac{x}{2}} \\
 \cos x=\frac{1-\tan^2\frac{x}{2}}{1+\tan^2\frac{x}{2}}  \\
 \tan x=\frac{2\tan\frac{x}{2}}{1-\tan^2\frac{x}{2}}  \\
 \mathrm{d}x=\frac{2}{1+\tan^2\frac{x}{2}}\,\mathrm{d}\!\left(\tan\frac{x}{2}\right)
\end{align}$$
6. (两角和差)$$\begin{align}
\sin(\alpha+\beta)=\sin\alpha\cos\beta+\cos\alpha\sin\beta  \\
\sin(\alpha-\beta)=\sin\alpha\cos\beta-\cos\alpha\sin\beta \\
 \cos(\alpha+\beta)=\cos\alpha\cos\beta-\sin\alpha\sin\beta  \\
 \cos(\alpha-\beta)=\cos\alpha\cos\beta+\sin\alpha\sin\beta  \\
 \tan(\alpha+\beta)=\frac{\tan\alpha+\tan\beta}{1-\tan\alpha\tan\beta}  \\
 \tan(\alpha-\beta)=\frac{\tan\alpha-\tan\beta}{1+\tan\alpha\tan\beta}
\end{align}$$