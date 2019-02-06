---
title: "FISCH Cluster Random Parwise Assignment"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
One problem researchers face when wanting to randomize clusters (e.g. schools, clinics) to two or more interventions is the fact that randomization does not always produce equivalent groups.  Techniques such as matched and blocking randomization which can match clusters on characteristics prior to randomization.

In this example, I provide a framework for how to use cluster analysis that allows for the inclusion of several different kinds of variables (e.g. continuous, ordinal, binary).     

First, we need to library the packages that we will use in this example.
```{r}
library(randomizeR)
library(cluster)
library(factoextra)
```
For this example, let us assume we a have smoking cessation program that we want to test in a large community mental health and behavior setting called Star Providers.  Star Providers is located in several states with many clinics inside those states.

Let us assume we have two treatment and one control conditions.  One approach we could take is to randomly assign the clinics, instead of individuals to the different interventions.

However, as we stated before it could be the case that the randomization does not work as we would hope and certain clinics with similar variables (e.g. race, gender) could get lumped together decreasing our study's validity.

Therefore, we could match them on several clinic level characteristics.  Let us assume that we have 12 clinics and we have aggregate data (i.e. percentages) for gender and race and we also have a binary indicator that indicates whether a clinic has a no smoking policy.
```{r}
dat = data.frame( race = c(.5, .4 ,.3, .2, .1, .15, .34, .78, .9, .2, .2, .3), gender = c(.45, .32, .21, .12, .9, .8, .7,.5, .45, .23, .2, .3), no_smoking = c(rep(1, 6), rep(0,6)))
dat
```
With a stratification design, using more than a few variables, particularly when we have a limited number of, clinics in this case, deepens the problem of dimensionality: https://en.wikipedia.org/wiki/Curse_of_dimensionality

Therefore, one approach is to create a distance matrix that can be used to cluster clinics that are similar.  However, many approaches such as Euclidean measure require the data be standardized before distance is calculated to put all variables on the same scale.  Therefore, researchers are limited in the types of variables that they can include since only continuous variables can be standardized.  

However, the Gower coefficient can be used to calculate the distance between nominal, binary, and continuous variables: https://www.rdocumentation.org/packages/cluster/versions/2.0.7-1/topics/daisy
```{r}
gower_dis = daisy(dat, metric = "gower")
gower_dis
```
With a distance matrix, we can then use standard cluster analysis techniques to identify which clinics are the most similar.  I refer the readers to another one of my blog posts for more information about cluster analysis: http://rpubs.com/mhanauer/461566
```{r}
anges_dat = agnes(x = gower_dis, diss = TRUE, method = "ward")
hcTree = cutree(anges_dat, k=3)
dat$hcTree = hcTree
cluster_map = fviz_cluster(list(data =dat, cluster = hcTree, repel = TRUE))
cluster_map
```
Once you know which clinics are in which clusters you can then use the crPar function in the randomizeR package, to randomize the clinics to the three interventions (treatment 1 = A, treatment 2 = B, and control = C).  

For example, cluster one has four clinics in it and we want to randomly assign them to the three intervention options.

Now we have a sequence of randomly assigned interventions within cluster one.  We could just assign these interventions in numerical order so... 

Clinic 1: A (treatment 1)
Clinic 2: B (treatment 2)
Clinic 3: C (control)
Clinic 4: B (treatment 2)
```{r}
set.seed(12345)
cluster1_crPar = crPar(N = 4, K = 3)
cluster1_crPar
ran_assign_clus1 = genSeq(cluster1_crPar)
ran_assign_clus1
```
Limitations:
With a small number of clusters as is the case here, it could be that a cluster does not contain enough, clinics in this case, to randomly assign at least one clinic to each of the interventions; however, this is a problem that is present with blocking random assignment as well.
