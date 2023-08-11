
Sources
===
- [Diskrete Kosinustransformation – Wikipedia](https://de.wikipedia.org/wiki/Diskrete_Kosinustransformation)
- [taramath: Diskrete Kosinustransformation](https://www.taramath.de/tools/cosine)
- [ChatGPT](https://chat.openai.com/c/588b4409-24b4-40db-8dc0-bf2487ce1927)


Description
===
A simpler version of the [[Discrete Fourier Transform]]. Good for data compression, as it tends to use fewer coefficients that [[Fourier transform]].


Difference: [[Discrete Fourier Transform]]
===
Frequency Components
---
- DFT: sinusoidal waves (Both Sine and Cosine)
- DCT: Cosine waves

Energy Concentration
---
- DFT: spreads the signal energy across both sine and cosine components. This means that for signals that are not purely periodic, the energy can be distributed across multiple coefficients, potentially requiring more coefficients to accurately represent the signal.
- DCT: The DCT tends to concentrate the signal energy in a smaller number of coefficients, also due to symmetry

Symmetry
---
- DCT: The DCT are real and symmetric
	- Easier implementation
	- Fewer coefficients

Applications
---
- DFT: Data analysis 
- DCT: Data compression (e.g. [[Jpeg Compression]], [[Mp3 Compression]])

Equations
---
- DFT: Complex, exponential functions. Uses both sine and cosine terms. 
- DCT: Defined with only cosine terms; Different types of DCT (e.g., DCT-I, DCT-II, DCT-III, DCT-IV)


Different types of DCT
===
- [[DCT-I]]
- [[DCT-II]]
- [[DCT-III]]
- [[DCT-IV]]

