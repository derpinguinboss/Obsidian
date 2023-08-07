
Sources
===
- [The Discrete Fourier Transform (DFT) - YouTube](https://www.youtube.com/watch?v=nl9TZanwbBk&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=15)


Description
===
Useful if we have discrete data points/ a [[Vektor]] of data.


Mathematical description
===

$$
\vec{\hat{f}} = \begin{bmatrix}DFT\end{bmatrix} \vec{f}
$$
---
DFT matrix: 
---
$$
\begin{flalign}

i=\sqrt{-1}\\
\omega_n=e^{-2\pi\frac{i}{n}}
&&
\end{flalign}
$$
$$
\begin{flalign}

\begin{bmatrix}

1 && 1 && 1 & ... & 1\\
1 && \omega_n^1 && \omega_n^2 & ... & \omega_n^{n-1}\\
1 && \omega_n^2 && \omega_n^4 & ... & \omega_n^{2(n-1)}\\
\vdots && \vdots && \vdots & \ddots & \vdots\\
1 && \omega_n^{n-1} && \omega_n^{2(n-1)} & ... & \omega_n^{(n-1)^2}\\

\end{bmatrix}

&&
\end{flalign}
$$


Data to Fourier Coefficients:
---
$$
\begin{flalign}
\hat{f_k} = \sum_{j=0}^{n-1}{f_j e^{-i 2 \pi \frac{k}{n}}}
&&
\end{flalign}
$$


Fourier Coefficients To Data:
---
$$
\begin{flalign}
{f_k} = \frac{1}{n} \sum_{j=0}^{n-1}{\hat{f}_je^{i2 \pi \frac{k}{n}}}
&&
\end{flalign}
$$




Implementation
---
For the [[Matrix]] multiplication, the [[Fast Fourier Transform]] is used.
```csharp

Vector DFT(Vector f) {
	// For matrix multiplication, the FFT will be used
	return OmegaMatrix(f) * f;
}

Matrix OmegaMatrix(Vector f) {
	// Calculate big DFT matrix
	
}

```

 