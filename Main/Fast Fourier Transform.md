Source: [The Fast Fourier Transform (FFT) - YouTube](https://www.youtube.com/watch?v=E8HeD-MUrjY&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=17)

Used to compute the [[Discrete Fourier Transform]]

[[Wavelet Transform]] is an enhanced FFT for compression.

Scaling: $O(nlog_2n)$

## Formulas
F = big DFT matrix
f = input vector of data points
$$
\hat{f} = {
	{F_{1024}f}
} = {
	\begin{bmatrix}
	I_{512} & -D_{512}\\
	I_{512} & -D_{512}
	\end{bmatrix}
	\begin{bmatrix}
	F_{512} & 0\\
	0 & F_{512}
	\end{bmatrix}
	\begin{bmatrix}
	f_{even}\\
	f_{odd}
	\end{bmatrix}
}
$$
$$
F_{1024} \to F_{512} \to ... \to F_{4} \to F_{2}
$$

## Implementation
