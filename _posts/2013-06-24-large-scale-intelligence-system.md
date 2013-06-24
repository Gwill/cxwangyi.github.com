---
layout: post
title: "Large Scale Intelligence System"
description: ""
category:
tags: []
---
{% include JB/setup %}

I am planning to write a book about my exploration in the field of
large-scale data mining during the recent five years.  I am
considering the follow topics:

  1. Frequent item-set mining

     This section is basing on my ACM RecSys 2008 paper *PFP: Parallel
     FP-growth for query recommendation*.  A well-known solution to
     frequent itemset mining is FP-growth, which relies on
     representing large data by a compact in-memory data structure
     known as FP-tree.  However, constraint by the size of memory,
     this idea does not scale well.  In my paper, we proposed to
     abandoning the idea of FP-tree; instead, we represent data and
     intermediate results literally in GFS, and process them brutely
     using MapReduce.  The sophisticated parallel computing
     infrustructure enabled us to mine long-tail patterns out of human
     data-set, an achivement that was never reached by previous
     methods.

  1. Graph clustering

     This section is basing on my KDD 2009 paper *Parallel community
     detection on large networks with propinquity dynamics*, where we
     solve the problem of mining quasi-cliques in a large graph using
     a sophisticated distributed computing framework, Pregel.  Our
     solution can process graphs that is two magnitudes larger than
     those reported in literature.

  1. Collaborative filtering

     This section is basing on my WWW 2009 paper, *Collaborative
     Filtering for Orkut Communities: Discovery of User Latent
     Behavior*, where I propose to solving the well-known
     collaborative filtering problem using MapReduce.


  1. Latent Dirichlet Allocation

     This section is basing on my AAIM 2009 paper, *PLDA: Parallel
     Latent Dirichlet Allocation for Large-scale Applications*, and
     later exploration on learning 100,000 latent topics from billions
     of training data.

Any suggestion would be highly appreciated!

