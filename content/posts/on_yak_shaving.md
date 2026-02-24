+++
date = '2019-01-27T13:43:54-05:00'
draft = false
title = 'On yak shaving'
+++

It is hard to do quantitative social science without picking up some of the habits of computer science. Most of these are technical, but sometimes you imbibe a bit of the culture as well. One cultural artifact that I picked up while a graduate student at MIT was the concept of “yak shaving.” I’ll quote a definition:

> You have probably noticed how the tiniest Java programming project, or home improvement project often grows seemingly without bound. You start to do something very simple. You need a tool to do that. You notice the tool needs to be tuned a bit to do the job properly. So you set out to sharpen the tool. To do that, you need to read some documentation. In the process, you notice the documentation is obsolete. So you set out to update the documentation... This chain goes on and on. Eventually you can unwind your to-do stack.

Ultimately, there’s a *Ren and Stimpy* episode whence the term comes. But this, reader, is yak shaving: doing the thing you need to do to do thing you need to do to do the thing.

If a project needs any new software, then it will involve yak shaving. The sign of an experienced social scientist is not that they do less yak shaving. It is that they learn to budget time for yak shaving. You cannot know the length of all yaks' hair in advance, but you know there lurks a shaggy bastard among them.

I have decided to analyze my Unfair Labor Practice data \(see the previous post\) using R, because R is what The Kids Today use. I’ve used some version of Stata since 2000, and I’m both good and fast in it. Yet I can feel the gap opening up between Stata’s metaphor of doing quantitative social science and the rest of the world. Every time I have to work with many small \(or large!\) data files, I feel the constraints of Stata’s “one data set in memory” architecture binding me. I also like to work with graduate students and younger faculty, and these days, no one learns Stata in their methods classes. I’ve always rolled my eyes at the senior faculty who force their students to learn SAS or \(*shudder*\) SPSS because they can’t be bothered to adapt, and I don’t want to join their ranks. So, R.

Thus, new software. Thus, yak shaving.

Yesterday I sat down planning to read the fixed-width text files I downloaded from CISER into some sort of database, then clean it up in R. I quickly realized that there just wasn’t enough data here to worry much about a database as such. Uncompressed, this dataset comprises two files, each about 35 megabytes. It would be simpler just to read things directly into R.

# The first digression

To this point, I’d saved the archives from CISER in my Downloads folder. This wouldn’t work going forward; any project requires more organization so you can remember how the pieces fit together when you go away and come back to it.

In principle, you can create a folder named “Project” and save *everything* inside it. I did this for my first couple projects. It sort of worked, but only because I have a good memory. As soon as I started working with co-authors, I discovered all the ways this was insufficient. I therefore started making project directories that have a structure like this one:

yakshave1.png

Here, “canonical” holds primary data. Is it a file that you cannot re-create using *other* files in the project? Then it goes in canonical. By contrast, any data files that I create go in “data,” and the code that generates them goes in “scripts.” I put any codebooks, manuals, or the like in “documentation,” and any articles or book chapters that inform the substantive research question \(as distinct from the data-creation and -analysis processes\) in “literature.” The output of the project winds up in “drafts,” “figures,” “tables,” and “presentations,” in ways that are probably self-explanatory.

This whole directory lives on Dropbox. Here, the data are public, and so there’s no privacy concern with posting things this way. My experience over the last eight years has been that Dropbox is the easiest way for me to share files with co-authors and keep everything synced. Before this, I used Subversion, but found that some co-authors never really cottoned to version control. For similar reasons, I’ve been slow to move things into Git repositories. Maybe some day. The key point is that, this way, the entire project lives somewhere that is shared, automatically synced, and protected with version control.

Thus the very first bit of yak shaving: organize the original files, such that the text files are in canonical and the codebooks are in documentation.

# Reading time

Reading fixed-width data into R is easy enough; you can use `read.fwf()` to do it. That command requires you to specify the *widths* of each element, that is, their start and end positions in the original 80-character record. This is where the codebook becomes important:

yakshave2.png

The codebook tells me, for example, that element 15 \(the count of charges in the situation\) is 3 characters wide \(fields 41-43\), that elements 16 is one character wide, and so on. I thus tried doing just that, and ran into problems:

yakshave3.png

The `read.fwf()` command is expecting simple ASCII or UTF-8 unicode in the text file. Fancy characters and multibyte strings are a problem because they violate the one-character, one-field mapping that a fixed-width file is supposed to respect. You see this sometimes: somewhere along the way, there was a mistake such that characters were written wrong at the bit level. This isn’t something that `read.fwf()` knows how to handle, and it dies. The solution is to inspect the file for non-ASCII characters, try to strip them out and replace them with whatever is supposed to be there, and move on.

The standard tool for this is `grep`, which if you haven’t used it is taken to mean “Get Regular Expression and Print.” \(If you’re doing quantitative social science and haven’t heard of `grep`, it’s time to stop and re-examine your priorities.\) I don’t do this often, but I know that there’s a pretty standard way to search for non-ASCII characters. Some Googling pulls up the right pattern; I try it, and no dice:

yakshave4.png

It seems that the version of `grep` that comes standard on the Mac doesn’t recognize the `-P` flag! Some more Googling reveals that, as of Mountain Lion, OSX uses BSD `grep` rather than GNU `grep`, and the former doesn’t support “Perl compatible regular expressions,” which is what I specified. All is not lost, though, because there’s a package I can install with the remarkable [Homebrew](https://brew.sh/) \(If you use a Mac and do not use Homebrew, again with the priorities\) called “`pcre`” and which has *inter alia* a command called `pcregrep` which, well, you can guess from the name.

So, I type `brew install pcre`. No dice again:

yakshave5.png

Xcode is out of date! If you do any sort of programming or even package management on a Mac, you know what Xcode is: the suite of developer tools that for some reason *aren’t* installed by default. I have them on this machine, but I haven’t updated them recently, and Homebrew has determined that some of pcre’s dependencies require a newer version. So, I start updating Xcode...which usually takes a little while.

# Down the rabbit hole

This is a good time to recap. Right now,

1. I am updating Xcode...
2. ...so I can install the pcre package...
3. ...so I can find non-ASCII characters in `unf.labproc.c6376`...
4. ...so I can bring the file into R with `read.fwf()`.

This is classic yak shaving. Frankly, I’m a little nervous that I’m going to have to update my operating system next...but no, Xcode eventually updates, and I start climbing back up the chain. Eventually I can run `pcregrep` and get real results:

yakshave6.png

This is telling us that, on the 257,194th line of this file \(which has about 440,000 lines\), There is a non-ASCII character just before the characters “806.” This is telling, because if you go back to the original error message, it said that there was an `invalid multibyte string at '<ba>806<39> \'`. It looks like this might be our culprit. Furthermore, when I open the file in Emacs and go to line 257194, I see a superscript “o” in that position. My best guess \(trust me?\) is that this is supposed to be a zero.

I change it, save the file, and run `pcregrep again`. This time it exits without reporting any hits. It seems we’ve found the source of the hiccups...

...until the next error. This one turns out to involve a lot of Googling, but little actual yak shaving. The file includes \#s, which are a comment character in R. You have to tell `read.fwf()` that the comment character is something else--I chose *nothing*--so it will ignore these. With that change, I can read in the file at last!

# And so it goes

As it turned out, I wasn’t done shaving yaks for the day. A lot of the data are stored as factor variables. I wanted to use the `recode()` command to clean these up. That command is in R’s “car” package, and to install that I had to update my version of R. Then, when I tried to install the all-important “tidyverse” package, I found that a chain of dependencies did not resolve because, ultimately, the “xml2” package relied upon the libxml-2.0 library, which isn’t on the Mac by default...

You get the idea. I worked on all of this for about five hours, off and on, yesterday. By the time I went to bed, I’d read in the first of the two ULP files and cleaned up 4 of its 40 variables. Some of the intervening time was spent learning the ins and outs of working with factor variables, date variables, and string functions in R, but the majority of the time was spent yak shaving.

I do not take this as a sign of incompetence. As I said at the start, experience doesn’t mean you do less yak shaving, it means you budget time for this. I know that the first several days on this project will have this feel. Could I do all of this in Stata, and already be done with this stage? Absolutely, but then I wouldn’t have learned anything about R, and I’d still have to do all of this if I were to try it in the future.

This is a trade-off in research that we all become familiar with. When does it make sense to invest the time to learn new tools? When is the productivity lost now outweighed by gains in the future? Many people, myself included, have to be careful about this, because “tooling up” can become a form of procrastination. The worst vices are the ones that can look like virtues. Here, I’ve specifically decided that this project will happen in my spare time, and that learning new systems is as big a part of it as the final result, so I don’t mind how that spare time is spent. But striking the right balance between tooling up and getting things done is one of the most important things one must do in a self-directed career like academia.