
Sources
===
[GitHub Repo](https://github.com/SebLague/Chess-Challenge), [YouTube Introduction](https://www.youtube.com/watch?v=iScy18pVR58)


Todo
===
- [ ] Update API version to 1.19
- [ ] On Initialization (constructor) decode the compressed data and cache in memory. Thus it doesn't have to be decompressed on runtime of the search, wasting computation time
- [ ] [[Piece Square Tables]]
	- [ ] Switch from jagged array to a single array
	- [ ] Bump up the king value (By some factor, not additive), this might improve the [[King Safety]]  in the [[Middle Game]] as well
	 - [ ] Better [[PSQT Compression]]
- [ ] [[Opening Book]]
	- [ ] [[PSQT Compression]]: The length of a number does not increase the token count
- [ ] [[End Game]]: Doesn't know how to win endgames
	- [x] -> [[Transposition Table]] !!!
	- [ ] -> If this is not enough, start exploring random moves on a decreased velocity/ negative acceleration of the evaluation
		- [ ] -> cache the last move or something
- [ ] [[Delta Pruning]]
- [ ] [[Null Move Pruning]]
- [ ] Maybe [[NNUE]]

