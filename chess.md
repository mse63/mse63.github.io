---
title: Chess AI
layout: template
filename: chess
order: 2
show_tab: 1
--- 

# Chess AI

## Example Game
Below is an example game of my AI (White) vs 2000 Elo Stockfish (Black):

<iframe src="https://lichess.org/embed/BHLqEa1J#0?theme=auto&bg=auto"
width=600 height=397 frameborder=0></iframe>

## Algorithm
Super cool description of my Chess AI algorithm and stuff.

## Future Development
I plan on having the AI store its evaluation between moves. Currently the AI starts its evaluation from scratch on every move, but that's not necessary because it should have evaluated its current position on previous moves (albeit to a lower depth). Using that partial evaluation as a starting point could save a significant amount of time on each move, allowing further depth. It would also allow me to expand the definition of a "considerable" move, because the evaluations of those moves wouldn't be lost between turns.

Additionally, I'd be interested in adding a transposition table, allowing the AI to recognize when the same position results from a different sequence of moves, potentially saving computation time, as it would not have to re-evaluate identical positions.
