+++
date = '2019-05-08T19:45:21-05:00'
draft = false
title = 'Will it work?'
+++

Every research project has one or two assumptions that, if violated, torpedo it. These assumptions are the scariest things to test, and they can keep me from working. They often turn out to hold, which means that I procrastinated for no reason, but I can do little with that knowledge. This is how the anxious brain works.

This week I'm working on a project to measure spatially-weighted racial employment segregation. The core idea is simple. We've found that racial segregation between establishments \(workplaces\) has grown over the last generation, yet many of us would say that our work environments have grown more racially diverse. Can we square that circle? One possibility is that we *do* work in more diverse environments, but many of the people we encounter actually work for other employers. Think about outsourcing. I may see a cafeteria worker, janitor, or landscaper of a different race every day, but they needn't be a recorded employee of *my* employer. We interact, but on paper we are in different firms.

\(This is all the more obvious now, having moved from the Bay Area to Montréal. Here, those positions are part of the same internal labor market. And lo, while their pay is below a professor's, they have many non-wage benefits similar to other university employees'. The [tale of two janitors](https://www.nytimes.com/2017/09/03/upshot/to-understand-rising-inequality-consider-the-janitors-at-two-top-companies-then-and-now.html) continues.\)

I could get at this idea by using a spatially-weighted segregation measure, one that takes into account both which firms people work for and where those worklpaces are located relative to each other. A few papers lay out the mathematical machinery to do this, but almost none tie the pieces together. Doing so seemed like a fun idea and an important contribution. Hence this project.

I have more than 40 years of data from the Equal Employment Opportunity Commission, showing the racial composition of establishment workforces. I have geocoded that data, such that I have longitude and latitude for about 85 percent of them. In other words, I have maybe the ideal data with which to do this study. So why am I so anxious?

Partly it's banal. I have to figure out to implement a lot of the necessary calculations. I wrote up a memo explaining how to do it, which is how I learn it myself; but even if I understand how to do it in the abstract, I have to implement it in software. This means getting up to speed on a bunch of geospatial Python packages. At least nowadays there's `geopandas`, which leverages `pandas` to provide something like a sane data structure, but I don't know `pandas` that well...you get the idea. It's yak shaving time again. But this isn't the real issue.

I have data for every *large* establishment. These are workplaces with at least 100 employees. Many contracted, external, or other workplaces might have fewer employees and thus not appear in these data. Instead I can look at the spatial proximity of large workplaces to each other, and get a sense from considering their spatial relations what the full population effect might look like. This means that this study will probably understate the effect of considering space \(if there is one\). By itself, this is not a fatal flaw. You'd rather have research designs that stack the deck *against* your finding an effect.

The root problem is this: what if large establishments are far enough apart that you can't get *any* meaningful overlap between them, unless you adopt an unrealistically large "reasonable" distance? I want to use something like a five-minute walk, or about a quarter of a mile. That will keep you on many corporate campuses, and keep you within a radius of one or two city blocks. Is that sufficient?

I couldn't know in advance. And this is what has been eating at me.

This is "research week" at Desautels. Those of us faculty who sign up are released from service work for the week \(*siiiiiiiiiiiiick*\) and given the space to make a big push on a particular project. I decided to work on this one. Partly because I need a week of concentration to learn some of the software and write the code, but mostly because daily meetings with my group would give me a commitment mechanism to see this through. It's always useful to find tricks that keep you working!

# So what did I find?

Monday was spent building the dataset: merging the geocoded addresses (which we assigned dummy IDs to preserve confidentiality) back into the main data, and smaller, annual datasets that would be fast to analyze. Tuesday was spent turning those CSVs into ESRI shapefiles, installing a shapefile viewer and other relevant Python packages, and generally prepping things for the real calculations. Which meant that at some point yesterday, I could open [QGIS](https://www.qgis.org/en/site/) and see this:

![foo](/dots1.webp)

That's every large workplace for which I have geospatial information in 1971. I'm showing 1971 for two reasons. First, if things look OK nearly fifty years ago, when there's less development, they'll look OK today. Second, even though it would be incredibly hard from these dots alone to identify specific employers who filed an EEO-1 survey in that year \(and in any case are legally required to do so\), I have to keep employer identities confidential. Using data from 48 years ago makes it almost impossible that even the most brilliant sleuth would back out a current workplace from these plots.

The action comes when you zoom in. Here's metro Boston:

![foo](/dots2.webp)

*A lot of the circles overlap!* Like, a *lot* of them. This strongly suggests that these large workplaces *are* close enough to one another to give some leverage to a spatial measure.

Similarly, here's San Francisco--again, 48 years ago:

![foo](/dots3.webp)

You can easily see downtown and the financial district in the city's northeast. Those familiar with SF can also probably eyeball a few major streets, like Geary and 19th Avenue, from clusters of firms along them.

So far, so good. But these are big, dense cities. Is the overlap only meaningful there? I don't need the rural areas to be that dense--fewer people live and work there, so by definition they're less important for aggregate measures--but what about the suburbs?

Here's Carrollton, Texas:

![foo](/dots4.webp)

Carrollton is a first-ring suburb of Dallas. It's next to Lewisville, where I went to junior-high and high school \(Go Farmers, etc.\). More importantly, in 1971 it was what we'd call a shitheel burg: A suburbanizing farm town that had had large employers set up shop along the I-35 corridor. Back then, Carrollton had a bit more than 13,000 people; Lewisville had fewer than 10,000. Both of these cities have more than 100,000 residents today. If I can get some relevant spatial overlap in these places that long ago, I feel pretty good about the data overall, and thus am more relaxed about the project's potential.

\(I can already hear people asking: If Carrollton was that podunk back then, why were there so many large employers \(relative to size\) there? Just to the southwest of the area clipped in that image is DFW Airport, well on its way to becoming one of the nation's busiest. The companies locating to Carrollton included firms like [Halliburton](https://en.wikipedia.org/wiki/Halliburton), which counted aircraft supply among its operations, and where a great-uncle of mine would later work. He was a Teamster and machinist, smoked 2-3 packs of coffin nails a day, and surprised no one by dying of lung cancer. I still have a box of drill-press bits he gave me, more than 25 years ago. I digress.\)

# So what's the point?

This exercise falls under the broad topic of exploratory data analysis. When I was in graduate school, I sometimes got the sense that exploratory data analysis was what you did to *come up with* a research question, which today strikes me as really shady. A better way to think about exploratory analysis is getting to know your data. The scary assumptions I've been talking about are also sanity checks. It makes no sense to proceed if this assumption I've been talking about doesn't hold. No matter how elegant the math, if there isn't enough spatial overlap to investigate the issue, we're done.

It's tempting to ignore these sanity checks as long as possible. But that's perverse. Imagine you get no results, and eventually discover that one such assumption didn't hold. That's even more time you've wasted. Worse, imagine that you *never thought to check this*. Many are the studies I've seen with odd or nonsensical results where I think the researcher never spent time on basic investigation of their dataset.

But I don't want to focus too much on the negative. There's also a strong positive reason to do this work. Having seen that the data pass this smell test, I'm way more confident *and excited* about the project. That burst of relief and self-confidence will propel me through another few days' fumbling with code. Forward!
