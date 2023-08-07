
Sources
---
- [The Fast Fourier Transform (FFT) - YouTube](https://www.youtube.com/watch?v=E8HeD-MUrjY&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=17)


Description
---
- Used to compute the [[Discrete Fourier Transform]] (DFT)
- [[Wavelet Transform]] is an enhanced FFT for [[Image Compression]].
- Scaling in [[Big O Notation]]: $O(nlog_2n)$


Mathematical description
---
F = big DFT [[Matrix]]
f = input [[Vektor]] of data points
I = [[Identity Matrix]]
D = [[Diagonal Matrix]] of $\omega$ 's from the DFT matrix

$$
\begin{flalign}
\hat{f} = {
	{F_{1024}f}=
} 
{
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
}&&
\end{flalign}
$$
$$
\begin{flalign}
F_{1024} \to F_{512} \to ... \to F_{4} \to F_{2}&&
\end{flalign}
$$


Implementation
---
```csharp
int[] FFT(int[] f) {
	// get next smaller step of F size
	int nextSize = f.Length / 2
	
	// Get fEven fOdd Input Vector
	int[] fEven = extractEven(f)
	int[] fOdd = extractOdd(f)
	int[] fEvenOdd = StackVectors(fEven, fOdd)
	
	// Get smaller DFT matrix 
	Matrix Fsmaller = // some recursive call here
	Matrix zeroPadding = Matrix.ZeroPadded(nextSize, nextSize)
	Matrix F = CombineDiagonal(Fsmaller, zeroPadding)
	
	// Get Identity and Diagonal matrices
	Matrix identity = Matrix.Identity(nextSize, nextSize)
	Matrix diagonal = // the [[Discrete Fourier Transform]] matrix
	Matrix ID = CombineVertical(identity, -diagonal)
	
	return ID * F * fEvenOdd
}

Matrix GetF(Matrix F) {
	
}
```