---
layout: post
type: mathpost
title: How to derive essential matrix from camera projection matrices?
date: 2016-12-21
modified: 2017-01-04
comments: true
tags: math note
---

* TOC
{:toc}

Essential matrix is an important concept in 3D computer vision, especially if you want to understand how 3D reconstruction works. The equivalent concept in photogrammetry is also well-known as the coplanar equation. How to derive the form of essential matrix has been explained in Hartley's famous book[^1]. But its derivation involves the use of pseudo matrix and the concept of epipole. Here I want to offer another derivation which only uses two camera projection matrices and some hopefully well-known properties of cross product[^2].

[^1]:
    Hartley, Richard, and Andrew Zisserman. **Multiple view geometry in computer vision.** Cambridge university press, 2003.


[^2]:
    See [Wikipedia](https://en.wikipedia.org/wiki/Cross_product#Properties).

So we know that essential matrix is describing the relationship of corresponding points in two images through the relative poses of the two cameras. Thus the first thing we have at hand is the following equation

$$
\begin{align} 
\mathbf{q_1} &= \mathbf{K(IX+0) = KX} \nonumber \\
s \mathbf{q_2} &= \mathbf{K(RX+t)} \label{eq:projection}
\end{align}
$$

where $\mathbf{X \in R^3}$ is a 3D Euclidean point observed as homogeneous image points $\mathbf{q_i \in R^3}$ by the left camera ($i=1$) and the right camera ($i=2$) respectively. Without loss of generality, let's assume the two cameras have the same known intrinsic matrix $\mathbf{K}$, and that the left camera defines the world coordinate frame with identity rotation matrix $\mathbf{I}$ and zero translation vector, while the left camera's pose in the right camera frame as rotation matrix $\mathbf{R}$ and translation vector $\mathbf{t}$. Oh, BTW, $s$ is an unknown scaling parameter that you will frequently see in projective geometry equations of 3D vision.

Since we don't know $\mathbf{X}$ before 3D reconstruction, we should eliminate this symbol in the equation $\eqref{eq:projection}$ by moving K to left-hand side, and let $\mathbf{p_i=K^{-1}q_i}$:

$$
\begin{align}
\mathbf{p_1} &= \mathbf{X} \nonumber \\
s \mathbf{p_2} &= \mathbf{RX+t} \nonumber \\
\Rightarrow \nonumber \\
s \mathbf{p_2} &= \mathbf{R p_1+t} \label{eq:coplanar}
\end{align}
$$

Now equation $\eqref{eq:coplanar}$ is the exact condition that two corresponding point has to satisfy in the sense of two-view geometry, and also involve the relative pose between the two cameras. However there is still an annoying scaling parameter $s$. How can we get rid of it from the equation so we only need to focus on what we care about to derive the exact form of essential matrix?

Here is the trick that people often use, such as the derivation of Direct Linear Transform (DLT) in [^1]: it can be seen as the statement that a vector $\mathbf{p_2}$ is colinear with another vector $\mathbf{R p_1+t}$. Thus it is equivalent to the following equation using cross product "$\times$":

\begin{equation}
(\mathbf{R p_1+t}) \times \mathbf{p_2} = \mathbf{0} \label{eq:condition}
\end{equation}

Now the problem is how to massage the equation $\eqref{eq:condition}$ so that a matrix form emerges:

$$
\begin{align}
& (\mathbf{R p_1+t}) \times \mathbf{p_2} = \mathbf{0} \nonumber \\
\text{cross product is distributive over addition} \Rightarrow & \nonumber \\
& (\mathbf{R p_1}) \times \mathbf{p_2} + \mathbf{t} \times \mathbf{p_2} = 0 \nonumber \\
\text{using equation $\eqref{eq:coplanar}$} \Rightarrow & \nonumber \\
& (\mathbf{R p_1}) \times \mathbf{p_2} + \mathbf{t} \times (\mathbf{R p_1+t}) = 0 \nonumber \\
\mathbf{t} \times \mathbf{t}=0 \Rightarrow & \nonumber \\
& (\mathbf{R p_1}) \times \mathbf{p_2} + \mathbf{t} \times (\mathbf{R p_1}) = 0 \nonumber \\
\text{multiply $\mathbf{p_2}^T$ at both sides} \Rightarrow & \nonumber \\
& \mathbf{p_2}^T \Big( (\mathbf{R p_1}) \times \mathbf{p_2} \Big) + \mathbf{p_2}^T \Big( \mathbf{t} \times (\mathbf{R p_1}) \Big) = 0 \nonumber \\
\mathbf{b}^T \mathbf{a} \times \mathbf{b} = 0 \Rightarrow & \nonumber \\
& \mathbf{p_2}^T \Big( \mathbf{t} \times (\mathbf{R p_1}) \Big) = 0 \nonumber \\
\mathbf{a} \times \mathbf{b} = [\mathbf{a}]_{\times} \mathbf{b} \Rightarrow \nonumber \\
& \mathbf{p_2}^T [\mathbf{t}]_{\times} \mathbf{R p_1} = 0 \label{eq:essential}
\end{align}
$$

Now the last equation $\eqref{eq:essential}$ shows exactly the essential matrix's form $\mathbf{E}=[\mathbf{t}]_{\times} \mathbf{R}$.



