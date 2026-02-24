+++
date = '2019-03-07T14:24:31-05:00'
draft = false
title = "Some blather about Theil's H"
+++

I was feeling guilty because I had not updated this blog in two weeks. Then I reminded myself that the whole *point* is that I put in entries here when I have time, and if I've delayed in doing so because I have been Living My Best Life(tm), that's a good thing!

I was in Cambridge last week, semi-covert, to work with my collaborator. We submitted one paper and *finally* broke a log jam on another that had stalled for several months. While the project is unrelated to the ULP-diffusion project I'm concentrating on in this blog, solving problems related to it also reminds me why I do research, and I think it is worth talking about. Also, this project, like the one on workplace segregation [that was covered on Vox](https://www.jpferguson.net/blog/blog-post-title-four-tl2th-m8k2z), uses Theil's information statistic to measure segregation. I do enough work with this thing, and have explained it enough times, that I might as well put it down here in the blog as well.

An intuitive definition of segregation is uneven division of a unit's population among its sub-units. Think of uneven division of a school district's students among its schools, or uneven division of a city's residents among its neighborhoods. You need a metric of how unevenly divided that population is. There are a lot of these, but we like Theil's statistic for several reasons. The most important is that it handles multiple groups gracefully. A lot of measures, including the popular index of dissimilarity, were devised for two-group cases. This is fine if you care about white/black or white/non-white, but they can give wonky results as the number of relevant groups increases. In America today, you really should use a multi-group measure.

The other great advantage of Theil's statistic is that it is *fractally decomposable*. Fancy words, but it just means that you can, say, separate segregation in the US into segregation *between* labor markets plus segregation *within* labor markets; then separate the latter into segregation *between* industries within labor markets, plus segregation *within* industries in labor markets, and so on. If your goal is to understand how changes at different levels might drive changes in segregation \(and that is ours!\), then this is an incredibly useful feature.

The statistic leverages the concept of entropy that Claude Shannon developed in his seminal 1948 article "[A Mathematical Theory of Communication](http://math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf)." A bit is informative to the extent that it is rare or unexpected. Imagine I were to feed you a letter at a time, and you were to try to guess the word that I was spelling. Before I speak, your possibility space is the entire vocabulary of English--that's the baseline level of entropy in the system. If I say "e," that reduces the search space for you some. But if I say "z," that reduces the search space a lot more. Because "z" is a rare letter and words starting with "z" even more so, knowing there's a "z" prunes the probability space a lot more than knowing there's an "e" does. It's this reduction in entropy that we define as information.

In the context of employment segregation, imagine we have a workforce. I choose a worker at random and ask you to guess their race. How much uncertainty do you have? That depends on the baseline level of entropy. If the workforce is all of one race, there's no entropy, and no uncertainty. As you add races, and as you distribute the workforce more evenly among however many races, your uncertainty increases. Thus if $\phi_r$ is the share of race $r$ in the workforce, entropy is defined as $\sum_r \phi_r \mathrm(ln)\phi_r$. \(Play around with that, keeping in mind that we'll define the log of a zero as zero. It works.\)

OK, so you have a baseline level of entropy in the workforce, and that specifies your uncertainty about the race of a randomly chosen worker. *Now suppose I tell you where they work.* How does that information affect your uncertainty?

- If there is *no* segregation, this is uninformative. Your uncertainty remains unchanged.
- If there is *total* segregation, this is extremely informative. Your uncertainty is removed completely.
- For in-between levels of segregation, your uncertainty is reduced proportionately to the level of segregation.

Theil's statistic leverages this idea. We calculate the entropy for the unit, $E$. We calculate the entropy for each of the $j$ sub-units in the unit, $E_j$. Then the statistic averages the sub-unit's deviations from the larger unit. It's a weighted average, where the weight is the sub-unit's size as a share of the unit, $p_j/p$. And we scale the entropy deviation by the unit's entropy, so that we can compare this measure across different types of units. That gives us this:

$$H = \sum_j \frac{p_j}{p} \frac{E-E_j}{E}$$

With me so far? Good. Here's where it gets fun.

Let's imagine that our workplaces are nested within groups. This happens all the time. In the US, for example, workplaces are scattered all over the country, and racial populations are also scattered--non-uniformly!--over the country. Thus there are a lot of latino workers in Californian worklplaces and far fewer in Maine workplaces. Does this difference represent segregation, though? It's a judgment call in each case, but in *this* case I would say no. No one who lives in California is likely to work in Maine and vice versa. Where people live of course isn't random, and a ton of people study that; but if you want to characterize and understand segregation among workers, you want to start with units that could feasibly be integrated. Another way to say this is, if all of the segregation between worklpaces in America could be reduced to the uneven geographic spread of different racial populations, then fretting about and intervening at the level of the workplace doesn't make much sense. We want a way to separate out these two, and Theil's $H$ gives us a way.

If the unit can be completely divided into $G$ mutually exclusive groups then Theil's $H$ can be decomposed into $G+1$ components: one for the segregation *between* groups and $G$ for the segregation within each group:

$$H = fancy$$

Notice that each of the within-group bits is just the simpler version of $H$. That's how the fractal nature of this statistic shows up. Those lower-level statistics are then put into a weighted sum, where the weights are the group's relative size *and* the group's relative diversity \(i.e., its relative entropy\).

I don't want to lose track of why this is useful. If you go back to my example, the first term above--that "Between-group" bit--would account for the different distribution of races between, say, California and Maine. This makes interpreting the second terms easier. *Given* the structure of the workforce in California, how segregated are workers between workplaces there? *Given* the structure of the workforce in Maine, how segregated are workers between workplaces there?

This also gives you a way to think about why the weights are what they are. Californian workplaces might be less segregated than "Downeaster" ones \(or whatever the Hell people from Maine call themselves--*I can't even be bothered to Google it*\). Yet California is a lot *bigger* than Maine, so what segregation there is affects more people. And California is a lot more *diverse* than Maine, so again the possible impact of segregation is bigger. Hence those weights.

You can keep doing this. We've calculated three-level decompositions that include segregation between occupations *within* workplaces, for example. But eventually the code for doing so starts to break your head. Or at least it does given how we'd written it! Solving *that* little problem will be a matter for future posts.