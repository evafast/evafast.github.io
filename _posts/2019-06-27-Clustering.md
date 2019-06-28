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
![Default parameters]({{ site.urlimg }}/media/clustering1.png "Default parameters")

I now wanted to try out how changing the number of KNN nearest neighbors and the Louvain resolution parameters would change the cluster number. Furthermore I wanted to look at how Silhouette Coefficient and the Davies-Bouldin index behaved with different KNN and resolution parameters.
The table below shows the numbers of clusters for different resolution parameters in columns versus different KNNs in the rows. For easy visualization a color scale is added with white representing higher and dark red lower values. You can see that cluster number increases with increasing resolution and decreases with higher numbers of KNN. This makes sense because with a higher number of connections (higher KNN) for each cell will build a more "global" graph, being less able to pick up smaller communities. Similarly increasing Louvain resolution results in more connections being "cut" and thus increasing cluster number.

{:.text-center img}
![Cluster numbers]({{ site.urlimg }}/media/clustering2.png "Cluster numbers")
Numbers of Clusters with different KNN (rows) and Louvain resolution (columns) parameters

Here is the same table but with the Silhouette Coefficient (the higher the better) for different KNN and resolution combinations.
{:.text-center img}
![Silhouette Coefficients]({{ site.urlimg }}/media/clustering3.png "Silhouette Coefficients")
Silhouette Coefficient with different KNN (rows) and Louvain resolution (columns) parameters

Here is the same table but with the Davies-Bouldin index (the lower the better) for different KNN and resolution combinations.
{:.text-center img}
![Davies-Bouldin index]({{ site.urlimg }}/media/clustering4.png "Davies-Bouldin index")
Davies-Bouldin index with different KNN (rows) and Louvain resolution (columns) parameters

Just from looking at the tables a high Louvain resolution combined with a low KNN gives high cluster numbers but also suboptimal scores in the two metrics. To make the selection of cluster number even easier I plotted the optimal value for Silhouette score and Davies-Bouldin index versus increasing numbers of clusters.
{:.text-center img}
![Optimal cluster number]({{ site.urlimg }}/media/clustering5.png "Optimal cluster number")
Optimal value of Davies-Bouldin index (minimum) and Silhouette score (maximum) plotted versus the cluster numbers.

This graph illustrates that a separation of this dataset in up to eight clusters results in a pretty stable (linear) metrics. This means that up to eight clusters give a "good" clustering score using both Davies-Bouldin index and the Silhouette score. With 9 and 10 clusters the metrics pretty drastically deteriorate (decrease in Silhouette score and increase of Davies-Bouldin index). Cells are at this point are likely over clustered into groups that are likely not really different from each other. Since a common goal of single cell RNAseq experiments is to find hidden heterogeneities a good strategy is to look for the maximum cluster number until there is a sudden dropoff in clustering score. In our case that would be a cluster number of eight. Let's look at the different KNN and Louvain resolution parameters that led to a cluster number of 8. The table below lists different KNN and Louvain resolution combinations ranked by decreasing Silhouette scores (mostly matching increasing Davies-Bouldin index).
{:.text-center img}
![Table of optimal cluster number]({{ site.urlimg }}/media/Capture1.JPG "Table of optimal cluster number")

As a nice positive control the original parameters from the tutorial (10 KNN and Louvain resolution of 1) have the second most optimal performance by the two metrics. Next, let's look at the clustering result by projecting the data in low dimensional UMAP space. I plotted the top 3 results as well as the bottom result with the "worst" scores.

**Clustering KNN:5 und resolution:0.9**
{:.text-center img}
![KNN5_res0.9]({{ site.urlimg }}/media/clustering6.png "KNN5_res0.9")

**Clustering KNN:10 und resolution:1.0**
{:.text-center img}
![KNN10_res1.0]({{ site.urlimg }}/media/clustering7.png "KNN10_res1.0")

**Clustering KNN:10 und resolution:1.1**
{:.text-center img}
![KNN10_res1.1]({{ site.urlimg }}/media/clustering8.png "KNN10_res1.1")

**Clustering KNN:5 und resolution:1.0**
{:.text-center img}
![KNN5_res1.0]({{ site.urlimg }}/media/clustering10.png "KNN5_res1.0")

When comparing the top three with the worst result one can immediately see the difference. Clusters in the top three are very well separated whereas in the bottom one cluster 2 and 0 are heavily intermixed which is likely the cause of the low score.
Result 2 and 3 is expectedly very similar since the hyper parameters are almost the same. Only a couple of cells from cluster 3 and cluster 0 are differentially assigned. Result 1 does give a slightly different result. The big cell population on the lower right is split up differently between 3 clusters. In result 1 it has two main clusters (0 and 2) and only a very small population belonging to cluster 6, whereas in Result 2 and 3 there is a much more even distribution between clusters. So which result is real? At this point the metrics don't really help but it is advisable to look more into the actual genes that define each of the clusters in this case a T-cell/NK cell cluster.
Interestingly, even the tutorials of Scanpy and Seurat reach a different clustering result of this population in their tutorials which illustrates that the clusters within this population are less "stable".

[Seurat](https://satijalab.org/seurat/v3.0/pbmc3k_tutorial.html) finds four subpopulations in their T-cell/NK cell cluster (lower left).
{:.text-center img}
![Seurat clustering]({{ site.urlimg }}/media/seurat_clustering.png "Seurat clustering")

[Scanpy](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html) finds three subpopulations in their T-cell/NK cell cluster (lower right).  
{:.text-center img}
![Scanpy clustering]({{ site.urlimg }}/media/scanpy_clustering.png "Scanpy clustering")


### Conclusions:

In summary the Silhouette score and Davies-Bouldin index can be used to help determine the optimal cluster number and pick hyperparameters associated with optimal clustering results. Here I performed clustering according to the default tutorials by the packages Seurat and Scanpy. Recently this very comprehensive [sc-RNAseq processing tutorial](https://www.embopress.org/doi/10.15252/msb.20188746) was published that gives a lot of detailed information about all the processing steps including clustering. There are of course a many different clustering algorithms that can be used for single cell analysis (more info [here](https://f1000research.com/articles/7-1141)). The metrics presented here should be applicable to all of them. Another useful approach to evaluate clustering results is by random subsampling the dataset and checking how stable clustering is between these "replicates". This [review](https://www.sciencedirect.com/science/article/pii/S0098299717300493#bib51)
 references a number of original papers that use the subsampling approach. [Scclusteval](https://github.com/crazyhottommy/scclusteval) is a handy R package that does the cluster subsampling on single cell data. Finally there's a number of approaches ([semi-soft clustering](https://www.pnas.org/content/116/2/466)
and using [topic-modelling](https://www.biorxiv.org/content/biorxiv/early/2018/11/12/461228.full.pdf)) that go beyond "hard" clustering, allowing cells not to be strictly assigned to one cluster only. This is especially handy when evaluating transitional states as often the case in Biology.

To find the full analysis - check out the [Jupyter notebook](https://github.com/evafast/Cluster_Testing/blob/master/DBindex_Silscore_clustering_tutorial.ipynb) on my Github page. Still need to set up a Binder and for some reason the clusters switched but all the analysis from this blogpost is there.
