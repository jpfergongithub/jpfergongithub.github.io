+++
date = '2019-03-07T16:46:17-05:00'
draft = false
title = 'Matrix math'
+++

On and off for about a year now, we've been working on a paper that tries to understand how changes to the US industrial structure have affected racial segregation between US workplaces. We think that decompositions of Theil's information statistic at the industry level give us a window into these changes. Details to follow in the paper we're writing.

We've had a problem, though. I wrote all of our original code for calculating Theil's $H$ in Stata. I did this because I know Stata quite well, but it is terrible at handling recursion. This for example is the code I wrote for calculating a three-level decomposition. And it's *just* the code, with all comments and data-wrangling removed:

```stata
foreach race of global races {
    forvalues o = 1/9 {
        generate phi_aj`o'_`race' = cond(w_aj`o' == 0, 0, `race'`o'/w_aj`o')
        generate E_aj`o'_`race' = cond(phi_aj`o'_`race' == 0, 0, ///
            phi_aj`o'_`race' * (ln(1/phi_aj`o'_`race')/ln(2)), 0)
    }
}
foreach race of global races {
    generate phi_aj_`race' = (`race'/w_aj)
    generate E_aj_`race' = cond(phi_aj_`race' == 0, 0, ///
        phi_aj_`race' * (ln(1/phi_aj_`race')/ln(2)), 0)
}
bysort year $area: egen w_a = total(w_aj)
foreach race of global races {
    by year $area: egen w_a_`race' = total(`race')
    generate phi_a_`race' = w_a_`race'/w_a
    generate E_a_`race' = cond(phi_a_`race' == 0, 0, ///
        phi_a_`race' * (ln(1/phi_a_`race')/ln(2)), 0)
}
egen a_tag = tag(year $area )
by year: egen w = total(w_a) if a_tag
by year $area: replace w = sum(w)
foreach race of global races {
    by year: egen w_`race' = total(w_a_`race') if a_tag
    generate phi_`race' = w_`race'/w if a_tag
    generate E_`race' = cond(phi_`race' == 0, 0, ///
        phi_`race' * (ln(1/phi_`race')/ln(2)), 0) if a_tag
}
foreach race of global races {
    forvalues o = 1/9 {
        drop phi_aj`o'_`race'
    }
    drop phi_aj_`race'
    drop phi_a_`race'
    drop phi_`race'
}
forvalues o = 1/9 {
    generate E_aj`o' = E_aj`o'_white + E_aj`o'_black + E_aj`o'_hispanic ///
        + E_aj`o'_other // occupation
}
generate E_aj = E_aj_white + E_aj_black + E_aj_hispanic + E_aj_other // est
generate E_a = E_a_white + E_a_black + E_a_hispanic + E_a_other // area
generate E = E_white + E_black + E_hispanic + E_other if a_tag // US
    by year $area: replace E = sum(E)
forvalues o = 1/9 {
    generate H_aj`o'_cmp = (w_aj`o'/w_aj) * ((E_aj - E_aj`o')/E_aj)
}
egen H_aj = rowtotal(H_aj1_cmp H_aj2_cmp H_aj3_cmp H_aj4_cmp H_aj5_cmp ///
    H_aj6_cmp H_aj7_cmp H_aj8_cmp H_aj9_cmp)
generate H_a_btw_cmp = (w_aj/w_a) * ((E_a - E_aj)/E_a)
by year $area: egen H_a_btw = total(H_a_btw_cmp)
generate H_a_win_cmp = ((w_aj*E_aj)/(w_a*E_a)) * H_aj
by year $area: egen H_a_win = total(H_a_win_cmp)
generate H_a = H_a_btw + H_a_win
generate H_btw_cmp = (w_a/w) * ((E-E_a)/E) if a_tag
by year: egen H_btw = total(H_btw_cmp) if a_tag
generate H_win_btw_cmp = ((w_a*E_a)/(w*E)) * H_a_btw if a_tag
generate H_win_win_cmp = ((w_a*E_a)/(w*E)) * H_a_win if a_tag
by year: egen H_win_btw = total(H_win_btw_cmp) if a_tag
by year: egen H_win_win = total(H_win_win_cmp) if a_tag
by year: generate H = H_btw + H_win_btw if a_tag
by year: generate HwithOcc = H_btw + H_win_btw ///
    + H_win_win if a_tag
by year $area: replace H = sum(H)
by year $area: replace HwithOcc = sum(HwithOcc)
```

I think you'll agree with me that this is ridiculous. It doesn't help that Stata insists on calling its variables "macros" and has an extremely particular syntax for referencing them, or that it insists on holding one data set in memory at a time...or that its programming language has even thornier syntax for defining helper functions. Things like this are why I've been trying to learn R. I don't love R, but it has strengths that offset many of Stata's weaknesses, and it plays better with other software.

By comparison, at one point last year I wrote some python to build a hierarchical tree structure in which to hold data like these. Once I had that tree structure built, I could write helper functions and ultimately a function like this one:

```python
def theil(tree, unit, levels):
    ''' Size-weighted sum of entropy deviations of a unit's sub-units. '''
    the_theil = 0
    if levels == 0:
        for subunit in tree.children(unit):
            the_theil += theil_cmp(tree, subunit.identifier)
    else:
        for subunit in tree.children(unit):
            the_theil += theil_cmp(tree, subunit.identifier)
            sub_weight = subunit.data.total() / tree[unit].data.total()
            the_theil += sub_weight * theil(tree, subunit.identifier,
                                            levels - 1)
    return the_theil
```

Hey, look, it calls itself recursively! The structure of the code reflects the structure of the computation! Obviously we should go this route!

This led to another problem, though. I can write code in python, but I can't necessarily write *fast* code in python. All of the functions I built to calculate the statistics were wicked fast, but the functions that actually built the data structure were unforgivably slow. A full set of analyses on our data, from building the structure through spitting numbers into a matrix, could take up to 40 minutes. Stata by comparison ran in less than 10.

That might feel OK. We're not writing production code, after all--we're academics! And code that's easy to modify, or generally to understand and work with after you've been away from it for a while, is really important. I learned this well in graduate school. The first time you get a revise-and-resubmit asking for further analyses, and you open up the project's code base and can't even remember which file calls which file, never mind what they each do...when you find yourself drawing a dependency graph for your *own* scripts, it's time to focus on writing clearer code.

All that said, though, 40 minutes was still unacceptable. In this project, we needed a way to distinguish what are basically random movements of the various components of the statistic from "real" shifts. There are no analytically defined standard errors for Theil's $H$, though--what statistical papers exist on the topic basically amount to a flame war about how every prior attempt is garbage. The way to proceed under such conditions is to simulate a probability distribution with the data at hand, either through bootstrapping or via a permutation test. Yet these techniques require you to perform a given calculation not once but hundreds or thousands of times. If we were to go with 1,000 repetitions, our projected compute time shot from 40 minutes to more than three months...or more than three weeks if we used the thorny Stata code.

Once we realized this, we set the project aside for several months. This is reasonable enough when you have a tough problem and a busy job. In between, we literally collected and analyzed data for another paper, wrote a draft, workshopped it, and submitted it to a journal. So it's not like we were idle. But this one has been gnawing at me. Finally in January I wrote my co-author and suggested that we book time at the end of February to sit down and finally figure this out.

Along the way, we decided to try this next in R. The python approach was fundamentally sound, but my co-author's as fast in R as I am in Stata, and we could just write and trouble-shoot code faster that way. Plus, if we could speed things up in R, it would be easier to share with other academics.

Like all statistical languages, R is optimized for fast vectorized calculations. And so I wanted to work out as much of this as I could in those terms, ideally before I got to Cambridge. So, I had the idea to ignore programming languages entirely and to try to write everything as matrix math.

This took some reviewing for me. I've never taken a full linear algebra class, and my shakiness with parts of it, more than anything about statistics *per se*, was often what tripped me up in econometrics classes. But here it seemed the right tool for the job.

I ultimately wrote [this document](https://www.jpferguson.net/programming.pdf). When I'm working on research and not sure about my math, I'll often write documents like this one. I find that the rigor involved in typesetting math, however tedious, and then seeing my handwritten work typeset often helps me spot errors I've made. Much of it often ends up in an article's appendices, and it can be useful for future researchers to see how you did something, in all its detail.

This time, it really paid off. It's straightforward to write matrix math in R, and so translating from the document to code was comparatively painless:

```R
p_agj <- (R %*% j)
p_ag <- (S_ag %*% (t(S_ag) %*% (R %*% j)))
p_a <- (S_a %*% (t(S_a) %*% (R %*% j)))
p <- (t(i) %*% (i %*% (R %*% j)))

w_agj <- p_agj / p_ag
w_ag <- p_ag / p_a
w_a <- p_a / p

phi_agj <- R / ((R %*% j) %*% t(j))
phi_ag <- (S_ag %*% (t(S_ag) %*% R)) / (S_ag %*% (t(S_ag) %*% ((R %*% j) %*% t(j))))
phi_a <- (S_a %*% (t(S_a) %*% R)) / (S_a %*% (t(S_a) %*% ((R %*% j) %*% t(j))))
phi <- (t(i) %*% (i %*% R)) / ((t(i) %*% ((i %*% R) %*% j)) %*% t(j))

E_agj <- -1 * ((phi_agj * tri(phi_agj)) %*% j)
E_ag <- -1 * ((phi_ag * tri(phi_ag)) %*% j)
E_a <- -1 * ((phi_a * tri(phi_a)) %*% j)
E <- -1 * ((phi * tri(phi)) %*% j)

d_agj <- E_agj / E_ag # This never gets used
d_ag <- E_ag / E_a
d_a <- E_a / E

d_agj <- ifelse(is.na(d_agj), 0, d_agj)
d_ag <- ifelse(is.na(d_ag), 0, d_ag)
d_a <- ifelse(is.na(d_a), 0, d_a)

h_agj <- w_agj * ((E_ag - E_agj) / E_ag)
h_ag <- S_ag %*% (t(S_ag) %*% h_agj)
h_a <- S_a %*% (t(S_a) %*% (w_ag * d_ag * h_ag))
h <- t(i) %*% (i %*% (w_a * (E - E_a) / E)) # Not used

h_agj <- ifelse(is.na(h_agj), 0, h_agj)
h_ag <- ifelse(is.na(h_ag), 0, h_ag)
h_a <- ifelse(is.na(h_a), 0, h_a)
h <- ifelse(is.na(h), 0, h) # Not used

H_g <- S_g %*% (t(S_g) %*% (w_a * d_a * w_ag * d_ag * h_agj))
```

That isn't as pretty as the python code, but the parallels at different levels of calculation are *much* clearer, and eventually we'll wrap and collapse all of this into some recursively defined helper functions.

Two things to note here. The first is that, when we first tried to run this, lines like `p_ag <- S_ag %*% t(S_ag) %*% R %*% j` were causing our machines to hang and crash. We were getting ready to abandon this approach when it hit me: $\mathbf{S_{ag}}$ is an $N \times K$ matrix, and so $\mathbf{S_{ag}}\mathbf{S_{ag}}^T$ is an $N \times N$ matrix. If, as here, $N \gg 100,000$, that thing can have tens or hundreds of billions of elements. Yet we don't need it! Our ultimate goal was an $N \times 1$ vector, so if we just changed the order of operations we could post-multiply the second term by the third and fourth, then pre-multiply that by the first. At no point did we actually need to create the gigantic matrix.

We did that, and the operation went from crashing our machines to finishing in a fraction of a second.

The second was with how we generated $\mathbf{S_{ag}}$ in the first place. We were using the `fastdummies` library in R. It looked like `fastdummies` wasn't fast, though. Generating these matrices took up to 30 seconds on our real dataset. After some searching, we found that we could generate sparse matrices with the `matrix` library...and what had been taking 30 seconds now took .3 seconds.

With changes like these, we sped up the computation 500 or more times. Our projected compute has fallen from more than three months to fewer than six hours.

I have two reasons for writing this all up. The first is that someone might find it helpful, both in its specifics and in the general sense of seeing how those of us with little formal training in computer science can still go about solving practical problems in computation. None of anything I did here is rocket science; it just takes persistence and rethinking how to approach the problem.

The second reason, of course, is that I figured something out in an area where I usually feel like a fraud, and I'm proud of it. It isn't a brag to share such moments with people. It's a chance to feel good about yourself, justifiably so, and have people validate those feelings. Given how often when doing research we feel rotten, it's important to mark the really triumphant moments, too.