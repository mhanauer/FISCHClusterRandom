---
title: "FISCH Cluster Random Parwise Assignment"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Library 
```{r}
library(experiment)
```
Now generate random data
```{r}
dat = data.frame( race = c(.5, .4 ,.3, .2, .1, .15, .34, .78, .9, .2, .2, .3), gender = c(.45, .32, .21, .12, .9, .8, .7,.5, .45, .23, .2, .3), bin = c(rep(1, 6), rep(0,6)))
dat
```
Now try randomize in experiment
Works with unequal designs
```{r}
test_random = randomize(data = dat, group = c("Treat", "Control"))
test_random$treatment

test_random_four = randomize(data = dat, group = c("Treat1", "Treat2", "Treat3", "Control"), block = "bin")

test_random_four$treatment
test_random_four$block

## Didn't work
test_random_match = randomize(data = dat, group = c("Treat", "Control"), match = c("race", "gender"))

test_random_match = randomize(data = dat, group = c("Treat", "Control"), match = "bin")
```
What if we get the mathabolis distance.  Then we use a cluster design to see which things cluster together.  Then we randomly assign within those clusters.  

Man doesn't do well with binary or non-normal variables, which may be fine for our case
```{r}
man_dist =  mahalanobis(dat, center = TRUE, cov(dat))
man_dist
```
Try ranked mah for help with non-normal bin data
gower works well with nominal, binary data and can be plugged into 
```{r}
library(nearfar)
library(cluster)
library(factoextra)
smahal(dat)
gower_dis = daisy(dat, metric = "gower")
gower_dis
anges_dat = agnes(x = gower_dis, diss = TRUE, method = "ward")
hcTree = cutree(anges_dat, k=3)
dat$hcTree = hcTree
cluster_map = fviz_cluster(list(data =dat, cluster = hcTree, repel = TRUE))
```
Ok so now I need to randomly assign with cluster????
This is not working.  So maybe subset each cluster then randomly assign based on that??
```{r}
library(experiment)
library(blockrand)
test_random_three = randomize(data = dat, group = c("Treat1", "Treat2", "Control"), block = "hcTree")

test_blockrand =  blockrand(n = 12, num.levels = 3, stratum = "hcTree")
```
Now try manually
Do this for each cluster
```{r}
library(randomizeR)
dat_clus_1 = subset(dat, hcTree == 1)
dat_clus_1
test_random_three = randomize(data = dat_clus_1, group = c("Treat1", "Treat2", "Control"))
test_crPar = crPar(N = 4, K = 3)
test_crPar
ran_assign_clus1 = genSeq(test_crPar)
ran_assign_clus1
```








