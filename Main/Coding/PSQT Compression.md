
Sources
---
- [Data compression - Wikipedia](https://en.wikipedia.org/wiki/Data_compression#Encoding_theory)
- [How are Images Compressed? JPEG In Depth - YouTube](https://www.youtube.com/watch?v=Kv1Hiv3ox8I)


Brain storming
---
- Multiple Values stored in one Big Integer.
	- Maybe even use Int8, not Int16 foreach uInt64
	- Decimal is a 128 bit number that is allowed to use, can represent more values that uInt64
- [[Run Length Encoding]]
- [[Delta Compression]]
- Using [[Delta Compression]], we can make the numbers smaller, e.g. they will fit into one byte, and we can bundle more numbers in one big number
- [[Fourier transform]]
- We CANNOT sort the Fourier coefficients directly, but if we could: might allow for other compression  (e.g. [[Delta Compression]]) algorithms to perform better on the Fourier compression
- Use [[Discrete Cosine Transform]] rather than [[Discrete Fourier Transform]]
	- Gives real coefficients, better for token storing 
	- Encoding: [[DCT-III]]
	- Decoding: [[DCT-II]] with a factor of $\frac{2}{N}$
- Use [[Huffman Encoding]] to store the coefficients (decoding implementation would take up too much tokens to be worth it)
- [[Byte Pair Encoding]]? (rather not)


Plan
---
Forward: 
1. Delta Compression, for smaller numbers
2. DCT on all but the first value
3. Frequency thresholding on the DCT coefficients
4. TBD: After the thresholding, were gonna have a lot of zeros -> RLE?
5. … Semantic token reduction

Inverse:



Goals
---
- Small Numbers
- Few Numbers
- Easy to decompress (few tokens)


Todo
---
- Test out the compression capabilities of 128Bit floating point number Decimal
- Look at how zip files do compression


Guidance rules
---
1. Data Analysis/Simplification
2. THEN Semantic token reduction

