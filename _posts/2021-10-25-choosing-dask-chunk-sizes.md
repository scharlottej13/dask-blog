---
layout: post
title: CZI EOSS Update
author: Genevieve Buckley
tags: [performance]
theme: twitter
---
{% include JB/setup %}

Confused about choosing [a good chunk size](https://docs.dask.org/en/latest/array-best-practices.html#select-a-good-chunk-size) for Dask arrays?

Array chunks can't be too big (we'll run out of memory), or too small (the overhead introduced by Dask becomes overwhelming). So how can we get it right?

It's a two step process:

1. First, start by choosing a chunk size similar to data you know can be processed entirely within memory (i.e. without dask)
2. Then, watch the dask dashboard task stream and worker memory plots, and adjust if needed.

## What are Dask array chunks?

Dask arrays are big structures, made out of many small chunks.
Typically, each small chunk is an individual [numpy array](https://numpy.org/), and they are arranged together to make a much larger Dask array.

<img src="https://raw.githubusercontent.com/dask/dask/ac01ddc9074365e40d888f80f5bcd955ba01e872/docs/source/images/dask-array-black-text.svg" alt="Diagram: Dask array chunks" width="400" height="300" />

## Too small is a problem

If array chunks are too small, it's inefficient. Why is this?

Using Dask introduces some amount of overhead for each task in your computation.
This overhead is the reason the Dask best practices advise you to [avoid too-large graphs](https://docs.dask.org/en/latest/best-practices.html#avoid-very-large-graphs).
This is because if the amount of actual work done by each task is very tiny, then the percentage of overhead time vs useful work time is not good.

It might be hard to understand this intuitively, so here's an analogy.

Let's imagine we're building a house. It's a pretty big job, and if there were only one worker it would take much too long to build.
So we have a team of workers and a site foreman. The site foreman is equivalent to the Dask scheduler: their job is to tell the workers what tasks they need to do.

Say we have a big pile of bricks to build a wall, sitting in the corner of the building site.
If the foreman (the Dask scheduler) tells workers to go and fetch a single brick at a time, then bring each one to where the wall is being built, you can see how this is going to be very slow and inefficient! The workers are spending most of their time moving between the wall and the pile of bricks. Much less time is going towards doing the actual work of mortaring bricks onto the wall.

Instead, we can do this in a smarter way. The foreman (Dask scheduler) can tell the workers to go and bring one full wheelbarrow load of bricks back each time. Now workers are spending much less time moving between the wall and the pile of bricks, and the wall will be finished much quicker.

## Too big is also a problem

If the Dask array chunks are too big, this is also bad. Why?
Chunks that are too large are bad because then you are likely to run out of working memory.

When this happens, Dask will try to spill data to disk instead of crashing.
Spilling data to disk makes things run very slowly, because all the extra read/write operations to disk. Things don't just get a little bit slower, they get a LOT slower, so it's smart to watch out for this.

To avoid data being spilled to disk, watch the **worker memory plot** on the dask dashboard.
Orange bars are a warning you are close to the limit, and gray means data is being spilled to disk - not good!
Take a look at the next section for tips on using the Dask dashboard.

## Use the Dask dashboard

If you're not very familiar with the Dask dashboard, or you just sometimes forget where to find certain dashboard plots (like the worker memory plot), then you'll probably enjoy these quick video tutorials:

- [Intro to the Dask dashboard (18 minute video)](https://www.youtube.com/watch?v=N_GqzcuGLCY)
- [Dask Jupyterlab extension (6 minute video)](https://www.youtube.com/watch?v=EX_voquHdk0)
- [Dask dashboard documentation](http://distributed.dask.org/en/latest/diagnosing-performance.html)

We recommend always having the dashboard up when you're working with Dask.
It's a fantastic way to get a sense of what's working well, or poorly, so you can make adjustments.
## Unmanaged memory

Remember that you don't only need to consider the size of the array chunks in memory, but also the working memory consumed by your analysis functions. Sometimes that is called "unmanaged memory" in Dask.

> "Unmanaged memory is RAM that the Dask scheduler is not directly aware of and which can cause workers to run out of memory and cause computations to hang and crash." -- Guido Imperiale

Here are some tips for handling unmanaged memory:

* [Tackling unmanaged memory with Dask (Coiled blogpost)](https://coiled.io/blog/tackling-unmanaged-memory-with-dask/) by Guido Imperiale
* [Handle Unmanaged Memory in Dask (8 minute video)](https://youtu.be/nwR6iGR0mb0)

## Thanks for reading

We hope this was helpful figuring out how to choose good chunk sizes for Dask. This blogpost was inspired by [this twitter thread](https://twitter.com/DataNerdery/status/1424953376043790341). If you'd like to follow Dask on Twitter, you can do that at https://twitter.com/dask_dev
