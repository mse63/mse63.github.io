---
title: Chess AI
layout: template
filename: chess
order: 2
show_tab: 1
--- 

# Chess AI
I created a Chess AI in Java with no pre-made libraries. Because I am a better coder than I am a chess player, my AI beats me in chess every time.

## Example Game
Below is an example game of my AI (White) vs 2000 Elo Stockfish (Black):

<iframe src="https://lichess.org/embed/BHLqEa1J#0?theme=auto&bg=auto"
width=600 height=397 frameborder=0></iframe>

## Algorithm

### Overview
My chess AI uses a [minimax](https://en.wikipedia.org/wiki/Minimax) algorithm with alpha beta pruning. In order to find the best move, the root node, representing the current game state, tells its children nodes, representing possible future game states, to find the best move and score how good that position is for the opponent. The root node then takes the move that results in the worst possible position for the opponent, as that would be the best position for us.

The score of a position is the opposite of the lowest opponent score of any of its children, because that is the optimal position the AI can force its opponent into.

### Evaluation Function
If the position is a draw (stalemate or 3-fold-repition), my AI evalutes it as a 0. The position is no better for either player.

If the position is a checkmate, my AI originally returned the minimum possible value it can. This led to two issues:

1. The AI would not "stall" and continue playing in forced mating positions. It didn't see losing in 1 move as worse than a guaranteed loss in 10 moves.
2. More importantly, the AI would not checkmate the opponent if it knew it could on a future turn. It didn't see any difference between winning immediately and winning 3 moves from now. This led to the awkward situation where the AI would stall forever and make nonsense moves when it had a mate-in-1, because it saw itself as winning anyway.

To fix these issues, I added the length of the game to the smallest possible value for evaluating checkmates, so a checkmate further along the game is seen as better for the loser.

However, we can't always evaluate all the way until we hit a checkmate or draw. Chess has more possible games than there are atoms in the universe so, unlike some games such as Tic-Tac-Toe, it is computationally impossible to calculate every potential future position with the algorithm described above. Therefore, after some finite depth, we need some method of scoring a board that does not rely on looking at future moves.

My AI mainly looks at material difference (using the standard 1,3,3,5,9 piece values), and has a small award if more of its pieces are towards the opposite side of the board. This reward exists to make the AI more aggressive and push pieces when possible.

### Considerable Moves
Sometimes, we wish we could have evaluated to just one level deeper on one move. For example, we don't want the AI playing according to a position 4 moves from now when it could lose its Queen 5 moves from now.

To solve this issue, I took inspiration from Alan Turning's AI, [Turochamp](https://en.wikipedia.org/wiki/Turochamp) (yes, he designed a chess AI, despite there being no computer that could run it at his time). Turing foresaw the same issue, and his solution was to have his AI only look at a subset of moves after a certain depth. Of course, the AI must also consider a "null" move in this case, to represent any non-considerable move, because the player isn't forced to make a "considerable" move. I implemented the idea with my AI, testing what exactly should count in this subset, so that we have a good evaluation function while also not taking up too much time.

I initially wanted to state that all captures are considerable, but that simply took too much time, especially in midgame positions with lots of possible captures. As a compromise, my AI deems any move which captures the piece that it just moved as "considerable". This means it accurately calculates positions where lots of trades occur, even if they occur quite a few moves from now.

### Summary
To review, my AI first evaluates every position up to some depth (determined by the amount of time it has left). After that, it evaluates any "considerable moves" to make sure that the position isn't extremely volatile. After that, it looks at material difference and piece location to determine how good the position is. After all that calculation, the AI takes the move which allows it to force the opponent into the worst possible position.

## Future Development
I plan on having the AI store its evaluation between moves. Currently the AI starts its evaluation from scratch on every move, but that's not necessary because it should have evaluated its current position on previous moves (albeit to a lower depth). Using that partial evaluation as a starting point could save a significant amount of time on each move, allowing further depth. It would also allow me to expand the definition of a "considerable" move, because the evaluations of those moves wouldn't be lost between turns.

Additionally, I'd be interested in adding a transposition table, allowing the AI to recognize when the same position results from a different sequence of moves, potentially saving computation time, as it would not have to re-evaluate identical positions.
