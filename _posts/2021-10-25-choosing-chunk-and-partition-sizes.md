---
layout: post
title: CZI EOSS Update
author: Genevieve Buckley
tags: [performance]
theme: twitter
---
{% include JB/setup %}



[this twitter thread](https://twitter.com/DataNerdery/status/1424953376043790341)

Add - define what chunks / partitions are, and that they are analoguous
Modify text to refer to array chunks OR dataframe partitions
Add - note about shuffles in dataframes being very inefficient

Confused about choosing a good chunk size for Dask arrays?

Array chunks can't be too big (we'll run out of memory), or too small (the overhead introduced by Dask becomes overwhelming).

So how can  we get it right?

It's a two step process:

1. First, start by choosing a chunk size similar to data you know can be processed entirely within memory (i.e. without dask)
2. Then, watch the dask dashboard task stream and worker memory plots, and adjust if needed.

We'll talk more about both those two steps in a minute, but first we'll talk about why having chunks/partitions that are too big or small is bad for performance.

## Too small is a problem

If array chunks are too small, it's inefficient. Why is this?

Using Dask introduces some amount of overhead for each task in your computation.
If the amount of actual work done by each task is very tiny, then the percentage of overhead time vs useful work time is not good.

It might be hard to understand, so here's an analogy.

Let's imagine we're building a house. It's a pretty big job, and if there were only one worker it would take much too long to build.
So instead, we have a team of workers and a site foreman. The site foreman is equivalent to our Dask scheduler, and their job is to tell the workers what tasks they need to do right now.


Say we have a big pile of bricks to build a wall. If the foreman (the Dask scheduler) tells workers to fetch a single brick at a time, it will be slow. But if they fetch a wheelbarrow of bricks each time, it will be finished quicker.

## Too big

And if the Dask array chunks are too big, you'll run out of RAM.

Dask will try to spill data to disk instead of crashing. This makes the computation inefficient, because of all the extra read/write operations to disk (those are slow!)

...


To avoid this, watch the worker memory plot on the dask dashboard. Orange is a warning you are close to the limit, and gray means data is being spilled to disk (bad, try and avoid this).

[Dask dashboard (18 minute video)](https://www.youtube.com/watch?v=N_GqzcuGLCY)
[Dask Jupyterlab extension (6 minute video)](https://www.youtube.com/watch?v=EX_voquHdk0)
[Dask dashboard documentation](http://distributed.dask.org/en/latest/diagnosing-performance.html)

...


Remember that you don't only need to consider the size of the array chunks in memory, but also the working memory consumed by your analysis functions. Sometimes that is called "unmanaged memory" in Dask,  There are tips for that

* https://coiled.io/blog/tackling-unmanaged-memory-with-dask/
* https://youtu.be/nwR6iGR0mb0

...

(This explanation was helpful for people at our tutorial yesterday, so I thought I should share it here too)

