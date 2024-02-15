	
Sources
---
[Diskrete Kosinustransformation – Wikipedia](https://de.wikipedia.org/wiki/Diskrete_Kosinustransformation)


Summary
---
- The DCT-III with a factor of $\frac{2}{N}$ is the inverse of [[DCT-II]].
- It's used to reconstruct a sequence of real values from a sequence of [[Discrete Cosine Transform]] coefficients.
- Like DCT-II, it's even-symmetric and only deals with real values.
- It's often used in the decompression process of formats that use DCT-II for compression, such as [[Jpeg Compression]].

Mathematical Description
---
$\hat{f}$ = [[Vektor]] of Coefficients, real numbers
$f$= [[Vektor]] of Input points, real numbers
$N$= Size of input vector


$$
\begin{flalign}

\hat{f}_k = {
	\frac{1}{2}f_0 + \sum_{n=0}^{N-1} f_n cos[\frac{\pi}{N}(k+\frac{1}{2})n]
}

&&
\end{flalign}
$$
