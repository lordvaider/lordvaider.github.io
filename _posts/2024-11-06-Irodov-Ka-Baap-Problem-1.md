# IKB - The Impossible Problem Version 1729.0

# Puzzle 1:

Two infinitely intelligent and truthful mathematicians $$P$$ and $$Q$$ meet to play a game. Each of them individually decides on a secret real number greater than 1. ($$P$$'s number is p and Q's number is q) They submit their numbers to a moderator M, who reveals to both of them the product pq, along with another number r. M doesn't reveal which number is pq and which number is r. This entire setup is common knowledge between $$P$$ and Q, i.e. they both know the setup and know that the other knows the setup etc. 

M then turns to $$P$$ and asks him if he knows q. If $$P$$ says no, he asks $$Q$$ if he knows p. If $$Q$$ says no, he then asks $$P$$ again, and so on. Prove that this game terminates after a finite number of rounds. 

# Solution (Updated 11Nov24):
This problem belongs to a category of puzzles that exploit the logical concept of Common Knowledge. These puzzles explore how shared knowledge, and the ability to deduce information based on what others know, can lead to surprising and often counterintuitive conclusions. Famous examples include ~~Martin Gardener's~~ Hans Freudenthal's [Impossible Puzzle](https://en.wikipedia.org/wiki/Sum_and_Product_Puzzle) and the [Blue-Eyed Islander Puzzle](https://terrytao.wordpress.com/2008/02/05/the-blue-eyed-islanders-puzzle/). If you haven't encountered these before, they are well worth exploring as quintessential examples of the genre. Also worth reading is Scott Alexander's DELIGHTFUL [short story](https://slatestarcodex.com/2015/10/15/it-was-you-who-made-my-blue-eyes-blue/) exploring the puzzle from the point of view of the islanders. 

At first glance, such puzzles can feel perplexing or even paradoxical. A common reaction is, "Where is the new information coming from?" After all, for progress to be made, the participants $$P$$ and $$Q$$ must gain new insights with each round, even though no additional explicit information is revealed. This leads to the realization that the source of this "new" information is not external but emerges from the participants’ reasoning about each other's reasoning.

The key to unraveling these puzzles is understanding that 1) $$P$$ and $$Q$$ are infinitely intelligent and that 2) they are both aware of the setup. This means in any given situation they can make all logically possible deductions and know that other participants can do the same. Therefore, the lack of immediate deductions becomes itself a crucial source of information. For example, if $$P$$ doesn’t know the answer in the first round, it implies that certain conditions must hold for $$Q$$, allowing $$Q$$ to refine their possibilities—and vice versa.

With this in mind, let us systematically track the information that both participants have in each round. 

## Start of the game
At the start of the game, both $$P$$ and $$Q$$ see 2 numbers $$a$$ and $$b$$ (Without loss of generality, $$a < b$$). Hence we have:

__$$P$$'s knowledge__: 
1. $$q = \frac{a}{p}$$ OR $$q = \frac{b}{p}$$
2. $$q > 1$$

__$$Q$$'s knowledge__: 
1. $$p = \frac{a}{q}$$ OR $$p = \frac{b}{q}$$
2. $$p > 1$$

## Round 1, Question 1:
If $$\frac{a}{p} < 1$$, $$P$$ can immediately eliminate it and knows that $$q = \frac{b}{p}$$
Conversely, if $$P$$ cannot immediately identify $$q$$, this must imply $$\frac{a}{p} > 1 \implies p < a$$.
Hence if $$P$$ answers No in the first round, $$Q$$'s knowledge gets updated as following:

__$$Q$$'s knowledge__: 
1. $$p = \frac{a}{q}$$ OR $$p = \frac{b}{q}$$
2. $$1 < p < a$$

## Round 1, Question 2:
If $$\frac{b}{q} > a$$, $$Q$$ can immediately deduce $$p$$.
Conversely, if $$Q$$ cannot deduce $$p$$, this must imply $$\frac{b}{q} < a \implies q > \frac{b}{a}$$.
Hence if $$Q$$ answers No in the first round, $$P$$'s knowledge gets updated as follows 

__$$P$$'s knowledge__: 
1. $$q = \frac{a}{p}$$ OR $$q = \frac{b}{p}$$
2. $$q > \frac{b}{a}$$

As we can see, after one round, $$P$$'s lower bound for $$q$$ went from $$1$$ to $$\frac{b}{a}$$. Continuing in this manner, we can see that after the $$n^{th}$$ round $$P$$'s lower bound for $$q$$ becomes $$\left( \frac{b}{a} \right)^n$$. Similarly, $$Q$$'s upper bound for $$p$$ at the end of the $$n^{th}$$ round is $$a \cdot \left( \frac{a}{b} \right)^n$$

Hence it's clear that sooner or later, one of them will deduce the other's number.


