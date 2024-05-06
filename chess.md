---
title: Chess AI
layout: template
filename: chess
order: 1
show_tab: 1
--- 

# Chess AI
I created a Chess AI in Rust with no pre-made chess libraries. Because I am a better coder than I am a chess player, my AI beats me in chess every time.

You can view the [source code](https://github.com/mse63/melsh_bot) on GitHub.

See if you can beat it by challenging it on its [lichess profile](https://lichess.org/@/melsh_bot).


## Example Game
Below is an example game of my AI vs Maia 1900 (an AI designed to mimic a human 1900 Elo player):

<iframe src="https://lichess.org/embed/game/hYYw800k?theme=auto&bg=auto#109"
width=600 height=397 frameborder=0></iframe>

## Algorithm

### Overview
My chess AI uses a [minimax](https://en.wikipedia.org/wiki/Minimax) algorithm with alpha beta pruning. In order to find the best move, the root node, representing the current game state, tells its children nodes, representing possible future game states, to find the best move and score how good that position is for the opponent. The root node then takes the move that results in the worst possible position for the opponent, as that would be the best position for us.

The score of a position is the opposite of the lowest opponent score of any of its children, because that is the optimal position the AI can force its opponent into.

### Evaluation Function
If the position is solvable at a shallow-enough search, my AI directly evaluates it depending on how many moves away it is from a forced checkmate or a forced draw.

However, we can't always evaluate all the way until we hit a checkmate or draw. Chess has more possible games than there are atoms in the universe so, unlike some games such as tic-tac-toe, it is computationally impossible to calculate every potential future position. Therefore, after some finite depth, we need some method of scoring a board that does not rely on looking at future moves.

My AI mainly looks at material difference (using the standard 1,3,3,5,9 piece values), and has a small award if more of its pieces are towards the opposite side of the board. This reward exists to make the AI more aggressive and push pieces when possible.

### Considerable Moves
Sometimes, we wish we could have evaluated to just one level deeper on one move. For example, we don't want the AI playing according to a position four moves into the future, if that position would lose its queen 5 moves into the future.

To solve this issue, I took inspiration from Alan Turning's AI, [Turochamp](https://en.wikipedia.org/wiki/Turochamp) (yes, he designed a chess AI, despite there being no computer that could run it at his time). Turing foresaw the same issue, and his solution was to have his AI only look at a subset of moves after a certain depth. Of course, the AI must also consider a "null" move in this case, to represent any non-considerable move, because the player isn't forced to make a "considerable" move. I implemented the idea with my AI, using the one-move change in the evaluation function to evaluate the possible moves, and after a certian depth, only considering moves with a value above a specific threshold. This threshold must be carefully chosen - if it is too low, we run the risk of a possible infinite loop, if both players are always able to play moves of that value. If it is too high, we may have the AI fail to see tactics because it didn't consider the move.

### Hashing
In chess, it's quite common to arrive at the same position through taking different sequences of moves. There are 8902 possible sequences of the first 3 moves, but only 5,362 unique positions among them. For this reason, it would be nice to be able to store all previous evaluations, so that they can be recalled and returned instead of recalculting whenever we see the same board state in our minimax search. This can be done in a HashMap. To save on memory, we can forgo storing the actual board positions, and only store a 64-bit hash of the position, assuming that no collisions exist. Therefore, picking an appropriate hash function is crucial. There exist more positions in chess than atoms in the universe, so there is no chance that we can come up with a hash function which will never collide on any two chess positions. However, we aren't evaluating every possible chess position. The goal is then to achieve a hash function with a high [Hamming Distance](https://en.wikipedia.org/wiki/Hamming_distance). That is to day: it's okay for two chess boards to have the same hash function, so long as those chess boards are dissimilar enough that the AI us unlikely to evaluate both of them within the same game. The odds of our AI evaluating both of them within the same game are so low, as the AI will evaluate nowhere near the 2^64 possible chess positions in one game, that we need not take it into account.

For my AI, I invented (and later discovered the existence of) [Zobrist Hashing](https://en.wikipedia.org/wiki/Zobrist_hashing). In this hash, any number of attributes may be assigned a randomly generated 64 bit number. The hash is then the XOR of all the properties that the object has. For example, a knight existing on c3 would be one such attribute. Updating the hash between states is then XORing the hash of the board with the hash of the changes attributes. This leads to pseudo-random hashes, that have an extremely low chance of colliding.

### Summary
To review, my AI first evaluates every position up to some depth (determined by the amount of time it has left). It searches deeper for "considerable moves" to avoid making decisions based on extremely volatile states. After all that calculation, the AI takes the move which allows it to force the opponent into the worst possible position.

## Future Development
Taking a more data-driven approach to the evaluation function would likely prove worthwhile. The evaluation function is just linear hand-picked constants. It would be interesting (and also more pure from an AI standpoint) to have some sort of correlation or machine learning to choose those constants, instead of it being manually chosen by me.
