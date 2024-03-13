# Optimization of Tic-Tac-Toe Storage

## ...or How I Went Crazy Driving to Work one Day.

I'm the words of Alejandra González "10 bits are 'boring'...". Yes, yes they are, 
because that's just too many bits... I'm a little crazy, so bare with me on this one. 


#### Inspiration:
[15 bits by Chris Barrick](https://cbarrick.dev/posts/2024/02/19/tic-tac-toe)

[18 bits by Alejandra González](https://blog.goose.love/posts/tictactoe/)

Both used material in this paper for information and inspiration (as far as I know).
[Evolution of No-loss Strategies for the Game of
Tic-Tac-Toe](https://www.egr.msu.edu/~kdeb/papers/k2007002.pdf)

Both Chris and Alejandra have great write ups on how to do this sensibly and with code examples! 
So do check them both out. I don't have code examples yet, but feel free to give it a try.

However, I am going to descend into *madness*. For this though I blame Chris (I saw his work first).
I became obsessed with this problem the moment I heard of it and I had to find a better solution. 
I knew intuitively there was one and I was determined to find it.

This is a semi-"clean room" idea, no other research beyond what was listed above was done, until I had 
written this down first. Otherwise, what's the fun in that?

#### The Problem:
How do we store a the state of a game of Tic-Tac-Toe?

I took a different spin on this. It's nice to be able to store a single state as small as possible.
But what if we were to store the replay of the game... as well as everything else that goes with 
Tic-Tac-Toe?

First, we need some Goals so we can make sure we have enough information so we can play the game, determine who won and replay it back step by step.

#### Goals:
We need to be able to do the following:
- Store game state (piece placement) per turn
- Store the final state (Win or Cat (Stalemate) )
- Who went where and who's turn is it (replay and continue where we left off!)
- Determine the Winner
- Store the game state in the most asinine way I can think of 

Well, you might say that's really simple, you take the binary representation of 
each "cell" (there are 9 in Tic-Tac-Toe) and then string (append) the bits together and you'll 
store all of the bits required for the state of the game.

```
Tic-Tac-Toe Board:

X | O | 
---------
  | X | O
---------
X | O | O

[See if you can spot the error! Clue: Check the rules] 
```
Of course, this is an obvious solution. Using this we see that we need 2 bits to store 
3 values (X, O & Space/Blank) for each of the 9 cells.

```
We can use:
00 - Blank
01 - X
10 - O
11 - Invalid

01 | 10 | 00 
---------
00 | 01 | 10
---------
01 | 10 | 10

So when we store this, we can do

Game = [011000000110011010]

or 98714 if you're fancy

2 * 9 = 18 bits - problem solved! 
```

Yes, this is true. This works and is a very linear way of thinking. To store a full 
game you'd need a maximum of 9 board states per full game (ie. Cat/Draw):

```
Game1 = [011000000110011010]
Game2 = [010001010110011010]
Game3 = [011001100100010010]
Game4 = [001001100010001010]
Game5 = [011000010010111010]
Game6 = [011000101110001010]
Game7 = [001001001110011010]
Game8 = [011010010111011010]
Game9 = [011010000010011010]

18 * 9 = 162 bits total, bleh...
```

The problem is that you got a bunch of wasted states if you use 2 bits (The invalid "11" above).
But you can work around the issue if you're using 15 bits per board state like Chris did 
(using Base-3), you'd end up with:

```
15 * 9 = 135 bits total, that's better! 
```

I do like this, but I think I can do better. I may not be able to store a full board state all at once in less
than 15 bits but... a snapshot of a board, doesn't tell me whos turn it is, who did what and when they did
something dumb so that I could win.

First, some rules. For Tic-Tac-Toe, cause those are the only ones I'm going to follow. 

#### Rules
- First to Move is X then O, always.
- We'll do the calculations for full games not including wins in less than 9 moves.
- We're not counting a blank Tic-Tac-Toe board as anything to be stored.
  - That would just compress into a 0, would make no sense.

### First Innovation: Key Frame Thinking Pt. 1

If I was going to discover another solution I needed to think about this problem in a 
different way (Outside the box!). What if we were to store just the change in the state of the game 
rather than just the instantaneous state of the  entire board at any one time?

As each player moves we simply put a 1 where a player moves. Let's start there that seems super simple! 


```
Tic-Tac-Toe Board:

1 | 0 | 0
---------
0 | 0 | 0
---------
0 | 0 | 0 

Board State = [10000000]

```

Well... Great, now I've broken it. I can't tell who went where nor where 
the pieces went. I can't tell who won... Not enough information. Good job... genius.

...Wait, hold on... If only I had more...

### Second Innovation: Temporal Reconstruction (Key Frame Thinking Pt. 2)

...Time, I can use time for each "Frame" in a video and play them back one after the other. 
I can do the same with Tic-Tac-Toe!

I can even cheat. We know that X goes first followed by O each time until 
the game is complete. I can use this to my advantage.

```
X goes First:

Move1:
1 | 0 | 0
---------
0 | 0 | 0
---------
0 | 0 | 0

Move2: (Obviously O)
0 | 0 | 0
---------
0 | 1 | 0
---------
0 | 0 | 0

Move3: (Obviously X, again)
0 | 1 | 0
---------
0 | 0 | 0
---------
0 | 0 | 0

```

For each "frame" of game state I just need to keep a record of each board state for each 
turn. Now, I can reconstruct everything.

```
Store each Frame, like this:

BoardT1 = [100000000]
BoardT2 = [000100000]
BoardT3 = [010000000]
...

```

So with this we have 9 bits per board with a maximum of 9 games. 

```
9 * 9 = 81 bits to store a whole game state

From 162 to 81 bits per game that cuts storage in half for the whole game!
```

Not bad! Not only can I reconstruct the board over time:
- We know whos turn it is 
- We also know where they place their pieces on the board
- We can also determine a winner here pretty easily

On top of all that there's literally only 9 states to keep track of. Friggin Awesome!

Well that was fun! Problem Solved, we can move on... 

> *The Abyss of Insanity* - "No, you can do better... less storage..."

Seriously?!?! I just cut the storage space in half... exactly what wasted...

### Third Innovation: Space... Spaaaaaaaaace!

... Space.... damnit... comedy comes in threes...

> Past Me Who is a Jerk - "Store the game state in the most asinine way I can think of."

... Fine, since we're using a sort of Temporal Reconstruction (Key Framing) to store the bits we can, instead of storing all 9 bits per "frame", store the data in an even more esoteric way. How?

Using X and Y... I know, I... I can hear the screaming, I just used time and now I'm using space to 
solve this problem... I'm sorry, stop screaming!

For each X and Y, we can use a pair of 3-bit values to store the position of the change bits in each of the 9 cells in Tic-Tac-Toe when used together.

```
It Would look like this (maybe):

1 | 0 | 0
---------
0 | 0 | 0
---------
0 | 0 | 0 

X = [100]
Y = [100]

0 | 0 | 0
---------
0 | 1 | 0
---------
0 | 0 | 0

X = [010]
Y = [010]

0 | 0 | 0
---------
0 | 0 | 0
---------
0 | 0 | 1

X = [001]
Y = [001]

Board States:

T1 = [100100]
T2 = [010010]
T3 = [001001]
...

```

Using a whopping *6 bits per frame* and 9 frames for a whole game crunches this down...

```
9 * 6 = 54 bits... For a whole game of Tic-Tac-Toe...

From 162 to 54 bits per game that's a 66% savings... 33% of the space! Not insignificant!
```

That's less than 7 bytes... That's less than the amount of space needed to store the word "Esoteric".

And because I'm a completionist (Chris used Base-3, we need moar bases!) we can store this using Base64. Each character representing exactly 6 bits. There's lots of tools to work with Base64 so storage would be reasonably efficient.

[Wikipedia - Base64](https://en.wikipedia.org/wiki/Base64)

```
Board States from above as Base64:

T1 = ["k"]
T2 = ["S"]
T3 = ["J"]
...

```

Well that's just plain weird... That would be valid win for X if it was on turns 1, 3 & 5.

### Conclusions

That was fun.

The caveat of course being that this is just the storage portion. The actual logic for placing the pieces on
the board or determining a win are for later. As also with the 765 possible non-duplicated states.

The rest will have to be an exercise to the reader, as storing all possible 765 states is hard since it would 
require each frame to store the instantanious state which this system doesn't do. You could however do the 18 bit 
classic version and you'd end up with:

```
765 States * 18 bits = 13.77 Kilobits = ~1,722 KBs = ~1.68 MBs
This doesn't include the Reflective and Rotational Duplicates however. But you could just add more bits
to each state. Probably 5 or 6.
```

This solution of course is a prime example of the "Space-Time Trade-off". We'll end up with a more 
complicated program to save even more space, but you'd be better off with using a little more space 
to keep the code simpler.

- [Wikipedia - Space-Time Tradeoff](https://en.m.wikipedia.org/wiki/Space–time_tradeoff)

Don't do what I did. This is maximum overkill. This is also the reason why meta-data IS the data. 
I'm not actually storing the actual data, I'm just storing data about the data and reconstructing everything 
from that.  If you find a way to store the whole played game state (9 moves) in less than 54-bits
you're a masochist... and I want to know more.

### Good Luck!
