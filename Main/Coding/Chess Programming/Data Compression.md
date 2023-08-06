- Multiple Values stored in one Big Integer.
	- Maybe even use Int8, not Int16 foreach uInt64
	- Decimal is a 128 bit number that is allowed to use, can represent more values that uInt64
- [[Run Length Encoding]]
- [[Delta Compression]]
- Using [[Delta Compression]], we can make the numbers smaller, e.g. they will fit into one byte, and we can bundle more numbers in one big number


Guide:
1. Data Analysis/Simplification
2. THEN Semantic token reduction

