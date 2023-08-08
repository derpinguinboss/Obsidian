# *TODO*
Sources
---
- [The Fast Fourier Transform (FFT) - YouTube](https://www.youtube.com/watch?v=E8HeD-MUrjY&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=17)
- [Fast Fourier transform - Algorithms for Competitive Programming](https://cp-algorithms.com/algebra/fft.html#application-of-the-dft-fast-multiplication-of-polynomials)


Description
---
- Used to compute the matrix-product $\hat{f}$ of the [[Discrete Fourier Transform]] (DFT) and the input [[Vektor]] $f$ 
- [[Wavelet Transform]] is an enhanced FFT for certain [[Image Compression]].
- Scaling in [[Big O Notation]]: $O(n log_2n)$


Mathematical description
---
DFT: [[Discrete Fourier Transform]]

$F_{1024}$ = DFT [[Matrix]]
$f$ = input [[Vektor]] of data points
$I$ = [[Identity Matrix]]
$D$ = [[Diagonal Matrix]] of $\omega$ 's from the DFT matrix
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
Vector FFT(Matrix DFT, Vector f) {
	
}

void fft(vector<complex<double>> &callersVector, bool invert) { 
	int n = callersVector.size(); 
	if (n == 1) 
		return; 
	
	vector<complex<double>> aEven(n / 2); 
	vector<complex<double>> aOdd(n / 2);
	
	for (int i = 0; 2 * i < n; i++) { 
		aEven[i] = callersVector[2*i]; 
		aOdd[i] = callersVector[2*i+1]; 
	} 
	
	fft(aEven, invert); 
	fft(aOdd, invert); 
	
	double ang = 2 * PI / n * (invert ? -1 : 1); 
	complex<double> w(1), 
	complex<double> wn(cos(ang), 
	complex<double> sin(ang)); 
	
	for (int i = 0; 2 * i < n; i++) { 
		
		callersVector[i] = aEven[i] + w * aOdd[i]; 
		callersVector[i + n/2] = aEven[i] - w * aOdd[i]; 
		
		if (invert) { 
			callersVector[i] /= 2; 
			callersVector[i + n/2] /= 2; 
		} 
		
		w *= wn; 
	} 
}
```