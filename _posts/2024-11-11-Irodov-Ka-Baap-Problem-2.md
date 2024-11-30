# IKB - Span of a Tetrahedron

# Problem
For a set $$E$$ in $$R^3$$, let $$L(E)$$ consist of all points on all lines determined by any two points of $$E$$. 

More formally, define $$L(E)$$ as: 

$
L(E) = \bigcup_{\{p, q\} \subset E} \ell(p, q),
$

where $$\ell(p, q) = \{ (1-t)p + tq : t \in \mathbb{R} \}$$ is the line passing through the points $$p$$ and $$q$$.


Thus if $$V$$ consists of the four vertices of a regular tetrahedron, then $$L(V)$$ consists of the six edges of the tetrahedron, extended infinitely in both directions. 

Does $$L(L(V))$$ span all of $$R^3$$?

# Solution 
I originally saw this problem in [Stan Wagon's problem collection](https://stanwagon.com/wagon/misc/bestpuzzles.html) (Lots of cool stuff in there and worth book-marking for every puzzle enthusiast) He credits Victor Klee with creating this problem. Klee was one of those badass geometers who had a fully functional amusement parks inside their brains. Among his many achievements, the ones I understood and was impressed are proposing the art gallery problem and showing that the worst case runtime of the simplex method is exponential. 

This problem is one of those beauties that befuddles experienced mathematicians, but that can be solved by a layperson with good enough geometric intuition. The solution below relies on this, as I don't have the patience to write the full blown algebraic proof. 

Firstly, we note that the set $$L(V)$$ is just the 6 edges of the tetrahedron, extended to infinity on both sides. 

Let's label these lines $$E_{12}, E_{13}, E_{14}, E_{23}, E_{24}, E{34}$$. (A line is named based on the 2 vertices of the tetrahedron it passes through)

If 2 lines are co-planar, the span of those lines can only include the plane containing those lines. If we want to fill space, we must focus on the skew lines. 

In the set above, there are 3 pairs of skew lines - $$(E_{12}, E_{34}), (E_{14}, E_{23})$$ and $$(E_{13}, E_{24})$$. 

Let's focus one of these pairs - Convince yourself that the set $$L(E_x, E_y$$) contains all points in $$R^3$$, except for two planes - The plane containing $$E_y$$ and parallel to $$E_x$$ and the plane containing $$E_x$$ and parallel to $$E_y$$. (If you're having difficulty visualizing this, you can use the fact that such a pair of skew lines can be rotated to the lines $$E_{12}: x=0, z=0$$ and $$E_{34}: x=1, y=0$$ and then algebraicially try to prove that you can attain all $$(x, y, z)$$ as linear combinations of points on these lines EXCEPT for the ones where $$x=0, z \neq 0$$ and $$x=1, y \neq 0$$.  

For each pair of skew lines, we get a pair of parallel planes that is NOT in the image of of L(E_x, E_y). The intersections of these pairs of planes give us 4 points that are not in the image $$L(L(V))$$. 

Is there a succint way to describe these points? One of my favourite professors used to say that whenever you encounter a regular tetrahedron, one potentially fruitful avenue is to inscribe it in a cube. If you have a cube with vertices $$\in {0, 1}^3$$, then you can inscribe in it the regular tetrahedron with vertices $$(0, 0, 0), (0, 1, 1), (1, 0, 1), (1, 1, 0)$$. The points not in $$L(L(V))$$ are then exactly the 4 remaining vertices of the cube. 





