# The Rising Sea

I'd like to say something tonight about problem solving, themed around a semi-famous problem solver, and his metaphor.

## Grothendieck

This young man is one of those sorts of people that is legendary within a very particular subculture, and relatively unknown otherwise:

![Grothendieck 1951](/img/grothendieck-young.png)

This is Alexander Grothendieck, a revolutionary Mathematician. His work in the 1950's through the 1970's changed the course of pure mathematics research in multiple fields.

Every mathematician knows this man's name, and are either familiar with his work, or familiar with how familiar some of their peers are with his work.

To give some sense of both his stature, and the importance of his work, some rough analogies in other domains:

- Algebraic Geometry: Grothendieck
- Physics: Paul Dirac
- Computer Science: Alonzo Church
- Programming Languages: John McCarthy

Each of these individuals is not the most well known figure in their fields: in mathematics, Newton and Euler are more well known, in Physics its Newton and Einstein, Computer Science has Turing, Programming Languages have K&R. But each of these people had a massive impact in their field, and are well known and deeply respected by practitioners.

Another anecdote to support the point: Grothendieck's obituary ran in the New York Times, and, eventually, the academic journal Nature. His friend David Mumford's first attempt at an obituary was *rejected* by Nature:

> The sad thing is that this was rejected as much too technical for their readership. Their editor wrote me that 'higher degree polynomials', 'infinitesimal vectors' and 'complex space' (even complex numbers) were things at least half their readership had never come across.

This may give some sense of the depth and diffculty of Grothendieck's work, and justifies (a little) the absense of discussion here of what Grothendieck accomplished in mathematics.

A note Grothendieck left in the guestbook of a coffee shop frequented by his mathematical colleagues give a sense of his personality:

![Grotheniek's Riemann-Roch Note](/img/grothendieck-riemann-roch.jpg)

A rough translation taken from Reddit:

> Witch's Kitchen 1971

> Riemann-Roch: The latest craze: The diagram [...] is commutative.

> In order to give this statement about f:x → y an approximate meaning, I had to abuse the audience's patience for nearly two hours. In black and white (in Springer's Lecture Notes) it seems to take up to 400 or 500 pages. A thrilling example for how our urge of knowledge and discovery plays itself out more and more in a logical delirium that is enraptured from life, while life itself goes to hell in a thousand ways - and is threatened by final annihilation. It's high time to change our course!

## The Rising Sea

My goal is not so much to discuss Grothendieck, or his work, which would be quite impossible. I'd like to discuss Grothendieck's philosophy and approach to problem solving (which in mathematics, is the proving of mathematical theorems). He used the analogy of a rising sea:

> A different image came to me a few weeks ago. The unknown thing to be known appeared to me as some stretch of earth or hard marl, resisting penetration...  the sea advances insensibly in silence, nothing seems to happen, nothing moves, the water is so far off you hardly hear it... yet it finally surrounds the resistant substance...

To Grothendeck, problems are not best solved by strenuous effort, but by the gradual building of structure and theory. Eventually, the problem no longer resists, and it is only necessary to observe that there is no longer any problem remaining at all.

His friend and student Pierre Deligne gives another angle:

> I have also learned not to take glory in the difficulty of a proof: difficulty means we have not understood. The ideal is to be able to paint a landscape in which the proof is obvious.

## The Rising Sea

In contrast, his contemporary Jean Paul Serre took the opposite approach, seeking the most direct attack on any problem faced. Grothendieck wrote extremely long and detailed cathedrals of theory, Serre wrote short works that are marvels of elegance and concision.

Serre is well known to Mathematicians for his legendary texbooks. The most well known of which is his "Course in Arithmetic", a short yet remarkably wide ranging and deep work on modern Number Theory. The title is a mistranslation, it's very much *not* about arithmetic as the layperson would understnd it. In my opinion, the mistake in translation is a charming one.

We consider the question: which approach leads to better computer programs?

## An Example: Advent of Code 2022, Problem 5

I'd like to illustrate Grothendieck's approach with a simple example programming puzzle. The idea for this talk formed in my mind while working on Advent of Code puzzles. I noticed that I felt a singular satisfaction when solving a certain type of puzzle.

My example is taken from Advent of Code 2022, it's problem five. I chose this puzzle because it's simple enough that it's non-strenuous to state, is amenable to the approach I want to illustrate, and because a very informal survey of reddit shows that it's common for newer puzzlers to get stuck on just this problem. If you're en experienced programmer, there's likely not much to learn from this example to improve *your* skills, but consider how you would explain to a new programmer how you structure your reasoning when solving such a problem.

### Setup
Our puzzle concerns vertical stacks of blocks:

```
    [D]
[N] [C]
[Z] [M] [P]
-----------
 1   2   3
```

We have some instructions for moving these blocks around:

```
move 1 from 2 to 1

[D]
[N] [C]
[Z] [M] [P]
------------
 1   2   3
 ```

Sometimes we need to move more than one block at a time:

```
move 3 from 1 to 3

        [D]
        [N]
    [C] [Z]
    [M] [P]
-----------
 1   2   3
```

The inputs to the puzzle are an initial setup of the stacks:

```
[T]     [Q]             [S]
[R]     [M]             [L] [V] [G]
[D] [V] [V]             [Q] [N] [C]
[H] [T] [S] [C]         [V] [D] [Z]
[Q] [J] [D] [M]     [Z] [C] [M] [F]
[N] [B] [H] [N] [B] [W] [N] [J] [M]
[P] [G] [R] [Z] [Z] [C] [Z] [G] [P]
[B] [W] [N] [P] [D] [V] [G] [L] [T]
-----------------------------------
 1   2   3   4   5   6   7   8   9
```

and a long sequence of instructions:

```
move 5 from 4 to 9
move 3 from 5 to 1
move 12 from 9 to 6
...
```

The task is, more or less evident, execute the instructions correctly and obtain the final arrangement of the blocks.

If we're (a caricature of) Serre, we can start to make a direct attack on this problem. We could read the stack in as a large string, maybe pad the top, and start executing the instructions as string operations. Maybe no-one would actually do this, but it's common to see solutions of this sort that make a direct attack without much structure or modeling. Serre starts by writing *code*, as soon as possible.

Let's be Grothendieck, let's build a cathedral. Our goal is to model the problem in fine enough detail that the solution is self evident, we refuse to write any *difficult code*. This forces our hand, we can only start by modeling the problem with *data structures*.

### Modeling
We begin by thinking kinda hard about what data structures model the problem well. In this case, there are relatively self-evident concepts we can model semi-directly:

    - An `Instruction`.
    - A `Program` is a sequence of instructions.
    - A `Stack` of boxes.
    - A `Harbor` is a collection of `Stack`s.

Instructions are simple data, are immutable, and have no need to ever change.

```python
@dataclass
class Instruction:
    source: int
    destination: int
    count: int
```

We can call our sequence of instructions a `Program`. There's no need for fancy modeling here, it's a `List` of `Instruction`s:

```python
Program = List[Instruction]
```

### Modeling the Stacks
The `Stack`s and `Harbor` are more interesting, since they have *behaviour*: they respond to instructions. The `Harbor` is responds to an instruction directly by modifying its internal setup of stacks:

```python
class Harbor:
    # Some representation of the data.
    def execute(self, i: Instruction):
        # Some code that moves around the stacks.
    def run(self, p: Program):
        for instruction in p:
            self.execute(instruction)
```

Note how we can write the `run` method before writing `execute`. There's only one possible way to write it, it is as simple as possible, and it follows immediately from how we setup our model.

We can also write the `execute` method, but it requires inventing some operations we have not implemented yet:

```python
@dataclass
class Harbor:
    stacks: List[Stack]
    def execute(self, i: Instruction):
        source = self.stacks[i.source]
        destination = self.stacks[i.destination]
        payload = source.popsome(i.count)
        destination.extend(payload)
```

The `popsome` and `extend` operations do not yet exist,

A single `Stack` needs to respond to two operations: when the stack is the *source* we want to remove the head of the stack and keep the resulting boxes. When the stack is the *destination*, we want to concatenate the removed boxes onto the end.

```python
class Stack:
    boxes: List[Box]
    def popsome(self, count: int) -> List[Box]:
        tail = self.boxes[-count:]
        self.boxes = self.boxes[:-count]
        return tail
    def extend(self, some: List[Box]):
        self.boxes.extend(some)
```

There's very little choice in how we write any of these methods, their implementation is forced by our careful choice of datastructures, *and that's a good thing*. When done well, this method feels effortless, and it's difficult to identify where any effort was expended.

As programmer's, we're accustomed to thinking our unique skill is in writing *code*, I suspect Grothendieck would claim the true path is in modeling the problem skillfully with *data structures*. If the data structures are well chosen, the code follows without effort.

## The Darkness
This approach does not work equally well for all problems, indeed, sometimes the code one must write is essentially and irreducibly hard.

Consider the string parsing needed to read the string representation of the stacks into our data structure, here's the code I wrote in 2022 to do that:

```python
def parse_harbor(stackstr: Iterable[str]) -> Harbor:
    bottomup = list(stackitr)[:-1]
    harbor: Harbor = Harbor([[] for _ in range(N_STACKS)])
    for line in bottomup:
        tokens = [
            ''.join(x for _, x in g[1])
            for g in groupby(
                enumerate(line),
                lambda t: t[0] // N_CHARS_IN_BOX_TOKEN
            )
        ]
        boxes: List[Box] = [t.strip('[] ') for t in tokens]
        for box, stack in zip(boxes, harbor.iter_stacks()):
            if box: stack.append(box)
    return stacks
```

This is awful and impenetrable, but I'm unsure if it can be any other way. String parsing like this seems like the sort of task that is irreducibly complicated. The best we can do is isolate this bit from the more organized part of our program, and test it thoroughly enough to rely on its correctness.

I've found this a common pattern in Advent of Code puzzles, parsing the input is often irredeemably messy, but once the data is organized into clean data structures, the second half can be completed cleanly and transparently.

I've seen this described as an on onion. The shitty stuff should happen on the outside layer of your program, that interacts with the "outside world". Once it's inside, organize it into nice data structures so that operations (i.e. code) are as self evident as possible.

## Closing Remarks

![Grothendieck as an Wizard](/img/grothendieck-wizard.png)

This messy reality eventually came for Grothendieck. His mathematical life's work was all aimed towards a proof of the *Weil Conjectures*, of which there are three. The first two fell two his methods. The final did *not*, and remained for his student Deligne to overcome. To Grothendieck's disappointment, this last conjecture has never fallen to his approach, it required clever and difficult analysis, and still does.