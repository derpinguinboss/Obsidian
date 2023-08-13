
Summary
---
- The DCT-IV is it's own inverse with a factor of $\frac{2}{N}$
- The DCT-IV is closely related to [[DCT-II]] and [[DCT-III]].
- It's defined by expressing [[DCT-III]] in terms of [[DCT-II]].
- It's used in a similar manner to [[DCT-III]] for signal reconstruction but with a different formulation.

Mathematical Description
---
$\hat{f}$ = [[Vektor]] of Coefficients, real numbers
$f$= [[Vektor]] of Input points, real numbers
$N$= Size of input vector

Forward
$$\begin{flalign}

\hat{f}_k = {
	\sum_{n=0}^{N-1} f_n cos 
	\left[ 	\frac{\pi}{N}\left(n+\frac{1}{2}\right) \left(k+\frac{1}{2}\right) \right]
}

&&\end{flalign}$$

Inverse
$$\begin{flalign}

\hat{f}_k = {
	\frac{2}{N}
	\sum_{n=0}^{N-1} f_n cos 
	\left[ 	\frac{\pi}{N}\left(n+\frac{1}{2}\right) \left(k+\frac{1}{2}\right) \right]
}

&&\end{flalign}$$