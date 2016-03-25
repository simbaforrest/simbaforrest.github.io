---
layout: post
type: mathpost
title: Plane Fitting using PCA
date: 2015-10-18
modified: 2016-03-25
tags: math note
---

Given a set of 3D points $\\{ p_i \\}$, we want to find out a plane (i.e., unit normal $n$ and center $q$) that describes this set of points as

\begin{equation} \label{eq_plane}
n^\intercal (p_i-q)=0, \forall i
\end{equation}

Of course real world data points usually won't strictly satisfy equation $\eqref{eq_plane}$. Thus we define function

\begin{equation} \label{eq_ppdist}
\text{dist}(p_i;n,q) \triangleq n^\intercal (p_i-q)
\end{equation}

which represents the *signed* point-plane distance between the plane $(n,q)$ and data point $p_i$ (recall that $n$ is a unit normal vector).

Now the problem of plane fitting can be formulated as a least square problem with the following cost function

$$
\begin{align}
\text{cost}(n,q) & \triangleq \sum_i \text{dist}^2(p_i;n,q) \nonumber \\
				 & = \sum_i (n^\intercal (p_i-q))^2 \nonumber \\
				 & = n^\intercal [ \cdots, p_i-q, \cdots ] [ \cdots, p_i^\intercal-q^\intercal, \cdots ]^\intercal n \nonumber \\
				 & = n^\intercal A(q) A(q)^\intercal n \label{eq_cost}
\end{align}
$$

where $A(q) \triangleq [\cdots, p_i-q, \cdots]$.

If $n$ is fixed to some value, to minimize the cost function $\eqref{eq_cost}$, we need to zero the following partial derivative (check [matrix calculus](https://en.wikipedia.org/wiki/Matrix_calculus))

\begin{equation} \label{eq_cost_dq}
\mathbf{0} = \frac{\partial \text{cost}(n,q)}{\partial q} \equiv \sum_i (2nn^\intercal q - 2nn^\intercal p_i).
\end{equation}

Solving equation $\eqref{eq_cost_dq}$ leads to the optimal plane center $ q^* $ as (no matter what $n$ is)

\begin{equation} \label{eq_bestq}
q^* = \frac{1}{|\\{p_i\\}|} \sum_i p_i.
\end{equation}


Now to solve the optimal plane normal $n$, we can plug equation $\eqref{eq_bestq}$ back to the cost function $\eqref{eq_cost}$ as

\begin{equation} \label{eq_costn}
\text{cost}(n; q^* ) \triangleq n^\intercal A( q^* ) A( q^* )^\intercal n = n^\intercal B( q^* ) n
\end{equation}

where $ q^* $ becomes a fixed parameter of this new cost function, and $$ B(q^* ) \triangleq A( q^* ) A( q^* )^\intercal $$.
Recall that $A(q)$ is a $ 3 \times |\\{p_i\\}| $ matrix, thus $B(q)$ should be a $ 3 \times 3 $ [*positive-definite*](https://en.wikipedia.org/wiki/Positive-definite_matrix) matrix. Now the optimal plane normal $ n^* $ needs to be solved by

$$
\begin{align}
n^* = \arg & \min_n 	 & & n^\intercal B( q^* ) n \label{eq_minn} \\
		   & \text{s.t.} & & n^\intercal n=1 \nonumber
\end{align}
$$

which reminds us a classic problem that can be solved by the singular value decomposition $A(q) \equiv U(q)S(q)V(q)^\intercal$, leading to (omitting $(q)$ in $B$ and other relevant matrices for clarity)

$$
\begin{align}
n^\intercal B n = & n^\intercal USV^\intercal VS^\intercal U^\intercal n \nonumber \\
				= & n^\intercal USS^\intercal U^\intercal n.
\end{align}
$$

Since $S=[\text{diag}(\sigma_1,\sigma_2,\sigma_3), \mathbf{0},\cdots,\mathbf{0}]$.

If we give each data point a corresponding weight $w_i$, then the distance equation $\eqref{eq_ppdist}$ can be changed to a weighted version as

\begin{equation} \label{eq_wdist}
\text{wdist}(p_i,w_i;n,q) \triangleq w_i n^\intercal (p_i-q)
\end{equation}

TO BE CONTINUED...