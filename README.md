---
title: "FISCH Cluster Random Parwise Assignment"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
One problem researchers face when wanting to random either indivudals or clusters to two or more interventions is the fact that randomization does not always.  Techniques such as matched designs and stratifed which can match indiviudals / clusters on charactersitcs prior to randomization.

In this example, I provide a framework for how to use cluster analysis that allows for the inclusion of several different kinds of variables (e.g. continious, oridnal, binary).     

First, we need to library the packages that we will use in this example
```{r}
library(experiment)
library(cluster)
library(factoextra)
```
Now generate random data
```{r}
dat = data.frame( race = c(.5, .4 ,.3, .2, .1, .15, .34, .78, .9, .2, .2, .3), gender = c(.45, .32, .21, .12, .9, .8, .7,.5, .45, .23, .2, .3), bin = c(rep(1, 6), rep(0,6)))
dat
```

Try ranked mah for help with non-normal bin data
gower works well with nominal, binary data and can be plugged into 
```{r}

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



