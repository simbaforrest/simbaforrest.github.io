---
layout: post
type: mathpost
title: How to Write a Matrix Multiplication Layer in Caffe?
date: 2017-04-17
comments: true
tags: math code note caffe
---

* TOC
{:toc}

In some deep neural networks (DNN), we may need to implement matrix multiplication, i.e.,
$\mathbf{Z}=\mathbf{XY}$.
In the popular [**Caffe**](http://caffe.berkeleyvision.org) library, the closest implementation of matrix multiplication is its [InnerProduct layer](http://caffe.berkeleyvision.org/tutorial/layers/innerproduct.html), i.e.,
$\mathbf{z=Wx+b}$.
However the difference is that the weight matrix $\mathbf{W} \in \mathbb{R}^{M \times N}$ has to be a parameter blob associated with the InnerProduct layer, instead of a bottum input blob of the layer.
To emulate a matrix multiplication in standard Caffe, as of April, 2017, is still doable but not so straightforward nor efficient, as explained in several stackoverflow discussions[^1][^2].

[^1]:
    http://stackoverflow.com/a/41658699/2303236

[^2]:
    http://stackoverflow.com/a/41580924/2303236

Since I'm not satisfied with such a solution, I'll describe below how to directly implement a matrix multiplication layer (in C++ for both cpu/gpu).

## Math
Assuming some basic understandings of backward propagation for solving DNN using stochastic gradient descent (SGD), firstly we need to figure out the partial derivatives.

### Definitions
Without loss of generality[^3], we have a loss function $l: \mathbb{R}^{M \times N} \to \mathbb{R}$. It accepts a input matrix $\mathbf{Z} \in \mathbb{R}^{M \times N}$, which itself is a matrix function performing matrix multiplication of two input matrices
$\mathbf{X} \in \mathbb{R}^{M \times K}, \mathbf{Y} \in \mathbb{R}^{K \times N}$, i.e., $\mathbf{Z=XY}$.

[^3]:
    $l$ does NOT need to be just a final loss function layer of DNN, it can be a composition of the final loss function with all layers after the matrix multiplication layer, but still be a scalar-valued function. Thus this is a general formulation.

Now we want to find out the partial derivatives of the loss function w.r.t. both $\mathbf{X}$ and $\mathbf{Y}$, i.e.,
$\underset{K \times M}{\frac{\partial l(\mathbf{Z})}{\partial \mathbf{X}}}$,
and
$\underset{N \times K}{\frac{\partial l(\mathbf{Z})}{\partial \mathbf{Y}}}$,
as expressions of the loss function's partial derivatives w.r.t. its direct input $\mathbf{Z}$, i.e.,
$\underset{N \times M}{\frac{\partial l(\mathbf{Z})}{\partial \mathbf{Z}}}$ is known[^4].

[^4]:
    We are sticking to the [Numerator-layout notation](https://en.wikipedia.org/wiki/Matrix_calculus#Numerator-layout_notation) for matrix calculus.


### Derivations
Since $\frac{\partial l(\mathbf{Z})}{\partial \mathbf{X}}$ is just a set of scalar-by-scalar partial derivatives arranged in a matrix layout, we can consider one entry and expand it using the chain rule and a scalar-by-matrix derivatives identity[^5]:

$$
\begin{align}
\text{the transposed $i,j$ is due to numerator-layout} \Rightarrow
\left( \frac{\partial l(\mathbf{Z})}{\partial \mathbf{X}} \right)_{ji}
&= \frac{\partial l(\mathbf{Z})}{\partial X_{ij}} \nonumber \\
&= tr(\frac{\partial l}{\partial \mathbf{Z}}\frac{\partial \mathbf{Z}}{\partial X_{ij}}) \nonumber \\
&= tr(\frac{\partial l}{\partial \mathbf{Z}}\mathbf{J}^{ij}\mathbf{Y}) \label{eq:partial_Z_partial_Xij} \\
&= tr(\mathbf{Y}\frac{\partial l}{\partial \mathbf{Z}}\mathbf{J}^{ij}) \label{eq:partial_l_partial_Xij} \\
\text{let } \underset{K \times M}{\mathbf{A}} \triangleq \mathbf{Y}\frac{\partial l}{\partial \mathbf{Z}} \Rightarrow
&= tr(\mathbf{A}\mathbf{J}^{ij}) \label{eq:A} \\
\text{definition of } tr(\cdot) \Rightarrow
&= \sum_{s=1}^{K} (\mathbf{A J}^{ij})_{ss} \nonumber \\
\text{definition of matrix multiplication} \Rightarrow
&= \sum_{s=1}^{K} \sum_{t=1}^{M} A_{st} (J^{ij})_{ts} \nonumber \\
&= A_{ji} (J^{ij})_{ij} + \sum_{s \neq j} \sum_{t \neq i} A_{st} (J^{ij})_{ts} \nonumber \\
&= A_{ji}. \label{eq:A_ji}
\end{align}
$$

[^5]:
    https://en.wikipedia.org/wiki/Matrix_calculus#Scalar-by-matrix_identities

Note equation $\eqref{eq:partial_Z_partial_Xij}$ comes from expanding $\frac{\partial \mathbf{Z}}{\partial X_{ij}}$ using the chain rule[^6], where the single-entry matrix $\underset{M \times K}{\mathbf{J}^{ij}}$ is $1$ only for the $(i,j)$-th entry and $0$ anywhere else. And equation $\eqref{eq:partial_l_partial_Xij}$ comes from a property of the matrix trace of multiple matrices' product[^7].

[^matrixcookbook]:
    Petersen, Kaare Brandt and Pedersen, Michael Syskind. [The Matrix Cookbook](http://www.imm.dtu.dk/pubdb/views/edoc_download.php/3274/pdf/imm3274.pdf).
    
[^6]:
    See equation 73-74 in section 2.4.1 of the matrix cookbook[^matrixcookbook].

[^7]:
    See equation 16 in section 1.1 of the matrix cookbook[^matrixcookbook].

### Conclusions
From equation $\eqref{eq:A}$ and $\eqref{eq:A_ji}$, one can easily conclude that:

$$
\begin{align}
\boxed{
\underset{K \times M}{\frac{\partial l(\mathbf{Z})}{\partial \mathbf{X}}} = \mathbf{A} = \underset{K \times N}{\vphantom{\frac{\partial l}{\partial \mathbf{Z}}} \mathbf{Y}} \underset{N \times M}{\frac{\partial l}{\partial \mathbf{Z}}}.
}
\label{eq:partial_l_partial_X}
\end{align}
$$

And similarly:

$$
\begin{align}
\boxed{
\underset{N \times K}{\frac{\partial l(\mathbf{Z})}{\partial \mathbf{Y}}} = \underset{N \times M}{\frac{\partial l}{\partial \mathbf{Z}}} \underset{M \times K}{\vphantom{\frac{\partial l}{\partial \mathbf{Z}}}\mathbf{X}}.
}
\label{eq:partial_l_partial_Y}
\end{align}
$$

## Code
In Caffe, each variable (e.g., $\mathbf{X}$) is stored as a blob. And each blob contains two parts of values, *data*, and *diff*,
where *data* stores the variable itself (i.e., $\mathbf{X}$),
and *diff* stores the partial derivatives of the loss function $l$ of the DNN w.r.t. the variable (i.e., $\left(\frac{\partial l}{\partial \mathbf{X}}\right)^T$).
Note that we are transposing the partial derivative matrix here.
Because in Caffe, *diff* and *data* have the same shape,
while in numerator-layout $\frac{\partial l}{\partial \mathbf{X}}$ is a $N \times M$ matrix with its entries corresponding to that of the transposed matrix $\mathbf{X}$ whose original shape is $M \times N$.

With these in mind, we can re-write the equation $\eqref{eq:partial_l_partial_X}$ and $\eqref{eq:partial_l_partial_Y}$ as the following equations:

$$
\begin{align}
\mathbf{X.diff} &= \mathbf{Z.diff} * \mathbf{Y.data}^T \label{eq:X_diff}, \\
\mathbf{Y.diff} &= \mathbf{X.data}^T * \mathbf{Z.diff} \label{eq:Y_diff}. \\
\end{align}
$$

Using equation $\eqref{eq:X_diff}$ and $\eqref{eq:Y_diff}$, we can write the backward propagation function (forward propagation is trivial) as the following pseudocode:

   ```C++
   backward(bottom, top) {
     // dl/dX' = dl/d(XY)' * Y', i.e., bottom[0].diff = top[0].diff * bottom[1].data'
     caffe_cpu_gemm(CblasNoTrans, CblasTrans, M, K, N, 1, top[0].diff, bottom[1].data, 0, bottom[0].diff);
     
     // dl/dY' = X' * dl/d(XY)', i.e., bottom[1].diff = bottom[0].data' * top[0].diff
     caffe_cpu_gemm(CblasTrans, CblasNoTrans, K, N, M, 1, bottom[0].data, top[0].diff, 0, bottom[1].diff);
   }
   ```

Of course, we need to take care of batched operations, and many other implementation details, following these wonderful tutorials[^8][^9]. A full implementation can be found in [my own version of Caffe](https://github.com/simbaforrest/caffe/blob/master/src/caffe/layers/matrix_multiplication_layer.cpp).

[^8]:
    https://github.com/BVLC/caffe/wiki/Simple-Example:-Sin-Layer

[^9]:
    https://chrischoy.github.io/research/making-caffe-layer/