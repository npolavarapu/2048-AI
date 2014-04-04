# 2048 AI

AI for the game [2048](https://github.com/gabrielecirulli/2048).

See it in action [here](http://ov3y.github.io/2048-AI/). (Hit the auto-run button to let the AI attempt to solve it by itself)

My approach encodes the entire board (16 entries) as a single 64-bit integer (where tiles are the nybbles, i.e. 4-bit chunks). On a 64-bit machine, this enables the entire board to be passed around in a single machine register.

Bit shift operations are used to extract individual rows and columns. A single row or column is a 16-bit quantity, so a table of size 65536 can encode transformations which operate on a single row or column. For example, moves are implemented as 4 lookups into a precomputed "move effect table" which describes how each move affects a single row or column (for example, the "move right" table contains the entry "1122 -> 0023" describing how the row [2,2,4,4] becomes the row [0,0,4,8] when moved to the right).

Scoring is also done using table lookup. The tables contain heuristic scores computed on all possible rows/columns, and the resultant score for a board is simply the sum of the table values across each row and column.

This board representation, along with the table lookup approach for movement and scoring, allows the AI to search a huge number of game states in a short period of time (over 10,000,000 game states per second on my 3-year-old laptop (single-core)).

The expectimax search itself is coded as a recursive search which alternates between "expectation" steps (testing all possible tile spawn locations and values, and weighting their optimized scores by the probability of each possibility), and "maximization" steps (testing all possible moves and selecting the one with the best score). The tree search terminates when it sees a previously-seen position (using a transposition table), when it reaches a predefined depth limit, or when it reaches a board state that is highly unlikely (e.g. it was reached by getting 6 "4" tiles in a row from the starting position). The typical search depth is 4-8 moves.

The heuristic scoring algorithm is very straightforward: open squares count for +20000, and for each row and column, +20000 if its largest value is on the edge (as opposed to being in the two middle squares). The former heuristic performs extremely well by itself (usually getting to 4096), but frequently dies after the maximum piece ends up in the middle 4 tiles. Thus, the latter heuristic guides the largest pieces towards the edges, where they are easier to combine.
