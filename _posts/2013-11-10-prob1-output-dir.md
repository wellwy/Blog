---
layout: post_m
title:  "Prob01.Output Directory Error"
date:   2013-11-10 16:45:27
categories: gae
sub : Prob01
summary: solbe"The output directory for the project should be set to /wellwy-demo/war/WEB-INF/classes"
---

## Problem Name
OutPut Directory Error.

## Problem Describe

> Description	Resource	Path	Location	Type
The output directory for the project should be set to /wellwy-demo/war/WEB-INF/

![problem_snap][]
## Reason

This error because of GAE project should put the runtime dependencies (_ **classes**,**resources**,**configs**,etc._) into directory `/war/WEB-INF` ,but maven project's **default** output directory is `/target` , so when build with maven ,it turned out an error.

see defualt package tree:

![default_package_tree][]

##Resolutions

[problem_snap]:{{site.graphs}}/gae/prob/prob01_outputdir_01.jpg
[default_package_tree]:{{site.graphs}}/gae/prob/prob01_default_package.jpg