+++
date = '2020-01-19T20:54:16-05:00'
draft = false
title = 'Fun with modulo and Boole'
+++

Segregation research has what is called the "Checkerboard problem." Back in 1983, Michael White pointed out that most segregation measures look at how evenly distributed groups are between sub-units in a larger unit, like neighborhoods in a city, but ignore the spatial arrangement of groups within and between those units. The problem's name comes from imagining a city where each census tract looks like a checkerboard. Assume the city is half white and half black. A census tract that has houses arranged in the classic checkerboard pattern would be half white and half black, like the city as a whole; but so too would a tract where all the squares on the left-hand side of the board are white and all the squares on the right-hand side are black. The "problem" here is that traditional segregation measures would say that both of these tracts are unsegregated, but the spatial arrangement tells us otherwise:

![foo](/boole1.webp)

One way around this would be to look at segregation between sub-tracts, but this has practical limitations. Data are only recorded to a certain degree of resolution most of the time. Instead, it makes sense to develop spatially weighted measures of segregation, as Reardon and O'Sullivan did in a 2004 paper. In that paper, they presented a nice graphic that shows the different dimensions of spatial segregation, contrasing Evenness/Clustering versus Isolation/Exposure:

![foo](/boole2.webp)

I'm working on a paper about spatial segregation, and I wanted to reproduce this diagram. The most obvious way to do that for most people would be to fire up PowerPoint, get to copying and pasting, and eventually save the result as a graphic you could paste into Word or even a LaTeX document. Sure, you *could* do that, but the previous posts on this blog strongly imply that I'm not most people. Implementing this programmatically, as a vector graphic, seemed like a fun challenge.

This meant it was time for TikZ. For the uninfected, TikZ is a way of building vector graphics that plays extremely well with LaTeX. Its name is a recursive German acronym for "TikZ ist kein Zeichenprogramm," or "TikZ is not a drawing program." This passes for humor among some nerds. I've used TikZ on and off for a decade. When everything works, it can produce some truly lovely and clean data-driven graphics. But it is *always* a pain in the ass. Why do I keep coming back to it? Because each time, I learn a little more, and I'm addicted to that feeling.

The checkerboards in the first figure, for example, I drew in TikZ. And the code to do so is genuinely not that complicated:

```tex
\documentclass{standalone} %
    \usepackage{tikz}          % Vanilla headers from LaTeX
    \begin{document}           %

    \begin{tikzpicture}[scale=0.5]
    \foreach \x in {0,...,7}         % Loops! We all know loops!
      \foreach \y in {0,...,7} {     % Iterate to draw a checkerboard.
        \pgfmathsetmacro{\color}{iseven(\x+\y) ? "black" : "white"} % ifthenelse
        \draw[fill=\color] (\x,\y) circle(0.4cm); % Conditional fill
      }
    \begin{scope}[xshift=9cm] % Move all reference right of what we've drawn
    \foreach \x in {0,...,7}                     % More loops!
      \foreach \y in {0,...,7} {                 %
        \pgfmathsetmacro{\color}{\x>3 ? "black" : "white"} % ifthenelse again
        \draw[fill=\color] (\x,\y) circle(0.4cm); % Conditional fill
      }
    \end{scope}
    \end{tikzpicture}
    \end{document}
```

For the most part, that code is going through loops to draw circles: the inner `\foreach \y` loop builds a column of circles, and the outer `\foreach \x` loop builds a row of these columns. Probably the most offputting lines in there are the ones that start with `\pgfsetmacro`. And they *should* be offputting! Those lines are your first indication that the whole TikZ language sits on top of *another* language: PGF, or the Portable Graphics Format. Confused yet?

Here's the thing about PGF, in my opinion: it's hot garbage. Or rather, it's an astonishingly low-level language, given how high-level TikZ itself is. This means that, whenever you have to do something tricky in your TikZ drawing, you punch through the glossy front end and fall a *long* way, down to a programming language that borrows from something clever that Don Knuth did in the 1970s.

\(Before I go on, let me clarify how the if-then-else operator works in PGF. `a ? b : c` means "If `a` is true then `b`, else `c`." Therefore here `\pgfsetmacro` is setting the value of a variable (or "macro"--Stata users *brrrrrrup brrrrrrup!*) called `color` based on the if-then-else condition in the following set of braces. In the first case, I'm adding the `x` and `y` indices of each circle, starting at the lower-left of the board, and coloring the circle black if the index is even. In the second, I'm coloring all circles in the column black if the column index is 4 or greater.)

# Clock math
The upper panels of Reardon and O'Sullivan's figure aren't that hard. They're basically repeating patterns in one dimension. *More or less* every third circle in the upper-right is black, for example. It gets a little janky as you move between rows, but we can work with that.

As soon as you hear "regular repeating pattern" you should think loops, right? NO. I've seen people in my field do that before, but--especially when you already have numbers you're looping over--you should be using the modulus function. This is not the place for an introduction to finite math, but if you want to be a quantitative social scientist and haven't spent much time with the modulus function, you have a homework assignment.

Modular arithmetic is just math done on a clock face. Everyone understands that three hours after 10 o'clock is 1 o'clock. That means that everyone understands that $10 + 3(mod 12) = 1$ or that `mod(10+3,12)=1`. Modular arithmetic shows up all over the place, especially in cryptography (Diffie-Helman key exchange, anyone?), but it's also a bog-standard tool for generating repeating sequences. Thus the code for generating the upper-right checkerboard isn't that much more complicated than our most basic checkerboard:

```tex
\foreach \x in {0,...,9} 
        \foreach \y in {0,...,11} {
          \pgfmathsetmacro{\shift}{mod(\y,2) ? .5 : 0}
          \pgfmathsetmacro{\bump}{mod(\y,2) ? 0 : 1}
          \pgfmathsetmacro{\color}{mod(\x+\bump,3) ? "white" : "black"}
          \draw[fill=\color] (\x+\shift,\y) circle (0.4cm);
      }
```

![foo](/boole3.webp)

There are only a few changes. I set a macro `shift` that scooches the odd-numbered rows over half the distance of a circle, to resemble the original. (Remember that all the row and column indices start from zero, and they start at the *lower-left*--I wasted a couple of hours on this code before I remember that I shouldn't be numbering and counting from the *upper*-left!) This means that `shift` affects the *placement* of the circles but not their *color*. Note that it isn't invoked until the actual `\draw` command. Meanwhile `bump` gives us an offset so that which exact circles are black alternates between rows. Alternating rows are actually only offset by one, but when combined with the shifting of the rows it looks like more.

Pay close attention to how the `mod()` operator works here. Row indices are being mapped onto a two-digit clockface, so $[0,1,2,3,\ldots{},8,9,10,11] \rightarrow [0,1,0,1,\ldots{},0,1,0,1]$. This is how `mod(\y,2)` lets me flag alternating rows. Then within the rows, the column indices are being mapped onto a three-digit clockface. On the even-numbered rows, $[0,1,2,3,4,5,6,7,8,9] \rightarrow [0,1,2,0,1,2,0,1,2,0]$. On the odd-numbered rows, because of the bump, $[1,2,3,4,5,6,7,8,9,10] \rightarrow [1,2,0,1,2,0,1,2,0,1]$. If you look closely, you'll see that the zeros in each of those right-hand sequences corresponds to a black circle. This is what that last if-then-else statement in the code does for us: any non-zero result receives the color "white" while zeros receive the color "black."

The upper-left panel of the figure works the same way, with different values for `\bump` and for the base of the modulus (Bonus points if you can figure them out). I had these two figured out within an hour of a cold start. The bottom two panels are more complicated, though. notice that they have multiple patterns, depending on the row index. And that lovely if-then-else operator, `a ? b : c`, *does not nest*.

If I were going to make those lower panels, I was going to have to spend more quality time with PGF.

# This is where the whining starts

Let me note that the idea for this post was born while I was wrestling fruitlessly with row and column indices within modular arithmetic earlier today. I kept getting bizarre results that were making me doubt my understanding of PGF, and possibly of my sanity. Sometimes the resulting pictures only made sense if you started enumerating indices from 1 rather than from 0; sometimes it looked like the if-then-else operator returned the first value for *zero* results and the second for everything else. Because the modular base is small \(i.e., two\), it was easy to see the results as flipped rather than offset. I pulled on my hair a lot this afternoon, and my confusion was not helped by the syntax of PGF.

Consider the lower-right panel. Here there are three different line patterns. The first row is all white, the second has every third circle colored black, and the third row has every third circle *offset by one* colored white. The classic way to deal with three cases is a nested if-then-else: `a ? b : c ? d : e`. Alternatively you could write that as `a ? c ? d : e : c`, but it doesn't matter because neither works. Instead, you need to use a very low-level PGF command, `\ifnum`.

This routine has some quirks. For starters, it only accepts integer arguments, which means that the floating-point results of, say, the `mod()` function will produce errors. Hence `int(mod())` will be necessary. Nor can you pass functions as arguments to `\ifnum` which can then be evaluated to produce variables. Instead the functions have to be evaluated outside the loop and passed to it as `\pgfmathresult`. As you might guess, each of these quirks took some time to sort out.

Eventually I could produce the lower-right panel, including the offset of the alternating "sets" of triangles. This was eased by the fact that the logical tests that you have to pass to the if-then-else operator are not that complicated: either a circle's index modulo 3 is zero or it isn't. If it is, color it black in one case, white in the other.

But with that said, turn your eyes back to the lower-*left* panel of Reardon and O'Sullivan's diagram. Consider in particular the third row up from the bottom, the one with two black circles filled in. Here a circle should be black if its index modulo 5 is zero *OR* if its index modulo 6 is zero. Suddenly the logic becomes more complicated.

It will not work to test for whether one condition *or* the other is satisfied here. But to understand why, we need to spend a little time with Boolean algebra.

Once again, here I describe something that people with a background in mathematics or computer science will find very basic. But I stress that, in most of the social sciences, this sort of thing is maybe mentioned in an early class on probability theory and then never disinterred. Furthermore, the way these things work in the context of a programming language are *never* discussed.

Take the humble `or()` operator. In Boolean algebra, $\mathrm{or}(a,b)$ evaluates as True if either $a$ or $b$ evaluates as true. Yet in many programming languages, including PGF, `or(a,b)` evaluates as True if either `a` or `b` returns a non-zero value. This is a problem for the modulus procedure, because most combinations will return a non-zero value. For example, `or(mod(\x+2,6),mod(\x+1,6))` will return a True value on *every* index, because either one of the moduli is non-zero or both are. I need to fill in circles with black only when one of those conditions is non-zero *and the other one is not*. This is the domain of `xor()`, the exclusive-or operator. Which has all sorts of uses, especially in binary arithmetic--but PGF does not have an `xor()` operator!

That said, the point of Boolean algebra is to be able to build more complicated logical expressions from simpler ones. If you play around for a while, you'll realize that $\mathrm{xor}(a,b) \equiv \mathrm{and}(\mathrm{or}(a,b),\mathrm{not}(\mathrm{and}(a,b)))$. I used that to generate that row. But then I had to handle the row above it, where there are *three* tests to run, and where I want to shade a circle if *only one* of the three moduli returns zero. This sort of test can be implemented as nested exclusive-ors: `xor(xor(a,b),c)`. But what does that look like if you do not have a native exclusive-or operator? Behold:

$\mathrm{and}(\mathrm{or}(\mathrm{and}(\mathrm{or}(a,b),\mathrm{not}(\mathrm{and}(a,b))),c),\mathrm{not}(\mathrm{and}(\mathrm{and}(\mathrm{or}(a,b),\mathrm{not}(\mathrm{and}(a,b))),c)))$

I will admit that I patted myself on the back when I typed that in and it worked the first time! Thus the code for the lower-left panel:

```tex
\foreach \x in {0,...,9}
        \foreach \y in {0,...,11} {
          \pgfmathsetmacro{\shift}{mod(\y,2) ? .5 : 0}
          \pgfmathsetmacro{\color}{"white"}
          \pgfmathsetmacro{\testx}{\x+3*int(\y/6)}
          \pgfmathparse{int(mod(\y-1,6))} % Tip of triangle
          \ifnum\pgfmathresult=0 
            \pgfmathsetmacro{\color}{mod(\testx+2,6) ? "white" : "black"}
          \else
            \pgfmathparse{int(mod(\y-2,6))} % 2nd of triangle
            \ifnum\pgfmathresult=0
              \pgfmathsetmacro{\color}{
                and(or(mod(\testx+2,6),mod(\testx+1,6)),not(
                and(mod(\testx+2,6),mod(\testx+1,6)))) ? "black" : "white"}
            \else
              \pgfmathparse{int(mod(\y-3,6))} % 3rd of triangle
              \ifnum\pgfmathresult=0
                \pgfmathsetmacro{\color}{
                  and(or(and(or(mod(\testx+3,6),mod(\testx+2,6)),not(and(
                  mod(\testx+3,6),mod(\testx+2,6)))),mod(\testx+1,6)),not(and(
                  and(or(mod(\testx+3,6),mod(\testx+2,6)),not(and(
                  mod(\testx+3,6),mod(\testx+2,6)))),mod(\testx+1,6)))
                  ) ? "white" : "black"}
              \else
                \pgfmathparse{int(mod(\y-4,6))} % top of triangle
                \ifnum\pgfmathresult=0
                  \pgfmathsetmacro{\color}{
                    and(or(mod(\testx-1,6),mod(\testx-2,6)),not(
                    and(mod(\testx-1,6),mod(\testx-2,6)))) ? "white" : "black"}
                \fi
              \fi
            \fi
          \fi
          \draw[fill=\color] (\x+\shift,\y) circle (0.4cm);
      }
```

![foo](/boole4.webp)

Hopefully we can all agree that the readability of the code suffers somewhat in this case. Yet at that point I could reproduce the entire figure in code:

![foo](/boole5.webp)

# Am I just an idiot?

Literally as I wrote this blog post, complaining about the lack of an `xor()` operator in PGF, I realized that you could do what I needed without calling an `xor()`. You see, you can specifically test whether the modulus evaluates to zero. Thus instead of specifying

```tex
and(or(mod(\testx+2,6),mod(\testx+1,6)),not(
    and(mod(\testx+2,6),mod(\testx+1,6)))) ? "black" : "white"}
```

I could instead write

```tex
or(mod(\testx+2,6)==0,mod(\testx+1,6)==0) ? "black" : "white"}
```

Similar shortening could be done for the three-way test. This thus raises the question of whether I'm just an idiot.

But let's take a step back. Am I not *already* an idiot for spending literally a day hacking away at this, when I could have produced it in an hour or so in something like PowerPoint?

I wrestle with this question all of the time. I like going down rabbit holes when I am working on things, and I like understanding how things work. I started trying to implement this figure in TikZ in the first place because I'd never really mastered the language's looping and conditional structures, and it seemed like a chance to learn those. I've definitely learned a ton! But I always have to balance that learning against the opportunity cost. Should I have gone skiing today instead?

I once discussed TikZ with some PhD students at Stanford while Mike Hannan was in the room. Mike told the students in no uncertain terms that, cool as this stuff was, while you are *writing* the paper you should make a sketch on a piece of paper and then set it aside. Only at the very end should you wrestle with figure generation, because it's such a time sink. I recognize the value in that injunction, and in general I try to follow it. But sometimes it's down in these rabbit holes that I learn things that speed up my work somewhere else. I use modular arithmetic all of the time, for example, not only in data analysis but even in laying out tables in Salesforce to help our staff award students financial aid.

Perhaps this has become a blog whose posts are triggered by my obsessions. But even then at least it may have the value of showing people how much work exists behind the work, and convincing you other obsessives that you are not alone!