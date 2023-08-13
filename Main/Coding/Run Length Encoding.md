
Sources
---
[Run-length encoding - Wikipedia](https://en.wikipedia.org/wiki/Run-length_encoding)


Summary
---
Good for data with a lot of _runs_ (sequences in which the same data value occurs in many consecutive data elements). These runs are stored as follows: `[Count][DataValue]` 
This compression is lossless.


Example
---
Input data: 
`WWWWWWWWWWWWBWWWWWWWWWWWWBBBWWWWWWWWWWWWWWWWWWWWWWWWBWWWWWWWWWWWWWW`

Encoded data: 
`12W1B12W3B24W1B14W`

