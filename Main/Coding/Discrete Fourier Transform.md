
Sources
===
- [The Discrete Fourier Transform (DFT) - YouTube](https://www.youtube.com/watch?v=nl9TZanwbBk&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=15)
- [Computing the DFT Matrix - YouTube](https://www.youtube.com/watch?v=Xw4voABxU5c&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=16)
- [Discrete Fourier Transform | Brilliant Math & Science Wiki](https://brilliant.org/wiki/discrete-fourier-transform/#:~:text=The%20DFT%20formula%20for%20X,%2C%20x%20N%20%E2%88%92%201%20))
- [How to Calculate Discrete Fourier Transform - Tutorial, Definition, Formula, Example](https://www.easycalculation.com/engineering/mechanical/learn-discrete-fourier-transform.php)
- [Diskrete Fourier-Transformation – Wikipedia](https://de.wikipedia.org/wiki/Diskrete_Fourier-Transformation#Inverse_Diskrete_Fourier-Transformation_(iDFT))

Description
===
Useful if we have discrete data points/ a [[Vektor]] of data.


Mathematical description
===
$\vec{\hat{f}} =$ Coefficient [[Vektor]] of [[Complex Numbers]], each one of which encodes the amplitude and phase of a sinusoidal wave
$\vec{f} =$ Input [[Vektor]] of [[Complex Numbers]]
$n =$ Index in $\vec{f}$
$k =$ Index in the $\vec{\hat{f}}$
$N =$ Number of Input elements
$W =$ "DFT [[Matrix]]" or "Fourier-Matrix", with dimensions of  $N * N$
$i=\sqrt{-1}$
$\omega_{knN} = e ^ {- 2 \pi i k n / N}$
$\not\omega_{knN} = e ^ {2 \pi i k n / N}$
$\omega_N = e^{-2 \pi i / N}$ 

Conversion
---
$$
\begin{flalign}
\vec{\hat{f}} = W \vec{f}
&&
\end{flalign}
$$


Transformation matrix W: 
---
The W matrix is a [[Vandermonde Matrix]] and a [[Unitary Matrix]]
$\omega_n = e^{-2 \pi i / N}$ 
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
(Basically Matrix Multiplication)
$$
\begin{flalign}
\hat{f_k} = \sum_{n=0}^{N-1}{f_n * \omega_{knN}}
&&
\end{flalign}
$$


Fourier Coefficients To Data:
---
(Basically Matrix Multiplication)
$$
\begin{flalign}
{f_n} = \frac{1}{N} \sum_{k=0}^{N-1}{\hat{f}_k *\not\omega_{knN}}
&&
\end{flalign}
$$


Implementation
===

For the [[Matrix]] multiplication, the [[Fast Fourier Transform]] is used preferably.
```csharp
static Complex Omega(int k, int n, int N) 
	=> ComplexExtensions.Exp(-2 * Constants.Pi * Complex.ImaginaryOne * k * n / N);

static Complex InverseOmega(int k, int n, int N) 
	=> ComplexExtensions.Exp(2 * Constants.Pi * Complex.ImaginaryOne * k * n / N);


static Complex[] Forward(int[] inputVector)
{
	int N = inputVector.Length;            
	Complex[] f_hat = new Complex[N];
	Complex[] f = inputVector.Select(x => new Complex(x, 0)).ToArray();
	
	// Iterate f_hat
	for (int k = 0; k < N; k++)
	{
		// Iterate f
		for (int  n = 0; n < N; n++)
		{
			f_hat[k] += f[n] * Omega(k, n, N);
		}
	}
	
	return f_hat;
}

static double[] Inverse(Complex[] inputVector)
{
	int N = inputVector.Length;
	Complex[] f_hat = inputVector;
	Complex[] f = new Complex[N];
	
	// iterate f
	for (int n = 0; n < N; n++)
	{
		// init
		f[n] = Complex.Zero;
		
		// iterate f_hat
		for (int k = 0; k < N; k++)
			f[n] += f_hat[k] * InverseOmega(k, n, N);
		
		// normalize
		f[n] /= N;
	}
	
	return f.Select(x => x.Real).ToArray();
}
```

 