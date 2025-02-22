---
layout: default
title: "homography"
nav_order: 4
parent: fitting and alignment
grand_parent: Computer Vision
---

# 2D transformations
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

After the [feature detection](https://eetose.github.io/docs/image%20cv/corner/), we need to 

- match image patches to each other
  - what is a patch?
  - How to look for a patch?

- figure out transform between images
  - how can we transform images?
  - how do we solve for this transformation in given matches?
  
## 2D planar transformations
![png](/assets/image/panorama/transformation.jpg)

### Scaling

$$
\mathbf{x}^{\prime}=\mathbf{S} \mathbf{x} \quad \mathbf{x}^{\prime}=\left[\begin{array}{ll}
s_{x} & 0 \\
0 & s_{y}
\end{array}\right] \mathbf{x}
$$

### Rotation

$$
\mathbf{x}^{\prime}= \mathbf{R}  \mathbf{x} \\
\text{where } \mathbf{R}=\left[\begin{array}{cc}
\cos \theta & -\sin \theta \\
\sin \theta & \cos \theta
\end{array}\right] \text{ - rotation matrix}
$$

### Shear

$$
\mathbf{x}^{\prime} = \left[\begin{array}{cc}
1 & a \\
b & 1 
\end{array}\right] \mathbf{x}
$$

### <a name="translation"></a> Translation

$$
\mathbf{x}^{\prime}=[\mathbf{I} \quad \mathbf{t}] \overline{\mathbf{x}} \\
\mathbf{x}^{\prime}=\left[\begin{array}{ccc}
1 & 0 & t_ \mathbf{x} \\
0 & 1 & t_ \mathbf{y}
\end{array}\right]\left[\begin{array}{l}
\mathbf{x} \\
\mathbf{y} \\
1
\end{array}\right] \\
\text{where } \overline{\mathbf{x}} = [\text{x y 1}]^{t} \text{ :augment vector; } \mathbf{t} \text{: translation matrix.} 
$$

## Homogeneous Coordinates (齐次坐标)
Or projective coordinate, homogeneous is damn confusing terminology. <br>
Excellent videos: [Projective geometry](https://www.youtube.com/watch?v=NYK0GBQVngs) provided by [NJ Wildberger](https://web.maths.unsw.edu.au/~norman/), **UNSW** Prof！
### Why we bring in this?
Concepts first introduced in [translation](#translation) where $\overline{\mathbf{x}} = (x, y, 1)$ is the augmented vector. <br>

![png](/assets/image/panorama/pinhole.png) <br> 
Consequences of such projection could see the below picture.<br>
![png](/assets/image/panorama/consequence.png)
1. Farther away objects are smaller $\frac{Y+h}{Z} - \frac{Y}{Z} = \frac{h}{Z}$
2. Parallel lines converge at a point.
3. Parallel planes converge! *消失的地平线*

### Cartesian -> Homogeneous
We can only understand the 2D projection in the 3D homogeneous coordinate.
<p align = "center">
<img src="/assets/image/panorama/homo.png" width = 500/><br>
<em>Cartesian plane overlaid in the homogeneous coordinate</em>
</p>

The $\color{orange}{homogenous}$ (scale invariant) representation of a 2D point $ \mathbf{x} = (x,y)$ is a line L from origin through $\widetilde{\mathbf{x}} = (\widetilde{x}, \widetilde{y}, \widetilde{z})$. The third coordinate $\widetilde{z}$ is fictitious(虚构) such that:

$$\mathbf{\widetilde{x}} \equiv\left[\begin{array}{c}
\widetilde{x} \\
\widetilde{y} \\
\widetilde{z}
\end{array}\right] \equiv  \left [\begin{array}{c}
\frac{\tilde{x}}{\tilde{z}} \\
\frac{\tilde{y}}{\tilde{z}} \\
1
\end{array}\right] \Rightarrow {\mathbf{x}} = \left[\begin{array}{l}
\frac{\tilde{x}}{\tilde{z}} \\
\frac{\tilde{y}}{\tilde{z}}
\end{array}\right] = 
\left[\begin{array}{l} x\\ y \end{array}\right]
$$

(In case $\tilde{z}=0$ "point at infinity")<br>
$\color{red}{Insights}$: Transform $\mathbf{homogenous}$ (scale invariant) 3D coordinates to ordinary cartesian 2D. <br>
The $\color{orange}{homogenous}$ representation of a 2D line:

$$
\mathbf{\widetilde{x} \cdot \mathbf{\widetilde{l}} = ax+by+c=0} 
$$

Where $\mathbf{\widetilde{l}=(a,b,c)}$ could be normalized as $\mathbf{\widetilde{l}=(\mathbf{n}, d)}$
<p align = "center">
<img src="/assets/image/panorama/line.jpg" alt="hi" class="inline"/><br>
<em>2D line equation</em>
</p>

### Euclidean: rotation + translation

 $$
 \mathrm{x}^{\prime}=\left[\begin{array}{cc} \mathbf{R} & \mathbf{t} \\ \mathbf{0} & 1\end{array}\right] \overline{\mathbf{x}}
 =\left[\begin{array}{ccc}\cos \theta & -\sin \theta & t_{x}\\ \sin \theta & \cos \theta & t_{y}\\ 0 & 0 & 1 \end{array}\right] \overline{\mathbf{x}}
 $$

### Similarity: scale + rotate + translate
 
 $$
 \mathrm{x}^{\prime}=\left[\begin{array}{cc}s \mathbf{R} & \mathbf{t} \\ \mathbf{0} & 1\end{array}\right] \overline{\mathbf{x}}
 =\left[\begin{array}{ccc}a & -b & t_{x} \\ b & a & t_{y} \\ 0 & 0 & 1 \end{array}\right] \overline{\mathbf{x}}
 $$

### Affine: combinations of above are still affine

$$
\mathbf{x}^{\prime}=\left[\begin{array}{ccc}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
0 & 0 & 1
\end{array}\right]\overline{\mathbf{x}}
$$

### Projective transform

$$
  {\widetilde{\mathbf{x}}}^{\prime}= \widetilde{\mathbf{H}} \widetilde{\mathbf{x}} \quad \quad
  \left[\begin{array}{l}
{\tilde{x}}^{\prime} \\
{\tilde{y}}^{\prime} \\
{\tilde{z}}^{\prime}
\end{array}\right]=\left[\begin{array}{lll}
h_{11} & h_{12} & h_{13} \\
h_{21} & h_{22} & h_{23} \\
h_{31} & h_{32} & h_{33}
\end{array}\right]\left[\begin{array}{c}
\tilde{x} \\
\tilde{y} \\
\tilde{z}
\end{array}\right]\\
$$

### Hierarchy of 2D coordinate transformations
![png](/assets/image/panorama/hierarchy.jpg)

<!-- 
### Review of transformations in homogeneous coordinates
\begin{aligned}
&{\left[\begin{array}{c}
\tilde{x} \\
\tilde{y} \\
\tilde{z}
\end{array}\right]=\left[\begin{array}{ccc}
s & 0 & 0 \\
0 & s & 0 \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{c}
x \\
y \\
1
\end{array}\right] \quad\left[\begin{array}{c}
\tilde{x} \\
\tilde{y} \\
\tilde{z}
\end{array}\right]=\left[\begin{array}{ccc}
1 & 0 & t_{x} \\
0 & 1 & t_{y} \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{c}
x \\
y \\
1
\end{array}\right]} \\
&\quad\quad\quad\quad\quad\quad\text {Scaling \quad\quad\quad\quad\quad\quad\quad\quad\quad\quad Translation} \\
&{\left[\begin{array}{c}
\tilde{x} \\
\tilde{y} \\
\tilde{z}
\end{array}\right]=\left[\begin{array}{ccc}
\cos \theta & -\sin \theta & 0 \\
\sin \theta & \cos \theta & 0 \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{c}
x \\
y \\
1
\end{array}\right] \quad\left[\begin{array}{c}
\tilde{x} \\
\tilde{y} \\
\tilde{z}
\end{array}\right]=\left[\begin{array}{ccc}
a & -b & t_{x} \\
b & a & t{y} \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{c}
x \\
y \\
1
\end{array}\right]} \\
&\quad\quad\quad\quad\quad\quad\quad\quad\text {Rotation \qquad\qquad\qquad\qquad\qquad\qquad Similarity} 
\end{aligned}
 -->

## [Computing homographys](https://www.youtube.com/watch?v=l_qjO4cM74o&list=PL2zRqk16wsdp8KbDfHKvPYNGF2L-zQASc&index=5)
**Task:** given a set of matching features/points between *source* and *destination* images, find the $\color{orange}{\text{homography H}}$ that best fits.

$$\left[\begin{array}{c}x_{d} \\ y_{d} \\ 1\end{array}\right] \equiv\left[\begin{array}{c}\tilde{x}_{d} \\ \tilde{y}_{d} \\ \tilde{z}_{d}\end{array}\right]=\left[\begin{array}{lll}h_{11} & h_{12} & h_{13} \\ h_{21} & h_{22} & h_{23} \\ h_{31} & h_{32} & h_{33}\end{array}\right]\left[\begin{array}{c}x_{s} \\ y_{s} \\ 1\end{array}\right]$$

8 degrees of freedom requires 4 matches. For a given pair i of corresponding points:

$$x_{d}^{(i)}=\frac{\tilde{x}_{d}^{(i)}}{\tilde{z}_{d}^{(i)}}=\frac{h_{11} x_{s}^{(i)}+h_{12} y_{s}^{(i)}+h_{13}}{h_{31} x_{s}^{(i)}+h_{32} y_{s}^{(i)}+h_{33}} \\
  y_{d}^{(i)}=\frac{\tilde{y}_{d}^{(i)}}{\tilde{z}_{d}^{(i)}}=\frac{h_{21} x_{s}^{(i)}+h_{22} y_{s}^{(i)}+h_{23}}{h_{31} x_{s}^{(i)}+h_{32} y_{s}^{(i)}+h_{33}} $$

Rearrange the terms：

$$x_{d}^{(i)}\left(h_{31} x_{s}^{(i)}+h_{32} y_{s}^{(i)}+h_{33}\right)=h_{11} x_{s}^{(i)}+h_{12} y_{s}^{(i)}+h_{13} \\
  y_{d}^{(i)}\left(h_{31} x_{s}^{(i)}+h_{32} y_{s}^{(i)}+h_{33}\right)=h_{21} x_{s}^{(i)}+h_{22} y_{s}^{(i)}+h_{23} $$

Linear equation:

$$\left[\begin{array}{ccccccccc}x_{s}^{(i)} & y_{s}^{(i)} & 1 & 0 & 0 & 0 & -x_{d}^{(i)} x_{s}^{(i)} & -x_{d}^{(i)} y_{s}^{(i)} & -x_{d}^{(i)} \\ 0 & 0 & 0 & x_{s}^{(i)} & y_{s}^{(i)} & 1 & -y_{d}^{(i)} x_{s}^{(i)} & -y_{d}^{(i)} y_{s}^{(i)} & -y_{d}^{(i)}\end{array}\right]\left[\begin{array}{l}h_{11} \\ h_{12} \\ h_{13} \\ h_{21} \\ h_{22} \\ h_{23} \\ h_{31} \\ h_{32} \\ h_{33}\end{array}\right]=\left[\begin{array}{l}0 \\ 0\end{array}\right]$$

Define least squares problem:

$$\mathop{\min}\limits_{\mathbf{h}} \|\mathbf{A} \mathbf{h}\|^{2} \text{ such that } \|\mathbf{h}\|^{2} = 1$$ 

We know that:

$$\|\mathbf{A} \mathbf{h}\|^{2}=(\mathbf{A} \mathbf{h})^{T}(\mathbf{A} \mathbf{h})=\mathbf{h}^{T} A^{T} A \mathbf{h} 
\quad \text{and} \quad\|\mathbf{h}\|^{2}=\mathbf{h}^{T} \mathbf{h}=1$$

Define Loss function $L(\mathbf{h}, \lambda)$:

$$
L(\mathbf{h}, \lambda)=\mathbf{h}^{T} A^{T} A \mathbf{h}-\lambda\left(\mathbf{h}^{T} \mathbf{h}-1\right)
$$

Taking derivatives of $L(\mathbf{h}, \lambda)$ w.r.t $\mathbf{h}$: 

$$A^{T} A \mathbf{h}=\lambda \mathbf{h}$$

<u>SVD explanation</u>:

$$
\left\|\mathbf{U S V}^{T} \mathbf{h}\right\|^{2}=\mathbf{h}^{T} \mathbf{V S} \mathbf{S}^{T} \mathbf{U}^{T} \mathbf{U S V}^{T} \mathbf{h}=\mathbf{h}^{T} \mathbf{V} \mathbf{S}^{T} \mathbf{S V}^{T} \mathbf{h}=\left\|\mathbf{S} \mathbf{V}^{T} \mathbf{h}\right\|^{2}
$$

Where $\mathbf{A}=\mathbf{U S V}^{T}$, $\mathbf{U}$ and $\mathbf{V}$ are orthonormal and $\mathbf{S}$ is diagonal. And the equation could be furthered developed as:

$$\|\mathbf{S} \mathbf{y}\| \text{ subject to }  \|\mathbf{y}\|=1 \text{ where } \mathbf{y} = \mathbf{V}^{T} \mathbf{h}$$ 

<u>Conclusion</u>: Eigenvector $\mathbf{h}$ with smallest eigenvalue $\lambda$ of matrix $A^{T} A$ minimizes the loss function $L(\mathbf{h})$.<br>
OR since $\mathbf{S}$ is diagonal and the eigenvalues are sorted in descending order, let $\mathbf{y}=[0,0, \ldots, 1]^{T}$ would minimize $\|\mathbf{S y}\|$. So the solution $\mathbf{h}=\mathbf{V} \mathbf{y}$ would be the last column vector of $\mathbf{V}$.