+++
date = '2019-01-21T13:26:14-05:00'
draft = true
title = 'A blast from my past'
+++

# A whole lotta context
In 2009, I attended a talk by [Bruce Western](https://sociology.columbia.edu/content/bruce-western) about the collapse of trade-union norms in the United States. He and Jake Rosenfeld had this paper where they analyzed the effect of trade-union membership on wage dispersion in various areas and times. They argued that "unions helped institutionalize norms of equity, reducing the dispersion of nonunion wages in highly unionized regions and industries." The fall of the house of labor, in turn, explained a non-trivial share of the growth in U.S. wage inequality in recent decades.

It’s a neat paper-–I mean, [it did well](http://www.asanet.org/sites/default/files/savvy/images/journals/docs/pdf/asr/WesternandRosenfeld.pdf)-–but I’ve never been OK with their argument that we could read a story of changing social norms into these data. Sociologists like the idea that many of our decisions are embedded in a moral economy, and that we rule out some actions not through any cost-benefit analysis but because we have been socialized to think that those actions are categorically wrong. I buy this idea, but it’s miserably hard to show. Never mind that we cannot put a probe in people’s heads and observe norm compliance. Many are the alternative reasons we could observe new patterns of behavior that have nothing to do with shifting *norms*. Here [Randall Collins](https://www.jstor.org/stable/2778745?seq=1#page_scan_tab_contents) deserves a shout-out:

> Of course, one may rescue the norms or rules as nonverbalizable or unconscious patterns which people manifest in their behavior. But such “norms” are simply observer’s constructs. It is a common, but erroneous, sleight-of-hand then to assume that the actors also know and orient their behavior to these "rules." The reason that normative sociologies have made so little progress in the past half century \[*prior to 1981, that is*\] is that they assume that a description of behavior is an explanation of it, whereas in fact the explanatory mechanism is still to be found. It is because of the potential for this kind of abuse that I believe that the terminology of norms ought to be dropped from sociological theory.

Gauntlet tossed down, Mr. Collins! I don’t think I’m ready to bin the term, but clearly there’s a bar we have to clear.

I mention all of this because the talk got me thinking. Changes in pay setting, by themselves, are interesting but hard to assert as changes in norms. What would be easier? The idea I had, sitting in that talk, was that you would want to look at something that had been widely accepted but then became censured or illegal; or, conversely, something that had once been censured or illegal that then became, well, *normal*.

With labor unions, we have something like that. The Wagner Act, which governs union formation in US workplaces, specifies various Unfair Labor Practices. These are things employers \(and, later, unions\) are prohibited from doing, under pain of legal penalties. They include intimidating workers during union-organizing campaigns, firing workers for union-related activity, and refusing to bargain in good faith with workers' legally certified representatives. For decades after the Wagner Act was passed in 1935, charges of ULPs were relatively rare. Starting in the 1970s, though, they started to take off.

Throughout the 1980s, people interested in industrial relations remarked on the increase in nominally illegal employer resistance to unions. The rate of ULP charges relative to union activity probably peaked in the 1990s. After that, they began a slight decline, but that’s relevant. By that point, “illegal” employer resistance had become so common, and attempts to sanction it so unsuccessful, that many union organizers seemed to abandon the formal complaint channel. Said differently, something that had once been censured or illegal had become, well, *normal*.

# A false start

I left that talk with a research idea. Wouldn’t it be neat to study the spread of ULP charges? No one had really done this before. I could imagine different predictions: that these had been more common in less profitable, more competitive industries, but had spread into the economic “core,” or that they had spread from more anti-union states and regions to more union-friendly ones. I had been working on labor-union representation-election data for my dissertation \(my first “real” academic research was on [ULPs during election campaigns](https://osf.io/kq6yu)\), and so understood this stuff well. I was also finishing up my PhD at MIT and soon off to Stanford to start as an Assistant Professor, so this seemed like a fun post-dissertation project.

I almost immediately ran into problems. I had forged a good relationship with the AFL-CIO’s research office during graduate school, and had strip-mined their computers for archival data that no one else seemed to still have. I had representation-election data, for example, going back to the mid-1960s. Yet when I pored over their old files, I found that their micro-data on ULP charges only went back to 1990. This was way too late–-all of the action would be in the 1970s and 1980s. Worse, the AFL-CIO’s data was better than the primary source, the National Labor Relations Board. A very friendly lawyer at the NLRB’s San Francisco offices told me that they didn’t keep any of their historical micro-data–-“But you can find any information you need in the tables of our annual reports!”

If you do quantitative research, you’ll recognize how dispiriting that remark is. I could get counts of charges by region, and I could get counts of charges by industry, but I couldn’t find out which industries were in which regions, and so on. I spent a while digitizing information from those annual reports \(for which thanks to [Amanda Sharkey](https://mendoza.nd.edu/mendoza-directory/profile/amanda-sharkey/), whom I met as an RA at this point\), but it was a lost cause. I set the idea aside.

# A new hope
There the story seemed to end. But then we cut to this past November, 2018. I still do *pro bono* data analysis for the AFL-CIO from time to time. This fall, I was one of several people working on briefing papers for their Future of Work Commission. Larry Mishels of the [Economic Policy Institute](https://www.epi.org/) wrote me. For his own briefing paper, he was running some simulations, and he needed values for some of his parameters. What was the win rate in union representation elections in, say, 1987? I’m one of the few people who is conversant with the election data, so Larry and I got to talking.

One of the things Larry needed to know was the size of the work groups involved in each election. In the NLRB’s Case Handling Information Processing System \(CHIPS\), which they used from roughly 1984 to 1999, there is a field in each record for the unit size. It isn’t a real number, though; it’s a categorical variable. This almost certainly corresponded to a coding scheme: 0 for 1-9, 1 for 10-19, 2 for 20-49, and so on. Where there’s a coding scheme, there must \(or at least should!\) be a codebook. But where?

I went online. Some searching directed me to Google Books, which hadn’t really existed back in 2009, and there I found an obscure volume by Gordon Law detailing sources of information about the NLRB. Eventually I was looking at this screenshot:

retro_ulp1.png

Let me highlight a bit:

> Data files were also produced covering elections \(1972-1987\), representation cases closed \(1964-1986\), **and unfair labor practices \(1963-1986\)**, although these are not available from the National Archives and Records Administration. **One copy of these earlier files is held by the Data Archive at the Cornell Institute for Social and Economic Research**.

Reader, my heart skipped a beat when I saw that. At some point in the late 1980s, someone *had* made a data-dump of the NLRB’s ULP micro-data, and it might still exist!

I hied to [CISER](https://ciser.cornell.edu/)’s website, and sure enough, they seemed to have the goods. I downloaded a series of files, unzipped one, and ran head to see what was in them:

retro_ulp2.png

That might look like gibberish. The key thing, though, is that each of the lines that ends in a backslash is *exactly 80 characters long*. Back in the day, a standard IBM punched card held 80 characters' worth of data. \(This is why standard terminals are also 80 characters wide.\) All signs point to these being the original records–and CISER had a code book, too.

Suddenly, nearly a decade after I set this project aside, it seemed like I could actually do it.

# Why I am writing this

If you’ve read this far, congratulations. Why so many words about this?

This project isn’t core to my research program. Heck, I’m knocking on tenure these days, and I have more than enough to keep me busy. I have a new website, though, and it would be nice to have content for a blog. What’s more, there are a lot of ways in which my research habits have ossified. I really should be using R and relational databases these days, but I still fall back on Stata for most tasks. What I need is a real project that I can use to retool, but a project that isn’t so large or complicated that re-learning data management and exploratory data analysis while working on it would be infeasible. Something like, say, modeling the spread of unfair labor practice charges?

I therefore thought I’d do some radically open science: I’d detail this project here on this blog. I do this in part so that I’ll have a record to show students of how research in our field actually happens. And along the way, perhaps I’ll have some fancy graphics to show you.

I do not expect posts to be *that* frequent; this really isn’t my main project. But it’ll be nice to have something to unwind with. So, forward!