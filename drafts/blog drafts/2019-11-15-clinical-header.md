---
title: 'Orchestrating Batch Submissions with a Parallel Workflow'
date: 2019-11-15 00:00:00
description: Running SAS code in parallel
featured_image: '/images/blog/IcelandGlacier.jpeg'
tags: clinical-trials workflow types-of-computing parallelization
---

![](/images/blog/IcelandGlacier.jpeg)

## Scaling Workflow
We all like short efficient code.  One of our favorite tricks to to separate our code into multiple files and then read them into a "main" file.  In SAS the was this is accomplished is with the ```sas %include``` statement.  

In SAS, a common way of building a project is to create the main.sas type file in a folder containing multiple subparts of the code.  In my days of writing code for clinical trials we did this for every trial. The idea was to run one single program in batch so all the code, logs, listings, output, datafiles would have related time stamps.  

example

### Introducing Parallel Workflows in SAS with SAS/Connect

### A simple alteration to scale our project

show edits to the code above that accomplish parallelizm

### talk about logs
all in one, or one for each files

### does embed work for examples?

{% gist e2cdf32dbf012009ba238793ff16bfbe list_to_rows.sas %}
