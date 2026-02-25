+++
date = '2019-06-18T19:54:53-05:00'
draft = false
title = 'Social science and the big O'
+++

...No, not *that* Big O. Behave yourselves.

One of my goals with this blog \(insofar as a blog *has* goals\) is to write about the work behind social-science research, to discuss the process as much as the product of this thing I do. It is useful to see the process, especially if you are newer to the field; but utility aside there is so much that goes into a research project that doesn't show up in the final product. One of the first, hard lessons of graduate school is that most of the work involved in *producing* a study is of interest to no one save the producer. Indeed you can often tell a paper is written by a graduate student because it lavishes space on the details of the matching algorithm or some other "input" step. You can hear the youngling saying, surely this sort of thing isn't meant to disappear forever, is it?

It *isn't* meant to be in the final paper, but that doesn't mean it's meant to disappear forever. Sometimes it would be useful to discuss these things, lest hard-won discoveries never spread. This post for example is about the efficiency of computation and things I learned relating to that, things that will only appear in the final paper insofar as they make that paper possible. But the blessing *and* curse of good research is that often you have to learn a little bit about a lot in order to do the job, because so much of our work is basically bespoke.

With that, a preface: I am not a computer scientist. Take the following as a clever layman's understanding of these issues, and expect me to get some details wrong.

[Big-O notation](https://en.wikipedia.org/wiki/Big_O_notation) is a thing in math and computer science that social scientists almost never encounter. You can read the Wikipedia entry for the details, but a common use of it is to describe how the compute time of an algorithm grows as the input data scales. As such it hooks into [P vs. NP](https://www.youtube.com/watch?v=YX40hbAHx3s) and all sorts of interesting ideas, but it often shows up in the workaday question, "Why the hell is my code running so slow?!"

There are many ways to implement most functions, and those ways can vary enormously in their efficiency. Fifteen years ago, R was notorious for the inefficiency of many of its commands. For lists above a certain size, it was faster to set up a counter in a for-loop and iterate over the elements in the list, incrementing the counter each time, than it was to run `length(list)`! In the worst case, a length function should run in $O(n)$, that is, compute time should be proportional to the size of the object. \(Let's call that "linear time."\) It takes a certain creativity to make that function take *more* than linear time. As a longtime Stata user, I cherished this tale about R, so much so that I've never bothered to check whether it is in fact apocryphal.

I have been reading up on big-O notation because I've been working on a project that involves spatial data. Bracket the research question. I have $(x,y)$ data for about 100,000 establishments all over the United States, each year, for 40 years. For each establishment, I have to calculate a statistic that is based on the properties of nearby establishments, where "nearby" is defined by a radius that I choose. The statistic itself isn't complicated. Nor is it complicated, conceptually, to figure out which establishments are near any focal establishment. But computationally, this can be really inefficient.

In the naïve case, for each point, you must calculate the distance to every other point, flagging those within the radius. If you can't keep track of which comparisons have already been made \(i.e., if you have to calculate the distance from A to B *and* from B to A\), then this involves $n \times (n=1)$, or $n^2-n$, comparisons. Even if you can keep track, that's still $(n^2-n)/2$ comparisons...and as $n$ grows, that quadratic term swamps the $1/2$ constant. Thus this naïve algorithm runs in $O(n^2)$, or quadratic time. Compute time here scales with the square of input size.

This is *muy malo*. From what I've seen on Stack Overflow posts, I'm working with data that's about 10 times larger than most GIS projects. A routine that runs in $O(n^2)$ will thus take about **100** times longer on my data. For the record, that has worked out to more than a day for each year's data.

But here's the thing: "nearby" in my case is about a quarter of a mile! I *know* that virtually all other establishments in the data are too far away to matter. What I needed was a fast way to zoom in on feasible candidates before further testing.

# Adventures in set theory

My first thought was that, given the tightness of my radius, I'd just need to look at others within the same county--I have county recorded in the data. At worst, there might be establishments near the edges of their counties, in which case I'd have to check adjacent counties, too.

It's pretty easy to create an adjacency list for U.S. counties (really!). In python, make a dictionary, where they keys are the county FIPS codes, and the values are lists with the adjacent counties' FIPS codes (include the county itself):

```python
county_adjacent = {
    1001 : [1001,1021,1047,1051,1085,1101],
    1003 : [1003,1025,1053,1097,1099,1129,12033],
    ...
    78020 : [78020,78030],
    78030 : [78020,78030],
}
```

Then you can write something like this:

```python
for county in county_adjacent[county]:
    for establishment in county:
        ** test for distance and do stuff **
```

This greatly reduces the search space but is still pretty inefficient. Consider an establishment in Manhattan. There are six adjacent counties \(Bergen and Hudson in New Jersey, plus the outer boroughs\), and all of them are just lousy with large establishments. At the very least, this process will bog down in large cities.

Next, it occurred to me to first try a spatial test. If I care about establishments within a certain radius, that implies a circle around the focal establishment (that's why I was building circles in the previous post). I could take that circle, as a geometric object, and see if it intersected with the various counties' geometries. Then I'd only have to check establishments in the counties that the circle actually overlapped--in most cases, just one!

```python
for county in county_adjacent[county]:
    if circle.intersects(counties[counties.county == county].geometry.iloc[0]):
        county_ests = ests[ests.county == county]
        in_area = county_ests[county_ests.within(circle)]
        sub_dyads = pandas.concat([sub_dyads, in_area])
```

Two things to notice here. First, this code uses the [geopandas](http://geopandas.org/) package, so the dataframe-query structure follows that of [pandas](https://pandas.pydata.org/), while functions like `intersects()` and `within()` are ultimately being drawn from [shapely](https://pypi.org/project/Shapely/). Second, this code involves three geodataframes: one with the establishments, one with the "circles" around each establishment, and one with the polygon geometries of all the counties. We'll come back to this use of dataframes and the like.

# A surprise appearance of $O(n^2)$

So far, so good. Code of this type took forever to run, though. It would start fine, managing some 3,000 establishments per minute, but it would gradually slow down. Indeed this took *longer* to complete than the original, naïve version--nearly 30 hours for 100,000 establishments! This isn't how programming is supposed to work, right? What was I doing wrong?

The problem lay in the `pandas.concat()` function. This takes two dataframes as inputs and returns their concatenation as an output. It might seem that this would just write the elements to the existing dataframe \(in which case it would run in $O(n)$\), but no: it writes both elements anew. Think about what this means at every step:

- Step 1: write $n_1$ rows
- Step 2: write $n_1 + n_2$ rows
- Step 3: write $n_1 + n_2 + n_3$ rows
- Step N: write $n_1 + n_2 + \ldots{} + n_N$ rows

This amounts to writing data $(n^2−n)/2$ times, which means we're back to $O(n^2)$. No matter how many operations I was saving by my clever spatial sub-setting, I was more than gaining them back with this incredibly inefficient write process! \(Before you ask, `pandas.append()` has the same behavior.\)

The solution to *this* problem, at least, was to create results as a list of lists rather than as a dataframe, and then use `list.append()`, which runs in $O(n)$. This removed the dramatic slowdowns, but by itself it could not surpass the fastest *initial* performance of this type of code, which as mentioned was about 3,000 establishments per minute.

# Detouring through search algorithms

Obviously, if I could figure out some faster way to subset the feasible candidates, I could go faster still. But how? By itself, Euclidean distance is pretty fast to calculate. And indeed that's what the `within()` function was doing. I began Googling phrases like "efficient algorithm to find points within distance" and the like. At some point, this led me to a page comparing linear to binary search.

Say you have a list. You want to find whether an item is in the list--or find its index in the list, which amounts to the same thing. If the list isn't sorted then the best approach is just to start at the first element and compare it to your item. If it matches, you're done! If not, take the next element, and repeat. On average, you'd expect to have to check $n/2$ elements, where $n$ is the length of the list. This means that the compute time for a linear search is proportional to the length of the list--classic $O(n)$ stuff.

Now imagine that the list is sorted. Then the best approach is to start, not at the list's start, but in its *middle*. If the middle element matches your item, you're done! If not, then ask whether it's bigger or smaller than your item. If it's bigger, then you can rule out the entire second half of the list. If it's smaller, then you can rule out the entire *first* half of the list. Take the middle element of the remaining half, and repeat. After each test, you halve the remaining search space, so after $k$ tests there are $n/2^k$ elements left. The most searches we can do before having one item left will therefore be when $n/2^k=1$; solving for $k$ gives $k=\mathrm{log}_2n$. This means that binary search runs in $O(\mathrm{log}(n))$, which is a damned sight faster than linear time!

# From $O(n^2)$ to $O(\mathrm{log}(n))$

This seemed exciting. What if I could sort my establishments by their locations? But that seemed problematic. $(x,y)$ coordinates are by their nature two-dimensional. How would you order them?

Ah, but here's the thing: sorting is itself something that can be implemented very efficiently. So why not just sort by $x$, and then sort by $y$? Granted, I didn't need to find each establishment in the sorted list, but instead all establishments within a certain $x$- or $y$-distance from them. In a sorted list, though, this just amounts to finding the indices of the establishments at $x \pm r$, where $r$ is the radius. Once you have those, you have the subset of establishments that are feasible candidates, at least in the $x$ dimension. If you do the same thing with a list sorted on the $y$ coordinate, then you can get the subset of establishments that are feasible candidates, at least in the $y$ dimension. Then you can find the list intersection \(another efficiently implemented algorithm\), and you're really quite close.

Consider it graphically, using a bit of Boston in 1971 from the last post:

![foo](/bigo_1.webp)

With this process, I could very quickly extract a strip of land half a mile wide with all $x$ candidates and very quickly extract another half-mile strip with all $y$ candidates. Then the list intersection would correspond to establishments within the square, half a mile on each side, centered on the focal establishment. All that I'd need to do then would be to run something like `distance()` on that comparatively tiny number of candidates to check that they were in the circle, not just in the "corners" of the square.

At this point, I started implementing a binary search function, one that could return a slice of the list around the position in question, rather than the position itself. This is extremely simple in comparison. From there you just need the intersection of those two lists of lists:

```python
def intersectionLoL(list_of_listsA, list_of_listsB):
    ''' Returns the list of lists that is the intersection of two
    other lists of lists. The tuple/set/list nonsense is there because
    lists are not themselves hashable objects '''
    return list(list(i)
                for i in list(set(tuple(i) for i in list_of_listsA) &
                              set(tuple(i) for i in list_of_listsB)))
```

From there, you can define a function that checks whether points in that list are within the circle:

```python
def withinCircle(intersect_list):
    ''' Takes a list of lists with (x_coord, y_coord) for points and
    (x_center, y_center) for some reference point. Returns the
    sub-list where the distance between those is below a quarter
    mile. (Yes, I know there's a lot of hard-coded garbage in here.) '''
    new_list = []
    for record in intersect_list:
        if math.sqrt((record[x_coord] - record[x_center])**2 +
                     (record[y_coord] - record[y_center])**2) >
                     .004167:
            continue
        else:
            new_list.append(record)
    return new_list
```

Obviously, there's more code involved than that to read in the data, create the sorted lists, and so on. Yet this gives me a way to find the relevant establishments around every focal establishment, a way that runs blazing fast in comparison to everything else I've tried.

You know I'm going to crow about it: whereas some of these approaches took more than 30 hours per year of data, this takes less than 20 seconds.

# Genuinely, some lessons are learned

It's a cliché in the quantitative social sciences that we don't roll our own code, whether in data management or in statistical estimation. Instead we rely on canned routines. Often you can date the diffusion of a new estimation technique, not from when it is discovered or demonstrated to be better than earlier approaches, but from when it appears as a standard command in a popular stats package. And in general, I think this is fine. Replication is easier when implementations are transparent. Ideally, canned routines are also efficient routines.

But this isn't always the case. A benefit of open-source programs and user-generated code, the ease with which new routines can be distributed, is also sometimes a curse. Inefficient initial implementations make it into codebases all the time.

I want to consider a more subtle problem, though. Frameworks like pandas were developed to make it easier for people used to a statistical-programming environment to wrassle with a general-purpose language like python. Pandas implements the dataframe structure, which feels very comfortable to those of us used to R, Stata, or the like. There's a cost, though. Pandas dataframes leverage [NumPy](https://www.numpy.org/)'s numerical arrays, which means you have interpreted code running interpreted code. And then geopandas is is itself leveraging pandas, as well as shapely, which draws on its own set of libraries...

In most cases, this is all worth it. Most of us will accept longer run time if it takes us less time to think and write the code we run. Implementing routines at a lower level of the "comprehensibility stack" rarely saves time. Usually, the only benefit is that you gain skills that you'll use later on elsewhere, and it's too easy to trot out *that* excuse whenever you want to procrastinate.

Sometimes though you want to bend a package to do something its authors didn't have in mind when they wrote it. In those moments, all of their conveniences become things you'll have to work around, which usually means inefficiency. And if the goal is speed or even just feasible run times, This Is A Problem.

None of this is news to computer scientists. But *social* scientists aren't taught to think in these terms. There are entire classes of models that traditionally weren't often used because their implementations took so long to run (multinomial logistic regression, anyone?). This wasn't too severe a problem--but as our data sets continue to grow in size, the underlying implementations in our workaday software increasingly pose constraints.

The irony is that, thirty or forty years ago, discussion of specific implementations was very common in our literature. When computers were far weaker, it mattered more to get the method of computation right. Why do you think we all got taught OLS--because it is realistic or flexible? No. It's relatively easy to implement by hand, and has an analytical solution. Many early papers in network analysis have appendices discussing the algorithms for calculating the network statistics. The spread of faster computers and standardized software packages has definitely been a net positive, but this type of thinking has fallen out of fashion over the last generation. (I make an exception, of course, for anyone obsessed with Bayesian statistics.) Yet as Moore's Law slows down while data continues to increase, I suspect we'll all have to get back in the habit.