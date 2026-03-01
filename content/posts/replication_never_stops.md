+++
date = '2025-01-08T13:07:46-05:00'
draft = false
title = 'Replication never stops'
+++

Every so often I get email from people who are working with union-represenation data from the National Labor Relations Board. \(No surprise here: [I host files and code for the CHIPS and CATS databases](https://www.jpferguson.net/nlrb-representation-case-data).\) Upon getting one recent such message, I decided to broaden the conversation, and copied *everyone* who had written me asking about such data in recent years. I think it has been useful for people to see what resources others have developed.

As part of that conversation, [Tom Baz](https://www.linkedin.com/in/tom-baz/) wrote that "I want to raise some more general questions. How do you approach connecting related R Cases and C Cases that happen in the same establishment?" To translate: the NLRB's union-election cases, or representation cases, are the "R Cases," while the charges of unfair labor practices, or complaints, are the "C Cases."

I told Tom that he had triggered my PTSD. In my first-ever sole-authored paper, I linked these two datasets together. The challenge is that the NLRB's fundamental unit in its own database is the case, which has nothing to do with more sensible units like the establishment. The NLRB doesn't record consistent employer IDs, and for many years chose not to release the address information included with these filings. Any researcher trying to link these data would be starting from scratch.

That paper was part of my dissertation. In said dissertation I wrote a detailed appendix about how I matched the records. I did this entirely because, at the time, I was getting pushback from other researchers about how I had identified ULP charges that accompanied specific union election drives. I pulled this appendix out of the thesis and shared a copy of it, which I'll also [share here](/Ferguson_2009thesis_AppendixA.pdf).

I'm not sharing this because I think there is a large untapped demand for it. I share it because, while it's existed since 2009, it exists as an appendix in a dissertation in the MIT Library system. The odds that someone would ever find it by searching the open web are extremely small. This is exactly the sort of work that gets lost in what should theoretically be a shared body of scientific progress.

I'm also sharing it because, dang, if you'd told me that the appendix that I wrote in my dissertation nearly sixteen years ago would *ever* be helpful to anyone, I would've laughed at you. Yet here I am, exposing how I documented my work. In a perfect world I would just direct the reader to a full replication package, but when I first started doing this project (in the summer of 2004!) those weren't much spoken of. Now I wonder: *can* I rebuild that project's results from scratch?

Sometimes, I get the impression that people think we only build replication packages because we want to guard against scientific fraud. And this leads to some of my colleagues complaining, stupidly, about the "replication police" or some such. But it has to be said, time and again, that you should take the time to document your work in reproducible form *because it helps the research process*. There is no better way to ensure that research will be built upon than to make sure that people can see how it was built in the first place.

UPDATE (24 January): Sixteen days after I wrote this post, I got *another* email asking me about how I matched RC and CA cases in that paper, so many years ago. I was delighted to direct them to this post! Rarely is a good turn so quickly repaid.