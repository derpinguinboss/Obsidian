
Sources
===
[3b1b backpropagation - YouTube](https://www.youtube.com/results?search_query=3b1b+backpropagation)


Misc.
===
- Activation function $A(x)$ and it's derivative $A'(x)$
- $i$ as the input iterator
	- $i_M$ Input size/max
- $j$ as the output iterator
	- $j_M$ Output size/max
- Biases $B_j$
- Weights $W_{ij}$ with
- Gradients $\gamma_{j}$ 
- Learning rates $L_j$
- Getting other layers from the net: 
	- $n$ Layer backward: $-n$ In superscript 
	- $n$ Layer forward: $+n$ In superscript 
	- Examples:
		- Previous layer Output: $O^{-1}_i$ (equivalent to input, therefor iterated using $i$)
		- Next layer gradients: $\gamma^{+1}_{k}$

Input
---
- 1 Dimensional

Output
---
- $O_j$
- 1 Dimensional


Forward
===
$$
\begin{flalign}

O_{j} = A(B_{j} + \sum_{i}^{i_M}{
	I_i * W_{ij}
}) 

&&\end{flalign}
$$

Gradient Descend
===
- Can be sped up via various [[Deep Learning Optimizers | optimizers]]

Gradient generation
---
$$
\begin{flalign}

\gamma_{j} = A'(O_j) * \sum_{k}^{O^{+1}_M} {
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




