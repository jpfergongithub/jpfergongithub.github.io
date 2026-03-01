+++
date = '2021-10-19T12:27:28-05:00'
draft = false
title = 'Crosswalks, or error propagation'
+++

In the summer of 2021, my research assistant Hasnain Shaikh and I made great progress cleaning up the Unfair Labor Practice data that I have mentioned elsewhere. I have written before about the challenges with making sense of geography in these records, because the National Labor Relations Board switched versions of their coding schemes without documenting that. (Also they had a lot of typos on ancient keyboards.) Eventually we reached the point where we could make some simple plots showing the relative intensity of ULP charges in various states, with the states scaled by their union membership.

![foo](/1974_cropped.webp)

![foo](/1984_cropped.webp)

In those two images above, for example, you can see 1974 and 1984. The upswing of employer intimidation and firings for union activity, commented on at the time, is well under way. The production of the cartograms is a story for another time (we did these with [QGIS](https://qgis.org/en/site/), but dang we need a python-only toolchain!). Given that these were text strings of punched-card records not too long ago, though, I'll call this progress.

The real challenge in these data though is getting the coding of *industry* right. All industry-classification systems are compromises, but the challenge inheres in converting between them. Industry change means that these schemes need frequent updating. Said updates can be drastic; the shift from the Standard Industrial Classification (SIC) codes to the North American Industry Classification Scheme (NAICS) codes around the turn of the century famously breaks many longitudinal series. The further back in time you go, the worse this can be.

The data we use run through 1999, which spares us the SIC/NAICS mess, but there are still multiple versions of the SIC to deal with. For the most part, industry classification in the data look like this:

![foo](/data_overlap_small.webp)

This isn't bad, and many crosswalks exist for moving between SIC classification schemes. I think it best to standardize these records to the 1987 SIC. This will produce the numerator for certain analyses. But what about the denominator?

When looking at geography, at least at the state level, a denominator is easy. [Hirsch, Macpherson, and Vroman's work](http://unionstats.com/MonthlyLaborReviewArticle.htm) gives us state-level union density annually back to 1964. For industry, we need union density by industry. Here the troubles begin: the Bureau of Labor Statistics is very clear that they have only gathered consistent data on union membership and coverage by industry since 1983. Even where they do have it, they have used *yet another* scheme: the Census Industry Code lists. This means the data to be made commensurate really look more like this:

![foo](/data_overlap.webp)

What is going on in the 1970s? There are references to data that the BLS gathered on union density between 1973 and 1983. In the data appendix to [Farber et al.'s 2021 QJE (gated)](https://doi-org.proxy3.library.mcgill.ca/10.1093/qje/qjab012), for example, the authors write that "The CPS first asks respondents their union status in 1973, and then only in selected months until 1983 from which time information on union status was collected each month in the CPS as part of the outgoing rotation group supplement" \(113\). [Shawn Perron's 2022 Social Currents (gated)](https://journals.sagepub.com/doi/epub/10.1177/23294965221089914) has more information: "The [CPS] has asked about employment status, household demographics, and other labor force characteristics, such as union membership and wages since 1973. Extracts are compiled by the National Bureau of Economic Research \(NBER\). ... These estimates draw on the May extracts of the CPS between 1973 and 1982 and the merged outgoing rotation group \(MORG\) from 1983 onward" \(375\). Once you know what you are looking for, it is easy to find the [CPS's archive of May extracts](https://www.nber.org/research/data/current-population-survey-cps-may-extracts-1969-1987). Not for the first time on this projectb, once I have the right search terms, the data are trivial to grab; the problem is finding those search terms.

Consider what this diagram implies:

- Before 1973, no industry-density information is available
- 1973-1982, we must match 1972 SIC to the May CPS extracts
- 1983-1987, we must match 1972 SIC to 1980 CIC
- 1987-1992, we must match 1987 SIC to 1980 CIC
- 1992-1999, we must match 1987 SIC to 1990 CIC

Again, this isn't *too* bad. The good news is that, at least in recent years, [CIC codes appear derived from NAICS codes](https://www.census.gov/topics/employment/industry-occupation/guidance/code-lists.html). This gives me hope that the same will hold for the era covered by SIC codes.

Thus we could eventually do with industry what we have done with geography: plot counts of ULP charges across industry, scaled by industry density. \(Whereas geography lends itself to cartograms, industry probably needs something like a mosaic plot.\) But before I do so, it's worth discussing error propagation.

# Error propagation in crosswalks

The social sciences \(apart from some parts of psychology\) don't spend much formal time on error propagation, or the [propagation of uncertainty](https://en.wikipedia.org/wiki/Propagation_of_uncertainty). I think that this is because direct measurement is so rare. We usually get our data from somewhere else. While we learn about measurement error, and understand that classical measurement error can introduce noise \(but not necessarily bias\) into our analyses, we spend very little time trying to quantify it. We also place too much emphasis on statistical significance *per se*, and downplay getting parameter estimates right. The situation is different in other sciences, where hypotheses are often about specific parameter values and where experimentalists often generate the data they analyze.

Assume we want to know how variable $y$ responds to changes in variable $x$
 -- i.e., correlation. Pearson's correlation coefficient $\rho_{xy}$ is just $\frac{\mathrm{cov}(x,y)}{\sigma_x \sigma_y}$, that is, how much $x$ and $y$ vary together as a share of how much they vary individually. (Ever notice how much $\rho$ looks like a Jaccard index?) Assume that we have classical measurement error; we measure $x$ with a lot of noise. A noisy measure is going to have more internal variance, which increases $\sigma_x$. Noise will also increase the variance of $x$ relative to any given variation in $y$, which by construction will decrease $\mathrm{cov}(x,y)$. Noise thus shrinks the numerator of $\rho_{xy}$ while inflating its denominator. This is why classical measurement error biases coefficients toward zero.

So far, so good--this is basic stats. It's also why experimental scientists obsess over accurate measurement at every stage: measurement error in any stage of a variable's construction will propagate. Social scientists know that measurement error happens, but because we usually inherit our data from some other entity that did the primary measurement, we limit ourselves to discussing how it *could* affect our results. Most often, we assume that measurement error will bias our results toward zero, and therefore treat any results as conservative estimates of effects.

There is a place though where social scientists generate a lot of data even though they make no primary measurements: crosswalks. This is the generic term for our heroic and often flailing attempts to make incompatible datasets compatible. One of my heroes in this area is David Dorn, whose [data page](https://www.ddorn.net/data.htm), replete with crosswalks, has bailed out countless researchers. \(Seriously, people like David should set up Patreon accounts. Though perhaps citations do the same, in the currency of academic reputation?\)

I want to distinguish two types of crosswalks. One is a straightforward recoding. You might have US states recorded as full names, two-digit abbreviations, or FIPS codes. This needs fixing, and there are [whole packages](https://cran.r-project.org/web/packages/crosswalkr/vignettes/crosswalkr.html) for it, but there is no uncertainty in this process. I don't care about these. The second type is when we make guesses \(educated guesses, but guesses nonetheless\) about the code we should assign an observation in scheme 2, given the code it was assigned in scheme 1. There is ample room for measurement error here.

\(Aside: crosswalks are just joins, to use the SQL term. There are a ton of [different types of joins](https://en.wikipedia.org/wiki/Join_(SQL)), though. The simple-recoding type of crosswalk boils down to a one-to-one join between two tables of codes. At the other end, many-to-many joins are often indeterminate. Because we use "crosswalk" to cover all of these, I think that the term can hide more than it reveals.\)

I think the *order* in which you should build a complicated crosswalk is interesting. Say that you have data that's been encoded using three schemes, S70, S80, and S90, and that you want to standardize everything to S90. You could harmonize the S70 to S90 and the S80 data to S90. Alternatively, you could harmonize S70 to S80, *then* harmonize S70/S80 to S90. These aren't the same! Is there a formal way to choose between them?

I think that the answer lies in the data-generating process of the schemes themselves. If industry change is random then we can think about said change as retroactively applied mistakes in coding individual observations. This is a stochastic process, like Brownian motion. If I learned anything from [Scott Ganz's work](https://pubsonline.informs.org/doi/abs/10.1287/orsc.2019.1330?journalCode=orsc), it's that fixing a data point partway through a Brownian process reduces the variance of our estimates later on in the process. In these cases, it would make sense to harmonize S70 to S80, then harmonize S70/S80 to S90.

There are problems with that line of thinking, though. Even if the assume that the enumeration of possible industries in each scheme is appropriate (and we have little choice but to assume that), we don't necessarily know where to assign a firm in scheme 2, given solely its assignment in scheme 1. The best such crosswalks have probability weights that can be used to build transition matrices, but that means you're introducing assignment error *at each stage*. This is when we cycle back to error propagation. When are sequential probabilistic assignments with small(er) errors preferable to a single probabilistic assignment with large(r) errors?

This is a post that almost certainly will have an update. But, gosh, I'm curious whether other people have worked on this point.