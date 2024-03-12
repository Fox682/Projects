# Optimization of Tic-Tac-Toe Storage

## ...or How I Went Crazy Driving to Work one Day.

I'm the words of Alejandra González "10 bits are 'boring'...". Yes, yes they are, 
because that's just too many bits... I'm a little crazy, so bare with me on this one. 


#### Inspiration:
15 bits by Chris Barrick
- https://cbarrick.dev/posts/2024

18 bits (the fun way) by Alejandra González
- https://blog.goose.love/posts/tictactoe/

Both used material in this paper for information and inspiration (as far as I know).
- https://www.egr.msu.edu/~kdeb/papers/k2007002.pdf

Both Chris and Alejandra have great write ups on how to do this sensibly and with code examples! 
So do check them both out. I don't have code examples yet, but feel free to give it a try.

However, I am going to descend into madness. For this though I blame Chris (I saw his work first).
I became obsessed with this problem the moment I heard of it and I had to find a better solution. 
I knew intuitively there was one and I was determined to find it.

This is a "clean room" idea, no other research about this was done, until I had written this down first. 
Otherwise, whats the fun in that?

#### Goals:
We need to be able to do the following:
- Store game state (piece placement) per turn
- Store the final state (Win or Cat (Stalemate) )
- Who went where and who's turn is it (replay)
- Determine the Winner 

#### The Problem:
How do we store a the state of a game of Tic-Tac-Toe?

Well, you might say that's really simple, you take the binary representation of 
each "cell" (there are 9 in Tic-Tac-Toe) and then string the bits together and you'll 
store all of the bits required for the state of the game.

Of course, this is the obvious solution. Using this we see that we need 2 bits to store 
3 values (X, O & Space/Blank) for each of the 9 cells.

```
2 * 9 = 18 bits - problem solved! 
```

Yes, this is true. This works and is a very linear way of thinking. To store a full 
game you'd need a maximum of 9 board states per game:

```
18 * 9 = 162 bits total, bleh...
```

If you're using 15 bits per board state like Chris did, you'd end up with:

```
15 * 9 = 135 bits, that's better! 
```

I do like this, but I think I can do better.

First, some rules. For Tic-Tac-Toe, cause those are the only ones I'm going to follow. 

#### Rules
- First to Move is X then O 
- Players keep playing until a Win or the board is filled, a Cats Game (Draw or Stalemate)
  - We'll do the calculations for full games not including wins in the 3rd play etc.
- 9 play maximum we're not counting a blank Tic-Tac-Toe board as anything to be stored
  - That would just compress into a 0, would make no sense.

### First Innovation: Key Frame Thinking

If I was going to discover another solution I needed to think about this problem in a 
different way. What if we were to store just the change in the state of the game rather 
than just the instantaneous state.

As each player moves we simply put a 1 where a player moves. Simple! 

Well... Great, now I've broken it. I'm an idiot, I can't tell who went where nor where 
the pieces went. I can't tell who won... Not enough information. Good job... genius.

..Wait, hold on... If only I had more...

### Second Innovation: Temporal Reconstruction

...Time, I can use time for each "Frame" in a film and play them back one after the other. 
I can do the same with Tic-Tac-Toe!

I can even cheat. Check this out, we know that X goes first followed by O each time until 
the game is complete. I can use this to my advantage.

For each "frame" of game state I just need to keep a record of each board state for each 
turn so I can reconstruct everything.

So with this we have 9 bits per board with a maximum of 9 games. 

```
9 * 9 = 81 bits to store a whole game
```

Not bad! Not only can I reconstruct the board over time I know who went and where the pieces belong!

Well that was fun! Problem Solved, we can move on... 

> *Voice my head* - "No, there's wasted space and you can do better..."

Seriously?!?! *Points to self in the mirror* look bud, we just went from 162 to 135 to 81 bits 
to store the game state AND now you have data to reconstruct the whole game from start to finish 
and you can find out who won. I just cut the storage space in half exactly what wasted...

### Third Innovation: Space... Spaaaaaaaaace!

... Space.... goddamnit... comedy comes in threes... maximum effort...

Since we're using a sort of Temporal Reconstruction to store the bits we can, instead of storing 
all 9 bits per "frame", store the data in an even more esoteric way. 

Using X and Y... I know, I can hear the screaming, I just used time and now I'm using space to 
solve this problem... I'm sorry.

For each X and Y, we can use a pair of 3-bit values to store the position of the change bits in each of the 9 cells in Tic-Tac-Toe when used together.

Using a whopping 6 bits per frame and 9 frames for a whole game crunches this down...

```
9 * 6 = 54 bits... For a whole game of Tic-Tac-Toe... 
```

That's less than 7 bytes... That's less than the amount of space needed to store the word "Esoteric".

And because I'm a completionist (sometimes) we can store this using Base64. Each character 
representing exactly 6 bits.

This solution of course is a prime example of the "Space-Time Trade-off". We'll end up with a more 
complicated program to save even more space, but you'd be better off with using a little more space 
to keep the code simpler.

- https://en.m.wikipedia.org/wiki/Space–time_tradeoff

Don't do what I did. This is maximum overkill. This is also the reason why meta-data IS the data. 
I'm not actually storing the data, I'm just storing data about the data and reconstructing everything from that. 
If you find a way to store the whole game state in less than 54-bits you're a masochist... 

## Good Luck!
