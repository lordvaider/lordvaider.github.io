# IKB - Span of a Tetrahedron

# Problem
For a set $$E$$ in $$R^3$$, let $$L(E)$$ consist of all points on all lines determined by any two points of $$E$$. 
More formally, define $$L(E)$$ as: 

\[
L(E) = \bigcup_{\{p, q\} \subset E} \ell(p, q),
\]
where \( \ell(p, q) = \{ (1-t)p + tq : t \in \mathbb{R} \} \) is the line passing through the distinct points \( p \) and \( q \) in \( E \).


Thus if $$V$$ consists of the four vertices of a regular tetrahedron, then $$L(V)$$ consists of the six edges of the tetrahedron, extended infinitely in both directions. Does $$L(L(V))$$ span all of $$R^3

# Solution 
I originally saw this problem in Stan Wagon's problem collection (Lots of cool stuff in there and worth bookmarking for every puzzle enthusiast) He credits Victor Klee with creating this problem. Klee was a famous geometer, who proposed the art gallery problem and showed that the worst case runtime of the simplex method is exponential. 

I will make some appeals to geometric intuition in what follows, as I don't have the patience to write out the algebraic proofs for some of these. 

Firstly, we note that the set $$L(V)$$ is just the 6 edges of the tetrahedron, extended to infinity on both sides. Let's call these lines $$E_1, E_2, ... E_6$$. If 2 lines are co-planar, the span of those lines can only include the plane. If we want to fill space, we must focus on the skew lines. There are 3 pairs of skew lines. Let's look at one pair in isolation. 

Under a suitable rotation, the pair of skew lines can be taken to E_1: z=0, x = 0. E_4: x=1, y=0. Convince yourself that the L(E1, E4) contains all points in R^3, except for two planes - The one containing E_4 and parallel to E_1 and the one containing E_1 and parallel to E_4. 

For each pair of skew lines, we get a pair of planes that is not in the image of of L(E_i, E_j). The intersectoins of these pairs of planes give us 4 points that are not in the image $$L(L(V))$$. 

Is there a succint way to describe these points? My high school math professor used to tell us that whenever we encounter a regular tetrahedron, one potentially fruitful avenue was to inscribe it in a cube. If you have a cube with vertices $$\in {0, 1}^3$$, then you can inscribe in it the regular tetrahedron with vertices $$(0, 0, 0), (0, 1, 1), (1, 0, 1), (1, 1, 0) $$ (Convince yourself that it is indeed regular). The points not in $$L(L(V))$$ are then exactly the 4 remaining vertices of the cube. 





