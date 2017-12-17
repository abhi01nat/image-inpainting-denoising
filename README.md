Denoising and Inpainting of Images Using Wavelets
=================================================

The code below was a part of my final submission for the class YSC4206 Harmonic Analysis at Yale-NUS College. I use [B-Spline wavelets](https://en.wikipedia.org/wiki/Spline_wavelet) for the denoising and inpainting of "natural" images i.e. images that are not themselves pictures of noise or randomly generated pixels. 

The basic idea is as follows - let $x_0$ be a signal in $\mathbb{R}^n \times \mathbb{R}^m$, and let $A: \mathbb{R}^n \times \mathbb{R}^m \to \mathbb{R}^p \times \mathbb{R}^q$ be a linear transformation. This linear transformation is a distortion; for images, typical examples include deletion of certain pixels, or convolution with some matrix (as in the case of a Gaussian blur, for instance). We wish to recover $x_0$ from the observed signal $y := Ax_0$. If $A$ is invertible, then the problem is not very interesting as there is only one possible solution to $Ax = y$. In most practical scenarios, however, $A$ is either non-invertible or ill-conditioned (meaning that its inverse is difficult to compute numerically). We must leverage additional structure in the problem. 

Natural images have large smooth regions (regions where the colour gradient is low, or regions where there is not much going on). This means that when translated to the wavelet domain, these regions have low-magnitude or zero wavelet coefficients. In other words, natural images are *sparse* in the wavelet domain. The recovery of sparse signals from a noisy, linearly distorted image of the signal using convex optimisation techniques is a well-studied topic; see [this paper](https://arxiv.org/abs/1012.0621) by Recht et al. If we know what $A$ is, we can restate the problem as follows - find the signal $x$ in $\mathbb{R}^n \times \mathbb{R}^m$ with the least possible non-zero coefficients in the wavelet domain such that $Ax=y$. Mathematically, we want $$\operatorname{argmin}_{x \in \mathbb{R}^n \times \mathbb{R}^m} \|Wx\|_0$$ subject to $$Ax = y$$ where $W$ is the analysis operator of our wavelet system (assumed to be a tight frame for the purposes of this discussion). Note that this is not guaranteed to give us back the original signal, since the solution set $\{x\ |\ Ax = y \}$ is a non-trivial affine subspace of $\mathbb{R}^n \times \mathbb{R}^m$. There could be other signals that are sparser in the wavelet domain[<sup>1</sup>](#1). These are likely to be close to our original signal if our assumption of sparsity in the wavelet domain holds true, since they will all be "smoothed-out" versions of our original signal.

In practice, $y$ might have noise added to it. If we assume that the noise if of low magnitude compared to the signal, then we can restate the problem as follows. Find $$\arg \min_{x \in \mathbb{R}^n} \|Ax - y\|_2 + \|\operatorname{diag}(\lambda)Wx\|_0$$ The second term in the expression is the regularisation term, and $\lambda > 0$ is a parameter that controls the regularisation penalty. 

The $\| \cdot \|_0$ function (the "zero norm") is not really a norm. In general, solving the above two problems is NP-hard. If we use the standard Euclidean norm, the problem becomes a simple one that can be solved using gradient descent. However, the standard Euclidean norm does not in general give the sparsest solution, while the 1-norm gives the sparsest solution with high probability. The 1-norm is not differentiable so standard gradient descent will not work. An alterative called [proximal gradient descent](https://en.wikipedia.org/wiki/Proximal_gradient_method) can be used instead. Here I have used an optimised version of this algorithm called accelerated proximal gradient descent (APGD), covered in [this blog post](https://blogs.princeton.edu/imabandit/2013/04/01/acceleratedgradientdescent/). 

<a name="1"> </a>[1] There *are* certain special conditions under which the problem is guaranteed to have a unique solution; this is the basis of compressed sensing. These conditions have to do with the nature of the matrix $A$. For a (relatively) simple introduction, watch [this Youtube lecture](https://www.youtube.com/watch?v=6ZNLGTfbo7g) by Ben Recht. 
