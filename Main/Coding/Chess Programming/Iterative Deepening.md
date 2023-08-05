Instead of a fixed depth, we search until we run out of time.

```csharp
for (int depth = 1 ; /*Last search took too long*/ ; depth++) {
	1.
	2.
}
```


1. Perform [[Alpha Beta]] search 

2. Sorting the moves after each iteration via [[Principle Variation]] allows for better Alpha Beta performance

*Note:* Probably very dependent on a good [[Transposition Table]] handling, (caching the depth etc.) (?) 
