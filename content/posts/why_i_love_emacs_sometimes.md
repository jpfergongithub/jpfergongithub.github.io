+++
date = '2019-10-08T20:40:28-05:00'
draft = false
title = 'Why I love Emacs sometimes'
+++

When I moved from Stanford to McGill, I was startled by how differently the two schools handle their exams. Stanford has a strong honor code; McGill has a strong police state. In Quebec I first heard the \(admittedly awesome\) term "invigilators," the personnel who perch at the front of an examination room and, in extremis, even accompany students to the bathroom during tests.

I mustn't let my dislike for this system hijack this post, so I'll zoom in on one aspect: preparing multiple-choice exams. For classes above a certain size, professors must submit multiple versions of any exams that have a multiple-choice component. The rationale is that, while students are supposed to sit with an empty chair between them, sometimes (*Horrors!*) they have to sit side by side. In such instances, they should have exams that present the questions in a different order, to complicate copying.

Obviously this system incentivizes professors to use simple multiple-choice exams wherever possible, because they're easy to score, easy to recycle, and can be protected from catastrophic cheating by the police-state procedures. Which might feel like pedagogical laziness, but you can always resolve that cognitive dissonance by choosing to belive that your students are devoted to cheating wherever possible...that's a hijacking. Let me start again.

Presume you have a multiple-choice component on your exam. You've written a document that has each question along with five possible answers:

```
1. This is a question
a) This is an answer
b) This is another
c) You get the idea
d) Hoo boy
e) It gets tedious

2. This is another question
a) In case you weren't sure
b) Presume there's five answers

3. This is a third question
a) There actually aren't answers
b) Learning that is part of growing up
```

You must now create multiple versions of this document, each with the questions in a different order, but each sequentially numbered.

This sort of situation, where paranoia mates with technophobia to breed stupidity, drives me up the wall. I once had someone advise me to create a second version by moving the first question to the end and then renumbering everything. Can we agree that that would be ceremonial compliance, which neither solves the problem nor questions the system that poses the problem in the first place? Alternatively, some faculty recommended an application that shuffles such questions and outputs the resulting text files. This special-purpose application of course only exists for Windows machines...and wants you to pay actual money for it.

I haven't figured out how to smash the system yet, but I *have* figured out a solution to the problem. Inevitably, it uses Emacs. I write it up here in part to remember how I did it, but also because it might be worth sharing.

To review some conventions about representing keystrokes in Emacs:

- `C-x` means "hold down `CTRL` and press `x`"
- `M-x` means "hold down `ALT` or `OPTION` and press `x`"
- `SPC` is the spacebar
- `RET` is Enter or Return

I'll presume that you know what regular expressions are, even if you're not super familiar with their syntax. Note that you represent a carriage return in a regular expression with `C-qC-j`; that is, you hold `CTRL` and press `q` then `j`. Note also that, because Emacs LISP uses a ton of parentheses, in Emacs you have to escape parentheses used in regular expressions.

**THUS**: Write the exam as you normally would, numbering each question as "xx":

```
xx. This is a question
a) This is an answer
b) This is another
c) You get the idea
d) Hoo boy
e) It gets tedious

xx. This is another question
a) In case you weren't sure
b) Presume there's five answers

xx. This is a third question
a) There actually aren't answers
b) Learning that is part of growing up
```

First, turn each question into a single line of text. This is done with Emacs's `replace-regexp` command, which is linked to `C-M-%`:

`C-M-% C-qC-j\([abcde]\))C-qC-j? RET ␣\1) RET`

```
xx. This is a question a) This is an answer b) This is another c) You get the idea d) Hoo boy e) It gets tedious
xx. This is another question a) In case you weren't sure b) Presume there's five answers
xx. This is a third question a) There actually aren't answers b) Learning that is part of growing up
```

Second, shuffle the questions, i.e., shuffle the lines. Doing so links up Emacs and shell commands in a way that makes nerds all shivery:

`M-< C-SPC M-> C-u M-| gshuf RET`

```
xx. This is a third question a) There actually aren't answers b) Learning that is part of growing up
xx. This is a question a) This is an answer b) This is another c) You get the idea d) Hoo boy e) It gets tedious
xx. This is another question a) In case you weren't sure b) Presume there's five answers
```

What's going on? First we move the cursor to the start of the file. `C-SPC` sets a mark, i.e., starts highlighting. `M->` moves the cursor to the end of the file, thus highlighting the whole thing. `M-|` will pipe the highlighted region as standard input to a shell command of your choice. Normally the standard output of that command appears in a new window, but prefixing with `C-u` causes the standard output to overwrite the highlighted region instead.

The shell command is `gshuf`, the Mac OS implementation of the Unix command `shuf`. It's available through the `coreutils` package. It reads from standard input, permutes the line order, and writes the result to standard output. The type of command that, once you learn about it, you wonder how you lived without it.

Note that you can save after you run this, then run and save again. Now you have two versions of your file with the lines in a different, random order.

Third, renumber the file. A superb feature of Emacs's regular-expression implementation is that including `\#` in the replacement string will cause the command to replace the matched pattern with an incremental count. This count starts at 0 by default, but it's simple enough to start it at 1:

`C-M-% ^xx RET \,(1+ \#) RET`

```
1. This is a third question a) There actually aren't answers b) Learning that is part of growing up
2. This is a question a) This is an answer b) This is another c) You get the idea d) Hoo boy e) It gets tedious
3. This is another question a) In case you weren't sure b) Presume there's five answers
```

Fourth, turn the lines back into question blocks with another `replace-regexp`:

`C-M-% ␣\([abcde]\)) RET C-qC-j\1 RET`

```
1. This is a third question 
a) There actually aren't answers 
b) Learning that is part of growing up
2. This is a question 
a) This is an answer 
b) This is another 
c) You get the idea 
d) Hoo boy 
e) It gets tedious
3. This is another question 
a) In case you weren't sure 
b) Presume there's five answers
```

Adding a blank line between blocks is left as an exercise for the reader.

# Why I learned Cyrillic

It took me 20 or 30 times as long to write this up as it did for me to *do* it. On the other hand, I first opened Emacs in 2005, so you could say it took me nearly 15 years.

I am not an "Emacs guru." I don't program in Elisp, I don't check my mail or Surf the 'Net in Emacs, and I don't care if you prefer Vi or Sublime or whatever. The thing that all of these pieces of software have in common, though, is a profoundly deep feature set, one that will reward continued engagement. Every few months I learn something new I can do in Emacs--not in the sense of a new mode (I want a niche T-shirt that reads SHUT UP ABOUT ORG MODE) but in the sense of a better way to *edit text*. Editing text is a huge part of what academics do for a living, and I think incremental investment is worth it.

People tell me they don't have the time to learn things like this. I can only say it so many ways, but: I didn't have the time either. That's why I've learned slowly. But each part of the effort was worth it. And certainly if you're in graduate school, and you have your career ahead of you, for god's sake spend some time engaging with software like this. If only so that, in a few years' time, I don't come across someone else trying to randomize their exam questions with an unholy combination of Wordpad and an Excel spreadsheet!