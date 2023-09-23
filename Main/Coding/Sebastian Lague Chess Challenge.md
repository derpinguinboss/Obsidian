
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
	- [ ] -> start exploring random moves on a decreased velocity/ negative acceleration of the evaluation
		- [ ] -> cache the last move or something
- [ ] [[Delta Pruning]]
- [ ] [[Null Move Pruning]]
- [ ] Maybe [[NNUE]]
- [ ] Analyse [[Stock Fish]] search function
- [ ] TTEntry as class so it can be accessed by reference easily
- [ ] `TTEntryPtr` class as a wrapper for the `TTEntry` struct in order to reference as a pointer 
- [ ] ContinuationHistory
- [ ] For the evaluation function use a bitboard(ulong) as a bonus indicator for each piece.
- [ ] Add UCI support [Discord](https://discord.com/channels/1132289356011405342/1135297519098793984)

Known Issues/Bugs
===
- [x] Sometimes, when in check, it returns a null move, even though at least a search of depth 1 has been completed
- [ ] Just blundering queen in one move sort of stuff :/
- [ ] Losing almost every endgame
- [ ] TT worse when static
```
PV  6  'g1f3'(23|12)
PV  6  'g1f3'(12|12) 7  'g8f6'(-12|-23) 8  'Null'(-22|-22)
PV  6  'g1f3'(22|12) 7  'g8f6'(-22|-23) 8  'Null'(-11|-22)
PV  6  'g1f3'(12|12) 7  'g8f6'(-12|-23) 8  'Null'(12|-22) 9  'd7d6'(0|11)
PV  6  'g1f3'(19|12) 7  'g8f6'(-19|-23) 8  'Null'(16|-22) 9  'e5d4'(26|12) 10  'f1g2'(20|0) 11  'Null'(-22|-22) 12  'h1h4'(980|-5) 13  'Null'(-980|-980) 14  'h1h4'(987|18) 15  'Null'(-987|-987) 16  'Null'(-32|-32)
PV  6  'g1f3'(12|12) 7  'g8f6'(-12|-23) 8  'Null'(12|-22) 9  'e8e7'(-7|91) 10  'Null'(12|-83) 11  'g8f6'(163|152) 12  'Null'(-15|-87) 13  'f6g4'(-12|-23) 14  'Null'(12|-45) 15  'Null'(-62|-62) 16  'e4d2'(994|28) 17  'Null'(-1028|-1028) 18  'Null'(-289|-289) 19  'Null'(-91|-130) 20  'Null'(-2|-52)
PV  6  'g1f3'(16|12) 7  'g8f6'(-16|-23) 8  'Null'(16|-22) 9  'e8e7'(-15|91) 10  'Null'(15|-83) 11  'g8f6'(-11|72) 12  'Null'(-43|-136) 13  'f6g4'(-12|-23) 14  'Null'(12|-45) 15  'd8d5'(13|-55) 16  'Null'(-13|-13) 17  'c8f5'(241|-113) 18  'Null'(-241|-293) 19  'h1g1'(26|-197) 20  'Null'(-26|-78)
PV  6  'e2e4'(13|12) 7  'g8f6'(-12|-23) 8  'Null'(13|-22) 9  'e8e7'(-13|91) 10  'Null'(13|-83) 11  'g8f6'(21|152) 12  'Null'(-16|-136) 13  'f6g4'(111|43) 14  'Null'(-111|-111) 15  'f6g4'(-10|-78) 16  'Null'(10|10) 17  'c8g4'(694|-331) 18  'Null'(-694|-694) 19  'd8d5'(926|-57) 20  'Null'(-926|-926) 21  'Null'(-250|-250) 22  'Null'(-283|-283) 23  'a8c8'(904|-110) 24  'Null'(-904|-904) 25  'Null'(-285|-545) 26  'Null'(-31|-31) 27  'Null'(-325|-325)

D:\Programmmieren\Projects\Chess-Challenge\uci\Chess-Challenge-Uci-uci\Chess-Challenge-Uci-uci\Chess-Challenge\bin\Release\net6.0\Chess-Challenge.exe (process 3500) exited with code 0.
Press any key to close this window . . .
```


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


