
Sources
===
[CS231n Winter 2016: Lecture 7: Convolutional Neural Networks - YouTube](https://www.youtube.com/watch?v=LxfUGhug-iQ&list=PLkt2uSq6rBVctENoVBg1TpCC7OQi31AlC&index=7)
[1610.02357.pdf](https://arxiv.org/pdf/1610.02357.pdf)
[Convolutional neural network - Wikipedia](https://en.wikipedia.org/wiki/Convolutional_neural_network)
[CS231n Convolutional Neural Networks for Visual Recognition](https://cs231n.github.io/convolutional-networks/)
[Gentle Dive into Math Behind Convolutional Neural Networks | by Piotr Skalski | Towards Data Science](https://towardsdatascience.com/gentle-dive-into-math-behind-convolutional-neural-networks-79a07dd44cf9#:~:text=Convolution%20Layers&text=Forward%20propagation%20consists%20of%20two,and%20then%20adding%20bias%20b.)


Misc.
===
- Input layer channels = filter depth
- Input get's [[Padding | padded]] , so the width and height of the output array is equal to the width and height of the input array
- Often, [[Zero Padding]] is used to signify to the NN, that the padded inputs are to be ignored
- Output layer channels = filter count
- One filter has 
	- $w * h *  d$ Weights
	- $1$ bias
- $P_p(M_{ijk})$: Padding function for a 3 dimensional matrix
	- $p$ : padding amount
- $f_M$ : filter span
- $f_D$ : filter depth
- $\gamma_{jk}$ : gradients for a single filter


Input 
---
- $I_{ijk}$ with
	- $_i$ and $_j$ as width and height index 
	- $_k$ as the input depth
- 3 Dimensional
	- Width
	- Height
	- Channel

Filter
---
- Weights: $W_{ijk}^{n}$ with 
	- $^n$ as the index of the filter  among all the filters of the layer
	- $_k$ as the input depth
	- $_i$ and $_j$ as width and height index 
- Bias: $B_k$
	- $_k$ as the input depth
- 3 Dimensional
	- Width 
	- Height
	- Depth
- $w * h *  d$  Weights
- $1$ bias
- One filter for each channel of the input layer

Output 
---
- $O_{ijk}$
- 3 Dimensional
	- Width
	- Height
	- Channel
- One channel for each filter in the layer


Forward
===
$$
\begin{flalign}

O_{ijk} = 
	B_i +
	\sum_{d=0}^{f_D} 
		\sum_{x=0}^{f_M} 
			\sum_{y=0}^{f_M} 
				{W_{xyd}^{i}} * {P_p(\ I\ )_{{i+x-p}\ ,\ \ {j+y-p}\ ,\ \ {d}}}

&&\end{flalign}
$$


Gradient Descend
===

Gradient generation
---
$$
\begin{flalign}

\gamma_{jk} = A'(O_j) * \sum_{k}^{O^{+1}_M} {
	\gamma^{+1}_k * W^{+1}_{jk}
}

&&\end{flalign}
$$

Weight adjustment
---
$$
\begin{flalign}

\delta W_{ij} = {
	\gamma_{j} * O^{-1}_i * L_j
}

&&\end{flalign}
$$

Bias adjustment
---
$$
\begin{flalign}

\delta W_{ij} = {
	\gamma_{j} * L_j
}

&&\end{flalign}
$$
