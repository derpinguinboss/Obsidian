

Implementation
===
File
---
`D:\Sonstiges\stockfish\Stockfish-master\Stockfish-master\src\search.cpp`


Search Function
===
Uses a `SearchStack` struct to keep track of information related to the current state of the search.
An array of `Stack` structs keeps track of the information for each layer in the game tree (Plys).


Parameters
---
- A reference to the board object
- A pointer a stack instance. 
	>`Stack` struct keeps track of the information we need to remember from nodes shallower and deeper in the tree during the search. Each search thread has its own array of `Stack` objects, indexed by the current ply.
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
Use [[Draw Randomization]] to evaluate the draw.
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
If we ran out of time, return a very high value, so the node get's ignored.


4. __Immediate Draw Evaluation__
___
If the current node is not the root node and the board is either a drawn position, or the maximum number of plies has been reached, the node returns the following. 
If the current plies exceeds the maximum number of plies and the stack is not in check, we return a static evaluation of the position. However, if neither one of the two conditions has been met, we evaluate the position as a draw using [[Draw Randomization]]. 
```cpp
if (!rootNode
	&& (pos.is_draw(ss->ply)
		|| ss->ply >= MAX_PLY))
	return (ss->ply >= MAX_PLY && !ss->inCheck) 
		? evaluate(pos)
		: value_draw(pos.this_thread());
```


5. __Mate distance pruning__
___
At non root nodes.
Mate distance pruning. Even if we mate at the next move our score would be at best `mate_in(ss->ply+1)`, but if alpha is already bigger because a shorter mate was found upward in the tree then there is no need to search because we will never beat the current alpha. Same logic but with reversed signs apply also in the opposite condition of being mated instead of giving mate. In this case, return a fail-high score.
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


__Case 1: Node is excluded__
Initialize the static evaluation to the one stored in the `Stack`, and provide the hint, that this node's accumulator will be used often brings significant Elo gain. (What ever that means)
```cpp
if (excludedMove)
{
	// Providing the hint that this node's accumulator will be used often brings significant Elo gain (13 Elo)
	Eval::NNUE::hint_common_parent_position(pos);
	eval = ss->staticEval;
}
```

__Else Case 2: Node is stored in the transposition table__
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

__Else Case 3__
This will take affect if none of the cases above are valid. 
Here we just generate a new static evaluation of the position and make an entry in the transposition table.
```cpp
ss->staticEval = eval = evaluate(pos);
// Save static evaluation into the transposition table
tte->save(posKey, VALUE_NONE, ss->ttPv, BOUND_NONE, DEPTH_NONE, MOVE_NONE, eval);
```

__Case 4: Improving quiet [[Move Ordering]]__
If the last move was a check and not a capture, we do this (?)
```cpp
if (is_ok((ss-1)->currentMove) && !(ss-1)->inCheck && !priorCapture)
{
	int bonus = std::clamp(-18 * int((ss-1)->staticEval + ss->staticEval), -1817, 1817);
	thisThread->mainHistory[~us][from_to((ss-1)->currentMove)] << bonus;
}

void operator<<(int bonus) {
	assert(abs(bonus) <= D); // Ensure range is [-D, D]
	static_assert(D <= std::numeric_limits<T>::max(), "D overflows T");
	
	entry += bonus - entry * abs(bonus) / D;
	
	assert(abs(entry) <= D);
}
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
If eval is really low check with [[Quiescent Search]] if it can exceed alpha, if it can't, return a fail low.
```cpp
if (eval < alpha - 456 - 252 * depth * depth)
{
	value = qsearch<NonPV>(pos, ss, alpha - 1, alpha);
	if (value < alpha) return value;
}
```


9. __[[Futility Pruning]]__
___
The depth condition is important for mate finding.
> Futility pruning: child node
```cpp
if (  ! ss->ttPv
	&&  depth < 9
	&&  eval - futility_margin(depth, cutNode && !ss->ttHit, improving) - (ss-1)->statScore / 306 >= beta
	&&  eval >= beta
	&&  eval < 24923) // larger than VALUE_KNOWN_WIN, but smaller than TB wins
	return eval;
	
Value futility_margin(Depth d, bool noTtCutNode, bool improving) {
	return Value((140 - 40 * noTtCutNode) * (d - improving));
}
```


10. __[[Null Move Search]] with [[Verification Search]]__
___
[[Null Move Pruning]]
[[Extended Null Move Reductions]]

```cpp
if (   !PvNode
    && (ss-1)->currentMove != MOVE_NULL
    && (ss-1)->statScore < 17329
    &&  eval >= beta
    &&  eval >= ss->staticEval
    &&  ss->staticEval >= beta - 21 * depth + 258
    && !excludedMove
    &&  pos.non_pawn_material(us)
    &&  ss->ply >= thisThread->nmpMinPly
    &&  beta > VALUE_TB_LOSS_IN_MAX_PLY)
{
    // Null move dynamic reduction based on depth and eval
    Depth R = std::min(int(eval - beta) / 173, 6) + depth / 3 + 4;

    ss->currentMove = MOVE_NULL;
    ss->continuationHistory = &thisThread->continuationHistory[0][0][NO_PIECE][0];

    pos.do_null_move(st);

    Value nullValue = -search<NonPV>(pos, ss+1, -beta, -beta+1, depth-R, !cutNode);

    pos.undo_null_move();

    if (nullValue >= beta)
    {
        // Do not return unproven mate or TB scores
        nullValue = std::min(nullValue, VALUE_TB_WIN_IN_MAX_PLY-1);

        if (thisThread->nmpMinPly || depth < 14)
            return nullValue;

        assert(!thisThread->nmpMinPly); // Recursive verification is not allowed

        // Do verification search at high depths, with null move pruning disabled
        // until ply exceeds nmpMinPly.
        thisThread->nmpMinPly = ss->ply + 3 * (depth-R) / 4;

        Value v = search<NonPV>(pos, ss, beta-1, beta, depth-R, false);

        thisThread->nmpMinPly = 0;

        if (v >= beta)
            return nullValue;
    }
}
```



11. __Dynamic Depth Adjustment and [[ProbCut]] Preparation__
___
```cpp
// If the position doesn't have a ttMove, decrease depth by 2
// (or by 4 if the TT entry for the current position was hit and the stored depth is greater than or equal to the current depth).
// Use qsearch if depth is equal or below zero (~9 Elo)
if (    PvNode
	&& !ttMove)
	depth -= 2 + 2 * (ss->ttHit && tte->depth() >= depth);

if (depth <= 0)
	return qsearch<PV>(pos, ss, alpha, beta);

if (    cutNode
	&&  depth >= 8
	&& !ttMove)
	depth -= 2;
	
probCutBeta = beta + 168 - 61 * improving;
```



12. __[[ProbCut]]__
___
```cpp
// If we have a good enough capture (or queen promotion) and a reduced search returns a value
// much above beta, we can (almost) safely prune the previous move.
if (   !PvNode
	&&  depth > 3
	&&  abs(beta) < VALUE_TB_WIN_IN_MAX_PLY
	// If value from transposition table is lower than probCutBeta, don't attempt probCut
	// there and in further interactions with transposition table cutoff depth is set to depth - 3
	// because probCut search has depth set to depth - 4 but we also do a move before it
	// So effective depth is equal to depth - 3
	&& !(   tte->depth() >= depth - 3
		 && ttValue != VALUE_NONE
		 && ttValue < probCutBeta))
{
	assert(probCutBeta < VALUE_INFINITE);

	MovePicker mp(pos, ttMove, probCutBeta - ss->staticEval, &captureHistory);

	while ((move = mp.next_move()) != MOVE_NONE)
		if (move != excludedMove && pos.legal(move))
		{
			assert(pos.capture_stage(move));

			ss->currentMove = move;
			ss->continuationHistory = &thisThread->continuationHistory[ss->inCheck]
																	  [true]
																	  [pos.moved_piece(move)]
																	  [to_sq(move)];

			pos.do_move(move, st);

			// Perform a preliminary qsearch to verify that the move holds
			value = -qsearch<NonPV>(pos, ss+1, -probCutBeta, -probCutBeta+1);

			// If the qsearch held, perform the regular search
			if (value >= probCutBeta)
				value = -search<NonPV>(pos, ss+1, -probCutBeta, -probCutBeta+1, depth - 4, !cutNode);

			pos.undo_move(move);

			if (value >= probCutBeta)
			{
				// Save ProbCut data into transposition table
				tte->save(posKey, value_to_tt(value, ss->ply), ss->ttPv, BOUND_LOWER, depth - 3, move, ss->staticEval);
				return value;
			}
		}

	Eval::NNUE::hint_common_parent_position(pos);
}
```

`moves_loop` jump point here:
```cpp
// Step 12. A small Probcut idea, when we are in check (~4 Elo)
probCutBeta = beta + 413;
if (   ss->inCheck
	&& !PvNode
	&& ttCapture
	&& (tte->bound() & BOUND_LOWER)
	&& tte->depth() >= depth - 4
	&& ttValue >= probCutBeta
	&& abs(ttValue) <= VALUE_KNOWN_WIN
	&& abs(beta) <= VALUE_KNOWN_WIN)
	return probCutBeta;

const PieceToHistory* contHist[] = { (ss-1)->continuationHistory, (ss-2)->continuationHistory,
									  nullptr                   , (ss-4)->continuationHistory,
									  nullptr                   , (ss-6)->continuationHistory };

Move countermove = prevSq != SQ_NONE ? thisThread->counterMoves[pos.piece_on(prevSq)][prevSq] : MOVE_NONE;

MovePicker mp(pos, ttMove, depth, &thisThread->mainHistory,
								  &captureHistory,
								  contHist,
								  countermove,
								  ss->killers);

value = bestValue;
moveCountPruning = singularQuietLMR = false;

// Indicate PvNodes that will probably fail low if the node was searched
// at a depth equal to or greater than the current depth, and the result of this search was a fail low.
bool likelyFailLow =    PvNode
					 && ttMove
					 && (tte->bound() & BOUND_UPPER)
					 && tte->depth() >= depth;
```


13. __Loop through all pseudo-legal moves until no moves remain__
___
  1. `__Pruning at shallow depth__`
Depth conditions are important for mate finding.

```cpp
Depth reduction(bool i, Depth d, int mn, Value delta, Value rootDelta) {
  int r = Reductions[d] * Reductions[mn];
  return (r + 1372 - int(delta) * 1073 / int(rootDelta)) / 1024 + (!i && r > 936);
}
```

```cpp
if (  !rootNode
  && pos.non_pawn_material(us)
  && bestValue > VALUE_TB_LOSS_IN_MAX_PLY)
{
  // Skip quiet moves if movecount exceeds our FutilityMoveCount threshold (~8 Elo)
  moveCountPruning = moveCount >= futility_move_count(improving, depth);

  // Reduced depth of the next LMR search
  int lmrDepth = newDepth - r;

  if (   capture
	  || givesCheck)
  {
	  // Futility pruning for captures (~2 Elo)
	  if (   !givesCheck
		  && lmrDepth < 7
		  && !ss->inCheck
		  && ss->staticEval + 197 + 248 * lmrDepth + PieceValue[EG][pos.piece_on(to_sq(move))]
		   + captureHistory[movedPiece][to_sq(move)][type_of(pos.piece_on(to_sq(move)))] / 7 < alpha)
		  continue;

	  Bitboard occupied;
	  // SEE based pruning (~11 Elo)
	  if (!pos.see_ge(move, occupied, Value(-205) * depth))
	  {
		 if (depth < 2 - capture)
			continue;
		 // Don't prune the move if opponent Queen/Rook is under discovered attack after the exchanges
		 // Don't prune the move if opponent King is under discovered attack after or during the exchanges
		 Bitboard leftEnemies = (pos.pieces(~us, KING, QUEEN, ROOK)) & occupied;
		 Bitboard attacks = 0;
		 occupied |= to_sq(move);
		 while (leftEnemies && !attacks)
		 {
			  Square sq = pop_lsb(leftEnemies);
			  attacks |= pos.attackers_to(sq, occupied) & pos.pieces(us) & occupied;
			  // Don't consider pieces that were already threatened/hanging before SEE exchanges
			  if (attacks && (sq != pos.square<KING>(~us) && (pos.attackers_to(sq, pos.pieces()) & pos.pieces(us))))
				 attacks = 0;
		 }
		 if (!attacks)
			continue;
	  }
  }
  else
  {
	  int history =   (*contHist[0])[movedPiece][to_sq(move)]
					+ (*contHist[1])[movedPiece][to_sq(move)]
					+ (*contHist[3])[movedPiece][to_sq(move)];

	  // Continuation history based pruning (~2 Elo)
	  if (   lmrDepth < 6
		  && history < -3832 * depth)
		  continue;

	  history += 2 * thisThread->mainHistory[us][from_to(move)];

	  lmrDepth += history / 7011;
	  lmrDepth = std::max(lmrDepth, -2);

	  // Futility pruning: parent node (~13 Elo)
	  if (   !ss->inCheck
		  && lmrDepth < 12
		  && ss->staticEval + 112 + 138 * lmrDepth <= alpha)
		  continue;

	  lmrDepth = std::max(lmrDepth, 0);

	  // Prune moves with negative SEE (~4 Elo)
	  if (!pos.see_ge(move, Value(-27 * lmrDepth * lmrDepth - 16 * lmrDepth)))
		  continue;
  }
}
```


	2. __Extensions__
We take care to not overdo to avoid search getting stuck.
```cpp
if (ss->ply < thisThread->rootDepth * 2)
{
	// Singular extension search (~94 Elo). If all moves but one fail low on a
	// search of (alpha-s, beta-s), and just one fails high on (alpha, beta),
	// then that move is singular and should be extended. To verify this we do
	// a reduced search on all the other moves but the ttMove and if the
	// result is lower than ttValue minus a margin, then we will extend the ttMove.
	// Depth margin and singularBeta margin are known for having non-linear scaling.
	// Their values are optimized to time controls of 180+1.8 and longer
	// so changing them requires tests at this type of time controls.
	if (   !rootNode
		&&  depth >= 4 - (thisThread->completedDepth > 22) + 2 * (PvNode && tte->is_pv())
		&&  move == ttMove
		&& !excludedMove // Avoid recursive singular search
	 /* &&  ttValue != VALUE_NONE Already implicit in the next condition */
		&&  abs(ttValue) < VALUE_KNOWN_WIN
		&& (tte->bound() & BOUND_LOWER)
		&&  tte->depth() >= depth - 3)
	{
		Value singularBeta = ttValue - (82 + 65 * (ss->ttPv && !PvNode)) * depth / 64;
		Depth singularDepth = (depth - 1) / 2;
	
		ss->excludedMove = move;
		value = search<NonPV>(pos, ss, singularBeta - 1, singularBeta, singularDepth, cutNode);
		ss->excludedMove = MOVE_NONE;
	
		if (value < singularBeta)
		{
			extension = 1;
			singularQuietLMR = !ttCapture;
	
			// Avoid search explosion by limiting the number of double extensions
			if (  !PvNode
				&& value < singularBeta - 21
				&& ss->doubleExtensions <= 11)
			{
				extension = 2;
				depth += depth < 13;
			}
		}
	
		// Multi-cut pruning
		// Our ttMove is assumed to fail high, and now we failed high also on a reduced
		// search without the ttMove. So we assume this expected Cut-node is not singular,
		// that multiple moves fail high, and we can prune the whole subtree by returning
		// a softbound.
		else if (singularBeta >= beta)
			return singularBeta;
	
		// If the eval of ttMove is greater than beta, we reduce it (negative extension) (~7 Elo)
		else if (ttValue >= beta)
			extension = -2 - !PvNode;
	
		// If we are on a cutNode, reduce it based on depth (negative extension) (~1 Elo)
		else if (cutNode)
			extension = depth > 8 && depth < 17 ? -3 : -1;
	
		// If the eval of ttMove is less than value, we reduce it (negative extension) (~1 Elo)
		else if (ttValue <= value)
			extension = -1;
	}
	
	// Check extensions (~1 Elo)
	else if (   givesCheck
			 && depth > 9)
		extension = 1;
	
	// Quiet ttMove extensions (~1 Elo)
	else if (   PvNode
			 && move == ttMove
			 && move == ss->killers[0]
			 && (*contHist[0])[movedPiece][to_sq(move)] >= 5168)
		extension = 1;
}


// Add extension to new depth
newDepth += extension;
ss->doubleExtensions = (ss-1)->doubleExtensions + (extension == 2);

// Speculative prefetch as early as possible
prefetch(TT.first_entry(pos.key_after(move)));

// Update the current move (this must be done after singular extension search)
ss->currentMove = move;
ss->continuationHistory = &thisThread->continuationHistory[ss->inCheck]
														[capture]
														[movedPiece]
														[to_sq(move)];
```


	3. __Make the move__
```cpp
pos.do_move(move, st, givesCheck);
```

Decrease reduction if position is or has been on the [[Principle Variation]] and node is not likely to fail low. (~3 Elo) 
Decrease further on [[Cut Node]]s. (~1 Elo)
```cpp
if (   ss->ttPv
  && !likelyFailLow)
  r -= cutNode && tte->depth() >= depth + 3 ? 3 : 2;
```

Decrease reduction if opponent's move count is high (~1 Elo)
```cpp
if ((ss-1)->moveCount > 8) r--;
```

Increase reduction for [[Cut Node]]s(~3 Elo)
```cpp
if (cutNode) r += 2;
```

Increase reduction if ttMove is a capture (~3 Elo)
```cpp
if (ttCapture) r++;
```

Decrease reduction for PvNodes based on depth (~2 Elo)
```cpp
if (PvNode) r -= 1 + (depth < 6);
```

Decrease reduction if ttMove has been singularly extended (~1 Elo)
```cpp
if (singularQuietLMR) r--;
```

Increase reduction if next ply has a lot of fail high (~5 Elo)
```cpp
if ((ss+1)->cutoffCnt > 3) r++;

else if (move == ttMove) r--;
```

```cpp
ss->statScore =  2 * thisThread->mainHistory[us][from_to(move)]
               + (*contHist[0])[movedPiece][to_sq(move)]
               + (*contHist[1])[movedPiece][to_sq(move)]
               + (*contHist[3])[movedPiece][to_sq(move)]
               - 4006;
```

Decrease/increase reduction for moves with a good/bad history (~25 Elo)
```cpp
r -= ss->statScore / (11124 + 4740 * (depth > 5 && depth < 22));
```


	4. __[[Late Move Reduction]]__
```cpp
// We use various heuristics for the sons of a node after the first son has
// been searched. In general, we would like to reduce them, but there are many
// cases where we extend a son if it has good chances to be "interesting".
if (    depth >= 2
    &&  moveCount > 1 + (PvNode && ss->ply <= 1)
    && (   !ss->ttPv
        || !capture
        || (cutNode && (ss-1)->moveCount > 1)))
{
    // In general we want to cap the LMR depth search at newDepth, but when
    // reduction is negative, we allow this move a limited search extension
    // beyond the first move depth. This may lead to hidden double extensions.
    Depth d = std::clamp(newDepth - r, 1, newDepth + 1);

    value = -search<NonPV>(pos, ss+1, -(alpha+1), -alpha, d, true);

    // Do a full-depth search when reduced LMR search fails high
    if (value > alpha && d < newDepth)
    {
        // Adjust full-depth search based on LMR results - if the result
        // was good enough search deeper, if it was bad enough search shallower
        const bool doDeeperSearch = value > (bestValue + 64 + 11 * (newDepth - d));
        const bool doEvenDeeperSearch = value > alpha + 711 && ss->doubleExtensions <= 6;
        const bool doShallowerSearch = value < bestValue + newDepth;

        ss->doubleExtensions = ss->doubleExtensions + doEvenDeeperSearch;

        newDepth += doDeeperSearch - doShallowerSearch + doEvenDeeperSearch;

        if (newDepth > d)
            value = -search<NonPV>(pos, ss+1, -(alpha+1), -alpha, newDepth, !cutNode);

        int bonus = value <= alpha ? -stat_bonus(newDepth)
                  : value >= beta  ?  stat_bonus(newDepth)
                                   :  0;

        update_continuation_histories(ss, movedPiece, to_sq(move), bonus);
    }
}
```


	5. __Full-depth search when LMR is skipped__
If expected reduction is high, reduce its depth by 1.
```cpp
else if (!PvNode || moveCount > 1)
{
    // Increase reduction for cut nodes and not ttMove (~1 Elo)
    if (!ttMove && cutNode)
        r += 2;

    value = -search<NonPV>(pos, ss+1, -(alpha+1), -alpha, newDepth - (r > 3), !cutNode);
}
```

```cpp
// For PV nodes only, do a full PV search on the first move or after a fail
// high (in the latter case search only if value < beta), otherwise let the
// parent node fail low with value <= alpha and try another move.
if (PvNode && (moveCount == 1 || (value > alpha && (rootNode || value < beta))))
{
  (ss+1)->pv = pv;
  (ss+1)->pv[0] = MOVE_NONE;

  value = -search<PV>(pos, ss+1, -beta, -alpha, newDepth, false);
}
```


	6. __Undo Move__
```cpp
pos.undo_move(move);
```


	7. __Check for a new best move___
```cpp
// Finished searching the move. If a stop occurred, the return value of
// the search cannot be trusted, and we return immediately without
// updating best move, PV and TT.
if (Threads.stop.load(std::memory_order_relaxed))
  return VALUE_ZERO;

if (rootNode)
{
  RootMove& rm = *std::find(thisThread->rootMoves.begin(),
							thisThread->rootMoves.end(), move);

  rm.averageScore = rm.averageScore != -VALUE_INFINITE ? (2 * value + rm.averageScore) / 3 : value;

  // PV move or new best move?
  if (moveCount == 1 || value > alpha)
  {
	  rm.score =  rm.uciScore = value;
	  rm.selDepth = thisThread->selDepth;
	  rm.scoreLowerbound = rm.scoreUpperbound = false;

	  if (value >= beta)
	  {
		  rm.scoreLowerbound = true;
		  rm.uciScore = beta;
	  }
	  else if (value <= alpha)
	  {
		  rm.scoreUpperbound = true;
		  rm.uciScore = alpha;
	  }

	  rm.pv.resize(1);

	  assert((ss+1)->pv);

	  for (Move* m = (ss+1)->pv; *m != MOVE_NONE; ++m)
		  rm.pv.push_back(*m);

	  // We record how often the best move has been changed in each iteration.
	  // This information is used for time management. In MultiPV mode,
	  // we must take care to only do this for the first PV line.
	  if (   moveCount > 1
		  && !thisThread->pvIdx)
		  ++thisThread->bestMoveChanges;
  }
  else
	  // All other moves but the PV, are set to the lowest value: this
	  // is not a problem when sorting because the sort is stable and the
	  // move position in the list is preserved - just the PV is pushed up.
	  rm.score = -VALUE_INFINITE;
}

if (value > bestValue)
{
  bestValue = value;

  if (value > alpha)
  {
	  bestMove = move;

	  if (PvNode && !rootNode) // Update pv even in fail-high case
		  update_pv(ss->pv, move, (ss+1)->pv);

	  if (value >= beta)
	  {
		  ss->cutoffCnt += 1 + !ttMove;
		  assert(value >= beta); // Fail high
		  break;
	  }
	  else
	  {
		  // Reduce other moves if we have found at least one score improvement (~2 Elo)
		  if (   depth > 2
			  && depth < 12
			  && beta  <  14362
			  && value > -12393)
			  depth -= 2;

		  assert(depth > 0);
		  alpha = value; // Update alpha! Always alpha < beta
	  }
  }
}


// If the move is worse than some previously searched move, remember it, to update its stats later
if (move != bestMove)
{
  if (capture && captureCount < 32)
	  capturesSearched[captureCount++] = move;

  else if (!capture && quietCount < 64)
	  quietsSearched[quietCount++] = move;
}
```


14. __Check for mate and stalemate__
___
```cpp
// All legal moves have been searched and if there are no legal moves, it
// must be a mate or a stalemate. If we are in a singular extension search then
// return a fail low score.

assert(moveCount || !ss->inCheck || excludedMove || !MoveList<LEGAL>(pos).size());

if (!moveCount)
	bestValue = excludedMove ? alpha :
				ss->inCheck  ? mated_in(ss->ply)
							 : VALUE_DRAW;

// If there is a move that produces search value greater than alpha we update the stats of searched moves
else if (bestMove)
	update_all_stats(pos, ss, bestMove, bestValue, beta, prevSq,
					 quietsSearched, quietCount, capturesSearched, captureCount, depth);

// Bonus for prior countermove that caused the fail low
else if (!priorCapture && prevSq != SQ_NONE)
{
	int bonus = (depth > 5) + (PvNode || cutNode) + (bestValue < alpha - 113 * depth) + ((ss-1)->moveCount > 12);
	update_continuation_histories(ss-1, pos.piece_on(prevSq), prevSq, stat_bonus(depth) * bonus);
}

if (PvNode)
	bestValue = std::min(bestValue, maxValue);

// If no good move is found and the previous position was ttPv, then the previous
// opponent move is probably good and the new position is added to the search tree. (~7 Elo)
if (bestValue <= alpha)
	ss->ttPv = ss->ttPv || ((ss-1)->ttPv && depth > 3);

// Write gathered information in transposition table
if (!excludedMove && !(rootNode && thisThread->pvIdx))
	tte->save(posKey, value_to_tt(bestValue, ss->ply), ss->ttPv,
			  bestValue >= beta ? BOUND_LOWER :
			  PvNode && bestMove ? BOUND_EXACT : BOUND_UPPER,
			  depth, bestMove, ss->staticEval);
```



