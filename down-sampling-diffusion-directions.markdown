---
title: Down-sampling diffusion encoding directions
layout: post
---

Diffusion MRI data are typically acquired with a series of diffusion encoding directions, corresponding to one or more b values, for the targeted analyses (e.g., diffusion tensor imaging,  DSI, HARDI and q-ball reconstruction among others). For example, MRI protocols with 30 diffusion-encoding directions at $$b=1000 \frac{s}{mm^2}$$ have been used in many diffusion-tensor imaging projects, and protocols with a higher angular-resolution (i.e., more diffusion-encoding directions) at multiple b values are generally needed for high-order reconstruction (e.g., DSI).   

For research projects that analyze data obtained with different protocols, one might need to down-sample higher angular-resolution diffusion MRI data (with more diffusion directions) to lower angular-resolution (with few directions), in a way that the down-sampled encoding schemes are similar across data sets.

In this [page]() we show a matlab script to down-sample a high angular-resolution `bvec` scheme defined by ['more_directions.txt']() to 30 directions, which resemble the `bvec` scheme defined in ['fewer_directions.txt'](). Basically, for each of the diffusion-encoding `bvec` directions defined in ['fewer_directions.txt'](), we use matlab `dot()` function (inner product) to identify the most similar one among all diffusion-encoding directions defined in ['more_directions.txt'](). Note that two diffusion encoding directions are considered the same, when the inner product is either 1 or -1 (assuming that diffusion encoding schemes `[1,0,0]` and `[-1,0,0]` have the same effect on diffusion signals).

The provided [matlab code]() has a few limitations. First, it is implemented with an assumption that the two `bvec` files have the same b value. For a more complicated scheme (e.g., only 1 b-value is used in low angular-resolution scan and multiple b values are used in high angular-resolution scan), then some additional manual processing (e.g., extracting diffusion-encoding directions with the same b value) would be needed. Second, this matlab code does not examine whether the down-sampled diffusion directions are equally distributed in 3D space.

