

Implementation
===
File
---
`D:\Sonstiges\stockfish\Stockfish-master\Stockfish-master\src\search.cpp`


Search Function
===
Uses a `SearchStack` struct to keep track of information related to the current state of the search.

Parameters
---
- A reference to the board object
- A pointer a stack instance. 
	>Stack struct keeps track of the information we need to remember from nodes shallower and deeper in the tree during the search. Each search thread has its own array of Stack objects, indexed by the current ply.
```cpp
struct Stack {
  Move* pv;
  PieceToHistory* continuationHistory;
  int ply;
  Move currentMove;
  Move excludedMove;
  Move killers[2];
  Value staticEval;
  int statScore;
  int moveCount;
  bool inCheck;
  bool ttPv;
  bool ttHit;
  int doubleExtensions;
  int cutoffCnt;
};
```
- alpha value
- beta value
- depth
- Whether the node is a [[Cut Node]] or not


Important Steps
---

1. __Repetition__
___
Check if we have an upcoming move that draws by repetition, or if the opponent had an alternative move earlier to this position.
- [[Draw Randomization]]: [engines - Stockfish draw value randomization and "3-fold blindness" - Chess Stack Exchange](https://chess.stackexchange.com/questions/29530/stockfish-draw-value-randomization-and-3-fold-blindness?newreg=c2bbd48616174497aa9191e57a7a6075)
- 50 Move rule 
- 3-fold-repitition
```cpp
if (   !rootNode
	&& pos.rule50_count() >= 3
	&& alpha < VALUE_DRAW
	&& pos.has_game_cycle(ss->ply))
{
	alpha = value_draw(pos.this_thread());
	if (alpha >= beta)
		return alpha;
}
```


2. __Quiescence search__
___
Dive into [[Quiescent Search]] when the depth reaches zero.
```cpp
if (depth <= 0)
	return qsearch<PvNode ? PV : NonPV>(pos, ss, alpha, beta);
```


3. __Check Time___
___
Return if a time limit has been reached and the current node is not the root.


4. __Immediate Draw Evaluation__
___
If the current node is not the root node and the board is either a drawn position or the maximum number of plies has been reached the node returns the following. 
If the current plies exceeds the maximum number of plies and the stack is not in check, we return a static evaluation of the position. However, if neither one of the two conditions has been met, we evaluate the position as a draw using [[Draw Randomization]]. 
```cpp
if (   !rootNode
	|| pos.is_draw(ss->ply)
	|| ss->ply >= MAX_PLY)
	return (ss->ply >= MAX_PLY && !ss->inCheck) 
		? evaluate(pos)
		: value_draw(pos.this_thread());
```


5. __Mate distance pruning__
___
Step 3. Mate distance pruning. Even if we mate at the next move our score would be at best mate_in(ss->ply+1), but if alpha is already bigger because a shorter mate was found upward in the tree then there is no need to search because we will never beat the current alpha. Same logic but with reversed signs apply also in the opposite condition of being mated instead of giving mate. In this case, return a fail-high score.
```cpp
alpha = max(mated_in(ss->ply), alpha);
beta = min(mate_in(ss->ply+1), beta);
if (alpha >= beta)
	return alpha;
```


6. __Transposition table lookup__
___
If this move should be excluded, skip the remaining code of this step.
```cpp
if (!excludedMove)
```

Determine whether this line is a [[Principle Variation]]-node or not.
```cpp
ss->ttPv = PvNode || (ss->ttHit && tte->is_pv());
```

If it is not, check for an early [[Transposition Table]] cutoff.
```cpp
if (  !PvNode
        && tte->depth() > depth
        && ttValue != VALUE_NONE // Possible in case of TT access race or if !ttHit
        && (tte->bound() & (ttValue >= beta ? BOUND_LOWER : BOUND_UPPER)))
{
	// If ttMove is quiet, update move sorting heuristics on TT hit (~2 Elo)
	if (ttMove)
	{
		if (ttValue >= beta)
		{
			// Bonus for a quiet ttMove that fails high (~2 Elo)
			if (!ttCapture)
				update_quiet_stats(pos, ss, ttMove, stat_bonus(depth));

			// Extra penalty for early quiet moves of the previous ply (~0 Elo on STC, ~2 Elo on LTC)
			if (prevSq != SQ_NONE && (ss-1)->moveCount <= 2 && !priorCapture)
				update_continuation_histories(ss-1, pos.piece_on(prevSq), prevSq, -stat_bonus(depth + 1));
		}
		// Penalty for a quiet ttMove that fails low (~1 Elo)
		else if (!ttCapture)
		{
			int penalty = -stat_bonus(depth);
			thisThread->mainHistory[us][from_to(ttMove)] << penalty;
			update_continuation_histories(ss, pos.moved_piece(ttMove), to_sq(ttMove), penalty);
		}
	}

	// Partial workaround for the graph history interaction problem
	// For high rule50 counts don't produce transposition table cutoffs.
	if (pos.rule50_count() < 90)
		return ttValue;
}
```


(6.5 __[[Tablebases]] lookup__)
___
Endgame solutions


7. __[[Static evaluation]] and improvement flag__
___
This step handles different cases, that apply if the node is not in check and which are dependent on the static evaluation of the position. 


__Case 1: Node is excluded from (?)__
Initialize the static evaluation to the one stored in the `Stack`, and provide the hint, that this node's accumulator will be used often brings significant Elo gain. (What ever that means)
```cpp
if (excludedMove)
{
	// Providing the hint that this node's accumulator will be used often brings significant Elo gain (13 Elo)
	Eval::NNUE::hint_common_parent_position(pos);
	eval = ss->staticEval;
}
```

__Case 2: Node is stored in the transposition table__
Initialize the static evaluation of the node and the `Stack` the to the one cached in the TT (or generate a new static evaluation if the cached eval does not exist). 
If none of these conditions are true and the node is a [[Principle Variation]] node, we provide the hint that this node's accumulator will be used often, like in case 1.
```cpp
if (ss->ttHit)
{
	// Never assume anything about values stored in TT
	ss->staticEval = eval = tte->eval();
	if (eval == VALUE_NONE)
		ss->staticEval = eval = evaluate(pos);
	else if (PvNode)
		Eval::NNUE::hint_common_parent_position(pos);

	// ttValue can be used as a better position evaluation (~7 Elo)
	if (    ttValue != VALUE_NONE
		&& (tte->bound() & (ttValue > eval ? BOUND_LOWER : BOUND_UPPER)))
		eval = ttValue;
}
```

__Case 3__
This will take affect if none of the cases above are valid. 
Here we just generate a new static evaluation of the position and make an entry in the transposition table.
```cpp
ss->staticEval = eval = evaluate(pos);
// Save static evaluation into the transposition table
tte->save(posKey, VALUE_NONE, ss->ttPv, BOUND_NONE, DEPTH_NONE, MOVE_NONE, eval);
```

__Improving flag__
It also sets up to `improving` flag, which is true if current static evaluation is bigger than the previous static evaluation at our turn (if we were in check at our previous move we look at static evaluation at move prior to it and if we were in check at move prior to it flag is set to true) and is false otherwise. The improving flag is used in various pruning heuristics. 
```cpp
bool improving =  (ss-2)->staticEval != VALUE_NONE ? ss->staticEval > (ss-2)->staticEval
				: (ss-4)->staticEval != VALUE_NONE ? ss->staticEval > (ss-4)->staticEval
				: true;
```
When we are in check, we skip all moves up until step 13 and initialize the `improving` flag to false.
```cpp
if (ss->inCheck)
{
    // Skip early pruning when in check
    ss->staticEval = eval = VALUE_NONE;
    improving = false;
    goto moves_loop;
}
```


8. __[[Razoring]]__
___


9. __[[Futility Pruning]]__
___


10. __[[Null Move Search]] with [[Verification Search]]__
___
[[Null Move Pruning]]?


11. __Dynamic Depth Adjustment and [[ProbCut]] Preparation__
___


12. __[[ProbCut]]__
___

_ProbCut improvement_


13. __Loop through all pseudo-legal moves until no moves remain__
___


14. __Pruning at shallow depth__
___


15. __Extensions__
___


16. __Make the move__
___


17. __[[Late Move Reduction]]__
___


18. __Full-depth search when LMR is skipped__
___


19. __Undo Move__
___


20. __Check for a new best move___
___


21. Check for mate and stalemate
___



