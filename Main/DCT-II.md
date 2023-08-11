
Sources
---
[Diskrete Kosinustransformation – Wikipedia](https://de.wikipedia.org/wiki/Diskrete_Kosinustransformation)


Summary
---
- The DCT-II with a factor of $\frac{2}{N}$ is the inverse of [[DCT-III]] 
- The DCT-II is the most commonly used variant and is often referred to simply as "the DCT."([[Discrete Cosine Transform]])
- It's used in popular image and audio compression formats like [[Jpeg Compression]] (image) and AAC (audio).
- The DCT-II transforms a sequence of real values into a sequence of real [[Discrete Cosine Transform]] coefficients.
- The DCT-II is even-symmetric and only deals with real values.
- It's defined as a weighted sum of cosine functions of different frequencies.
- In [[image compression]], it helps to capture energy compaction in images by concentrating the most important information in the lower-frequency coefficients.

Mathematical Description
---
$\hat{f}$ = [[Vektor]] of Coefficients, real numbers
$f$= [[Vektor]] of Input points, real numbers
$N$= Size of input vector

$$
\begin{flalign}

\hat{f}_k = {
	\sum_{n=0}^{N-1} f_n cos[\frac{\pi}{N}(n+\frac{1}{2})k]
}

&&
\end{flalign}
$$