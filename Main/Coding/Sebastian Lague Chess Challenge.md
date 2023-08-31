
Sources
===
[GitHub Repo](https://github.com/SebLague/Chess-Challenge), [YouTube Introduction](https://www.youtube.com/watch?v=iScy18pVR58)


Todo
===
- [ ] On Initialization (constructor) decode the compressed data and cache in memory. Thus it doesn't have to be decompressed on runtime of the search, wasting computation time
- [ ] [[Piece Square Tables]]
	- [ ] Switch from jagged array to a single array
	- [ ] Bump up the king value (By some factor, not additive), this might improve the [[King Safety]]  in the [[Middle Game]] as well
	- [ ] Better [[PSQT Compression]]
	- [ ] Use [[PeSTO]]
- [ ] [[Opening Book]]
	- [ ] [[PSQT Compression]]: The length of a number does not increase the token count
- [ ] [[End Game]]: Doesn't know how to win endgames
	- [x] -> [[Transposition Table]]
	- [ ] -> If this is not enough, start exploring random moves on a decreased velocity/ negative acceleration of the evaluation
		- [ ] -> cache the last move or something
- [ ] [[Delta Pruning]]
- [ ] [[Null Move Pruning]]
- [ ] Maybe [[NNUE]]
- [ ] Analyse [[Stock Fish]] search function
- [ ] TTEntry as class so it can be accessed by reference easily
- [ ] `TTEntryPtr` class as a wrapper for the `TTEntry` struct in order to reference as a pointer 
- [ ] ContinuationHistory
- [ ] For the evaluation function use a bitboard(ulong) as a bonus indicator for each piece.


Sketching
===
![[Untitled.txt]]
```
index: 1, sizeofT: 5
00000 11111 00000
11111 10101 01100
<<
10101 01100 00000
>>
00000 00000 10101


This does not work. dunno why :(
index: 1, sizeofT: 5
00000000_00000000_00000000_00000000_00000000_00000000_00000011_11100000
00000000_00000000_00000000_00000000_00001000_01101000_10010010_10001000
<< 64 - index * sizeofT - sizeofT = 54
10100010_00000000_00000000_00000000_00000000_00000000_00000000_00000000
>>> 64 - index * sizeofT = 59
00000000_00000000_00000000_00000000_00000000_00000000_00000000_00010100



tools:
http://www.convertalot.com/bitwise_operators.html
https://www.rapidtables.com/convert/number/binary-to-decimal.html

truthT:
https://www.google.com/search?sca_esv=561755169&sxsrf=AB5stBhr-SqG4zw8cshdpxeDyM_lWiFOzA:1693521538270&q=bitwise+operators+truth+table+C%23&tbm=isch&source=lnms&sa=X&ved=2ahUKEwil2dSt-4eBAxVqh_0HHTJeAdAQ0pQJegQIChAB&biw=1920&bih=923&dpr=1#imgrc=GscWmxkhria6WM
```

![[1111100000_.txt]]
```
1111100000
1010101100
0000001100

0,1 -> 1
1,0 -> 0
1,1 -> 0
0,0 -> 0
```

![[1111100000.txt]]
```
1111100000
0000011111

1010101100

0000001100

0,1 -> 0
1,0 -> 0
1,1 -> 1
0,0 -> 0
```


