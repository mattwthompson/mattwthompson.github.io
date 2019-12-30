---
title: 'Using progressbar in Python scripts'
date: 2019-12-30
permalink: /posts/2019/12/progressbar/
tags:
  - python
---

I often find myself running a script, usually analysis of some molecular dynamics trajectory, without knowing how long it will take. This typically for any of the following reasons:

* I need to finely sample a trajectory, i.e. calling some function on thousands of frames
* I'm running a novel analysis and have not developed an intuition about how long it should take
  (how well it should scale, if it is likely to encounter a memory bottleneck that makes it hard to
  prototype on a MacBook, etc.)
* Like above, maybe I just wrote it - and I try to ahere to the philosophy of "get it to work, then
  get it to work well" (I may write about this another time, but for now I will just say [this
  slide](https://twitter.com/FRoscheck/status/1159158552298229763)
  from a talk by Stan Seibert at SciPy 2019 was formative for me) which means I've probably used
  NumPy functions but left some other optimizations on the table.

So I often find myself running `python my_cool_function.py` and .... waiting. If it runs after a few
seconds, nothing to worry about. But if it takes a minute or two, then I can either hope it takes
only a few minutes longer or, as happens often, wait something like 15 minutes. Not great practice.
Until recently, my approach was to set some counter in the middle of the expensive part of the
script and just print stuff out:

```
i = 0
num_iters = len(thing_im_iterating_over)

for var in thing_im_iterating_over:
    # Do some computations
    print('Done {} i out of {}'.format(i, num_iters))
```

This works well enough for diagnosing how long my script may take - it's easy enough to see if
it'll take the order of minutes or hours, in which case something else in the workflow needs to
change - but isn't very pretty. There's nothing wrong with using print statement for debugging and
toying around, but I find myself doing this often enough I wanted to look for a prettier solution.
I've seen some programs (i.e. [signac](https://signac.io) lately, or downloading packages from
various package managers) use progressbars to inform the user of the status of an operation,
particularly long ones. Unsurprisingly, it's fairly straightforward to do in Python, and here I am
sharing a couple ways I have used it. I used the [package](https://pypi.org/project/progressbar/) of
the same name, although I'm sure there are other fine options out there.

The first simple example is structured like above, where we have a clear iterator (or generator, of
course) we're iterating over and we know its length. 

```
import progressbar


bar = progressbar.ProgressBar()

for i in bar(thing_im_iterating_over):
    # Do some computations
```

This will print out the progress bar to the terminal and it will update as the loop is executed.

A slight variation of this I found useful was the case in which we don't necessarily want to update
each iteration of a loop, but only want to update when some criteria in the loop was met.

```
import progressbar


pbar = progressbar.ProgressBar().start()

ts = 0
num_ts = 1500000

with open('huge_file.txt', 'r') as fi:
    for line in fi:
        line = line.split()
        if line[0] == 'TAG_I_CARE_ABOUT':
            # Do some some computations
            if ts % 1000 == 0:
                pbar.update(ts / num_ts * 100)
            ts += 1
```

What I don't like about this approach is that I'm carrying an extra counter variable that I probably
don't need. That aside, this has the benefit of  only updating the progress bar every few thousand
iterations of the outer `for` loop. In this case, I was given a 1170315604 line text file and needed
to parse a particular comment line every few thousand lines, but even of those I only needed to
track every few thousand hits.
