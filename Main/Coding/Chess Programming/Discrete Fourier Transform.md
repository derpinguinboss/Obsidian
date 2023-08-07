
Sources
---
- [The Discrete Fourier Transform (DFT) - YouTube](https://www.youtube.com/watch?v=nl9TZanwbBk&list=PLMrJAkhIeNNT_Xh3Oy0Y4LTj0Oxo8GqsC&index=15)


Description
---
Useful if we have discrete data points/ a vector of data.


Mathematical description
---
Data to Fourier Coefficients:
$$
\hat{f_k} = \sum_{j=0}^{n-1}{f_j e^{-i 2 \pi \frac{k}{n}}}
$$

Fourier Coefficients To Data:
$$
{f_k} = \frac{1}{n} \sum_{j=0}^{n-1}{\hat{f}_je^{i2 \pi \frac{k}{n}}}
$$


Implementation
---
[[Fast Fourier Transform]]
 