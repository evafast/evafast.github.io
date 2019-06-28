---
layout: post
title: "Clustering of single cell data - a practical guide"
date: "2019-06-28"
slug: "example_content"
description: "A practical guide to clustering single cell RNAseq data"
category:
  - single cell RNAseq
  - genomics
# tags will also be used as html meta keywords.
tags:
  - examples
  - common_tag
show_meta: true
comments: true
mathjax: true
gistembed: true
published: true
noindex: false
nofollow: false
# hide QR code, permalink block while printing.
hide_printmsg: false
# show post summary or full post in RSS feed.
summaryfeed: false
## for twitter summary card with squared image and page description or page excerpt:
# imagesummary: foo.png
## for twitter card with large image:
# imagefeature: "http://img.youtube.com/vi/VEIrQUXm_hY/0.jpg"
## for twitter video card: (active for this page)
#videofeature: "https://www.youtube.com/embed/iG9CE55wbtY"
#imagefeature: "http://img.youtube.com/vi/iG9CE55wbtY/0.jpg"
#videocredit: tedtalks
---

Whether you've analyzed single cell RNAseq experiments yourself your you have only been exposed to them via publications chances are high that you have encountered data that is sub grouped into clusters. Even before the advent of single cell techniques finding subgroups within data has been abundant in biological publications for example for phylogenetic relatedness or to find genes with common expression among samples.

The single cell RNaseq field keeps on producing many publications on this subject including this excellent [review](https://www.nature.com/articles/s41576-018-0088-9) from Martin Hemberg's lab. Typically the use of either gold or silver standards allows you to determine how good your clustering method is. Gold standard means that you have some ground truth labels that you compare the results of your clustering method against. Silver standard means that you have enough knowledge about the labels from the underlying biology to evaluate results from clustering even if there's no ground truth available. Frequently peripheral blood mononuclear cells (PBMCs) are being used because they are well characterized and well separable based on their transcriptional profile. The following two publications provides some validation of clustering results both with both gold and silver standards ([Pub 1](https://f1000research.com/articles/7-1297/v2) and [Pub 2](https://www.nature.com/articles/nmeth.4236)).

Personally even though the above publications have been very helpful they still didn't really give me any concrete guidance in how to cluster my own dataset. In most cases "real" sc-RNAseq data will not contain as well characterized cells as PBMCs or even gold standard data. Since it might even be the purpose of your study to identify new subpopulations of cell states or cell types, I wanted to incorporate some metrics in my pipeline that evaluate clustering in the absence of ground truth data.

While looking for metrics that evaluate cluster properties in the absence of ground truth data I came across the [Silhouette Coefficient](https://en.wikipedia.org/wiki/Silhouette_(clustering)) and the [Davies-Bouldin index](https://en.wikipedia.org/wiki/Davies%E2%80%93Bouldin_index) which are part of Python's sckit-learn package. Briefly both methods evaluate clustering by comparing the distances of samples (or cells) within clusters to the distance of samples outside the cluster. There are subtle differences between performance and depending on cluster shape. More information on how each metric is calculated can be found on the [sckit-learn documentation](https://scikit-learn.org/stable/modules/clustering.html#clustering).

Before applying the Silhouette Coefficient and the Davies-Bouldin index to my own datasets I first wanted to test them using a silver standard dataset (3KPBMCs) that conveniently comes with all major sc-RNAseq analysis packages. I decided to follow the standard tutorial of [Scanpy](https://github.com/theislab/scanpy-tutorials/blob/master/pbmc3k.ipynb) (uses Python but based on the pipeline from the [popular R package Seurat](https://satijalab.org/seurat/v3.0/pbmc3k_tutorial.html)) and evaluate the default clustering methods by the Silhouette Coefficient and the Davies-Bouldin index. Running through these tutorials is usually pretty straight forward. However there are some input settings (also called hyperparameters) that need to be set by the user that will affect clustering. My approach was to alter the following two main hyperparameters in a combinatoric fashion and evaluate their effect on cluster size and associated metrics.
### 1) Number of k nearest neighbors (KNN)
The standard pipeline of Seurat/scanpy includes the generation of a nearest neighborhood graph. This is essentially finding the K number of closest cells for each cell in multidimensional space. [This blogpost](https://towardsdatascience.com/knn-k-nearest-neighbors-1-a4707b24bd1d) nicely explains k nearest neighborhood algorithm. The number of K nearest neighbors that each cell is associated with needs to be manually picked.
Scanpy's [description for the neighborhood function](https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.neighbors.html#scanpy.pp.neighbors) (scanpy.pp.neighbors) explains that "Larger values result in more global views of the manifold, while smaller values result in more local data being preserved. In general values should be in the range 2 to 100". I have yet to find a good reference that guides the choice of numbers of neighbors based on the type of biological sample used. The default value in the Seurat function [FindNeighbors](https://github.com/satijalab/seurat/blob/245d72b5f7c3ae75b185f2ae2d3a5b4c67c61e9a/R/clustering.R) is set to 20 so choosing nearest neighbors around that number seems to be a good starting point.
### 2) Resolution of the Louvain clustering
Once the neighborhood graph is created one can now cluster on this graph object (as well as project it into low dimensional space for visualization). Simply speaking Louvain clustering partitions the neighborhood graphs into different chunks separating the parts or groups of cells that are not as much connected. This clustering method has been found to work particularly well on single cell RNAseq data and was validated via gold standard methods in this [publication](https://www.sciencedirect.com/science/article/pii/S0092867415006376). Changing the resolution parameter determines which connections will be kept and in turn how many clusters will result.
To start with my analysis I ran through scanpys standard pipeline which uses 10 KNN nearest neighbors and a resolution parameters of 1 and came to this result of 8 clusters.

{:.text-center img}
![Default parmeters]({{ site.urlimg }}/media/clustering1.png "Default parmeters")
