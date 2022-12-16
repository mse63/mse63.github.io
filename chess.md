---
title: Chess AI
layout: template
filename: chess
order: 2
show_tab: 1
--- 

# Chess AI
I created a Chess AI in Rust with no pre-made libraries. Because I am a better coder than I am a chess player, my AI beats me in chess every time.
See if you can beat it by challenging it on lichess: [melsh_bot](https://lichess.org/@/melsh_bot)

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
2. More importantly, the AI would not checkmate the opponent if it knew it could on a future turn. It didn't see any difference between winning immediately and winning 3 moves into the future. This led to the awkward situation where the AI would stall forever and make nonsense moves when it had a mate-in-1, because it saw itself as winning anyway.

To fix these issues, I added the length of the game to the smallest possible value for evaluating checkmates, so a checkmate further along the game is seen as better for the loser.

However, we can't always evaluate all the way until we hit a checkmate or draw. Chess has more possible games than there are atoms in the universe so, unlike some games such as tic-tac-toe, it is computationally impossible to calculate every potential future position with the algorithm described above. Therefore, after some finite depth, we need some method of scoring a board that does not rely on looking at future moves.

My AI mainly looks at material difference (using the standard 1,3,3,5,9 piece values), and has a small award if more of its pieces are towards the opposite side of the board. This reward exists to make the AI more aggressive and push pieces when possible.

### Considerable Moves
Sometimes, we wish we could have evaluated to just one level deeper on one move. For example, we don't want the AI playing according to a position four moves into the future, if that position would lose its Queen 5 moves into the future.

To solve this issue, I took inspiration from Alan Turning's AI, [Turochamp](https://en.wikipedia.org/wiki/Turochamp) (yes, he designed a chess AI, despite there being no computer that could run it at his time). Turing foresaw the same issue, and his solution was to have his AI only look at a subset of moves after a certain depth. Of course, the AI must also consider a "null" move in this case, to represent any non-considerable move, because the player isn't forced to make a "considerable" move. I implemented the idea with my AI, testing what exactly should count in this subset, so that we have a good evaluation function while also not taking up too much time.

The AI has a heuristic for how "valuable" a move is. That value is simply the change in the evaluation function as a result of the move, so taking a bishop would have a value of about `+3`. My AI deems a move "considerable" if its value is greater than a given threshold. This threshold must be carefully chosen - if it is too low, we run the risk of a possible infinite loop, if both players are always able to play moves of that value. If it is too high, we may have the AI fail to see tactics because it didn't consider the move. With these considerations taken into account, a value of `+1` was chosen.

### Hashing
In chess, it's quite common to arrive at the same position through taking different sequences of moves. There are 8902 possible sequences of the first 3 moves, but only 5,362 unique positions among them. For this reason, it would be nice to be able to store all previous evaluations, so that they can be recalled and returned instead of recalculting whenever we see the same board state in our minimax search. This can be done in a HashMap. To save on memory, we can forgo storing the actual board positions, and only store a 64-bit hash of the position, assuming that no collisions exist. Therefore, picking an appropriate hash function is crucial. There exist more positions in chess than atoms in the universe, so there is no chance that we can come up with a hash function which will never collide on any two chess positions. However, we aren't evaluating every possible chess position. The goal is then to achieve a hash function with a high [Hamming Distance](https://en.wikipedia.org/wiki/Hamming_distance). That is to day: it's okay for two chess boards to have the same hash function, so long as those chess boards are dissimilar enough that the AI us unlikely to evaluate both of them within the same game. odds of our AI evaluating both of them within the same game are unlikely.

Many ameture chess engines use a specific version of [Zobrist Hashing](https://en.wikipedia.org/wiki/Zobrist_hashing), in which a chess board is represented by a 64-bit number, each bit of which represents whether or not a certain square is occupied. This is, frankly, a very poor choice. It doesn't carry any information regarding the actual pieces, and has an extremely low coding distance (in fact, a position a capture will collide with every other capture that piece could have made, leading to potentially disastrous results).

For my AI, I invented (and later discovered the already-documented existance of) a more general version of Zobrist Hashing. In this hash, any number of attributes may be assigned a randomly generated 64 bit number. The hash is then the XOR of all the properties that the object has. For example, a knight existing on c3 would be one such attributes. Updating the hash between states is then XORing the hash of the board with the hash of the changes attributes. This is much better as a single piece change will change, on average 32 bits in the hash, rather than the 1 bit in the more specific Zobrist Hashing verison.

### Summary
To review, my AI first evaluates every position up to some depth (determined by the amount of time it has left). After that, it evaluates any "considerable moves" to make sure that the position isn't extremely volatile. After that, it looks at material difference and piece location to determine how good the position is. After all that calculation, the AI takes the move which allows it to force the opponent into the worst possible position.

## Future Development
Taking a more data-driven approach to the evaluation function would likely prove worthwhile. The constants chosen for the value of various pieces is just the generally accepted values. It would be interesting (and also more pure from an AI standpoint) to have some sort of correlation or machine learning to choose those constants, instead of it being manually chosen by me. 
