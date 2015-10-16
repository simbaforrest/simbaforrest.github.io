---
layout: post
type: mathpost
title: Plane Fitting using PCA
date: 2015-10-16
tags: note
---

Given a set of 3D points $\\{ p_i \\}$, we want to find out a plane (i.e., unit normal $n$ and center $q$) that describes this set of points as
$$
\begin{equation} \label{eq_plane}
n^\intercal (p_i-q)=0, \forall i
\end{equation}
$$

Of course real world data points usually won't strictly satisfy equation $\eqref{eq_plane}$. Thus we define function
$$
\begin{equation} \label{eq_ppdist}
\text{dist}(p_i;n,q) \triangleq n^\intercal (p_i-q)
\end{equation}
$$
which represents the *signed* point-plane distance between the plane $(n,q)$ and data point $p_i$ (recall that $n$ is a unit normal vector).

Now the problem of plane fitting can be formulated as a least square problem with the following cost function
$$
\begin{align}
\text{cost}(n,q) & \triangleq \sum_i \text{dist}^2(p_i;n,q) \nonumber \\\\
				 & = \sum_i (n^\intercal (p_i-q))^2 \nonumber \\\\
				 & = n^\intercal \[ \cdots, p_i-q, \cdots \] \[ \cdots, p_i^\intercal-q^\intercal, \cdots \]^\intercal n \nonumber \\\\
				 & = n^\intercal A(q) A(q)^\intercal n \label{eq_cost}
\end{align}
$$
where $A(q) \triangleq [\cdots, p_i-q, \cdots]$.

If $n$ is fixed to some value, to minimize the cost function $\eqref{eq_cost}$, we need to set
$$
\begin{equation} \label{eq_cost_dq}
0 = \frac{\partial \text{cost}(n,q)}{\partial q} \equiv \sum_i (2nn^\intercal q - 2n^\intercal p_i n).
\end{equation}
$$
Solving equation $\eqref{eq_cost_dq}$ (check [matrix calculus](https://en.wikipedia.org/wiki/Matrix_calculus)) leads to
$$
\begin{equation} \label{eq_bestq}
q^* = \frac{1}{|\\{p_i\\}|} \sum_i p_i.
\end{equation}
$$