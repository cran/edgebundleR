# Circle plot with bundled edges

[![Travis-CI Build Status](https://travis-ci.org/garthtarr/edgebundleR.svg?branch=master)](https://travis-ci.org/garthtarr/edgebundleR) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/edgebundleR)](http://cran.r-project.org/package=edgebundleR/) [![](http://cranlogs.r-pkg.org/badges/edgebundleR)](http://cran.r-project.org/package=edgebundleR)

This package allows R users to easily create a hierarchical edge bundle plot.  The underlying D3 code was adapted from  Mike Bostock's examples (see [here](http://bl.ocks.org/mbostock/7607999) or [here](https://mbostock.github.io/d3/talk/20111116/bundle.html)) and the package is based on the [htmlwidgets](https://github.com/ramnathv/htmlwidgets) framework.

Many thanks to [timelyportfolio](https://github.com/timelyportfolio) for some major improvements.

## Installation

You can install `edgebundleR` from Github using the `devtools` package as follows:

```s
# install.packages("devtools")
devtools::install_github("garthtarr/edgebundleR")
```

Or you can get it on CRAN:

```s
install.packages("edgebundleR")
```


## Usage

```s
require(edgebundleR)
```

The main function in the edgebundleR package is `edgebundle()`.  It takes in a variety of inputs:
 - an igraph object
 - a symmetric matrix, e.g. a correlation matrix or (regularised) precision matrix
 - a JSON file structured with `name` and `imports` as the keys


The result of the `edgebundle()` function is a webpage that is rendered in the RStudio Viewer pane by default, but also may be exported to a self contained webpage, embedded in an Rmarkdown document or used in a Shiny web application.

### Input options

#### igraph object

Given an igraph object as the input, the function will extract the linkages and plot them.  For example,

```s
require(igraph)
ws_graph <- watts.strogatz.game(1, 50, 4, 0.05)
edgebundle(ws_graph,tension = 0.1,fontsize = 18,padding=40)
```

Here's a more complicated example adapted from [this](http://stackoverflow.com/questions/30708674/network-chord-diagram-woes-in-r/32260962#32260962) stackoverflow question and answer.

```s
library(igraph)
library(data.table)
d <- structure(list(ID = c("KP1009", "GP3040", "KP1757", "GP2243",
                           "KP682", "KP1789", "KP1933", "KP1662", "KP1718", "GP3339", "GP4007",
                           "GP3398", "GP6720", "KP808", "KP1154", "KP748", "GP4263", "GP1132",
                           "GP5881", "GP6291", "KP1004", "KP1998", "GP4123", "GP5930", "KP1070",
                           "KP905", "KP579", "KP1100", "KP587", "GP913", "GP4864", "KP1513",
                           "GP5979", "KP730", "KP1412", "KP615", "KP1315", "KP993", "GP1521",
                           "KP1034", "KP651", "GP2876", "GP4715", "GP5056", "GP555", "GP408",
                           "GP4217", "GP641"),
                    Type = c("B", "A", "B", "A", "B", "B", "B",
                             "B", "B", "A", "A", "A", "A", "B", "B", "B", "A", "A", "A", "A",
                             "B", "B", "A", "A", "B", "B", "B", "B", "B", "A", "A", "B", "A",
                             "B", "B", "B", "B", "B", "A", "B", "B", "A", "A", "A", "A", "A",
                             "A", "A"),
                    Set = c(15L, 1L, 10L, 21L, 5L, 9L, 12L, 15L, 16L,
                            19L, 22L, 3L, 12L, 22L, 15L, 25L, 10L, 25L, 12L, 3L, 10L, 8L,
                            8L, 20L, 20L, 19L, 25L, 15L, 6L, 21L, 9L, 5L, 24L, 9L, 20L, 5L,
                            2L, 2L, 11L, 9L, 16L, 10L, 21L, 4L, 1L, 8L, 5L, 11L),
                    Loc = c(3L, 2L, 3L, 1L, 3L, 3L, 3L, 1L, 2L,
                            1L, 3L, 1L, 1L, 2L, 2L, 1L, 3L,
                            2L, 2L, 2L, 3L, 2L, 3L, 2L, 1L, 3L, 3L, 3L, 2L, 3L, 1L, 3L, 3L,
                            1L, 3L, 2L, 3L, 1L, 1L, 1L, 2L, 3L, 3L, 3L, 2L, 2L, 3L, 3L)),
               .Names = c("ID", "Type", "Set", "Loc"), class = "data.frame",
               row.names = c(NA, -48L))
# let's add Loc to our ID
d$key <- d$ID
d$ID <- paste0(d$Loc,".",d$ID)
# Get vertex relationships
sets <- unique(d$Set[duplicated(d$Set)])
rel <-  vector("list", length(sets))
for (i in 1:length(sets)) {
  rel[[i]] <- as.data.frame(t(combn(subset(d, d$Set ==sets[i])$ID, 2)))
}
rel <- rbindlist(rel)
# Get the graph
g <- graph.data.frame(rel, directed=F, vertices=d)
clr <- as.factor(V(g)$Loc)
levels(clr) <- c("salmon", "wheat", "lightskyblue")
V(g)$color <- as.character(clr)
V(g)$size = degree(g)*5
# igraph static plot
# plot(g, layout = layout.circle, vertex.label=NA)

edgebundle( g )
```

#### Symmetric matrix

```s
require(MASS)
sig = kronecker(diag(3),matrix(2,5,5)) + 3*diag(15)
X = MASS::mvrnorm(n=100,mu=rep(0,15),Sigma = sig)
colnames(X) = paste(rep(c("A.A","B.B","C.C"),each=5),1:5,sep="")
edgebundle(cor(X),cutoff=0.2,tension=0.8,fontsize = 14)
```

Alternatively, you could do some regularisation and plot the results of that:
```s
require(huge)
data("stockdata")
# generate returns sequences
X = log(stockdata$data[2:1258,]/stockdata$data[1:1257,])
# perform some regularisation
out.huge = huge(cor(X),method = "glasso",lambda=0.56,verbose = FALSE)
# identify the linkages
adj.mat = as.matrix(out.huge$path[[1]])
# format the colnames
nodenames = paste(gsub("","",stockdata$info[,2]),stockdata$info[,1],sep=".")
head(cbind(stockdata$info[,2],stockdata$info[,1],nodenames))
colnames(adj.mat) = rownames(adj.mat) = nodenames
# restrict attention to the connected stocks:
adj.mat = adj.mat[rowSums(adj.mat)>0,colSums(adj.mat)>0]
# plot the result
edgebundle(adj.mat,tension=0.8,fontsize = 10)
```


#### JSON file

If you already have an appropriately formatted JSON file with `name` and `imports` as the keys linking various nodes, you can load it directly as follows:

```s
filepath = system.file("sampleData", "flare-imports.json", package = "edgebundleR")
edgebundle(filepath,width=800,fontsize=8,tension=0.95)
```

In this example, the first few lines of the file are:

```s
system(paste("head -4",filepath))
```
```
[
{"name":"flare.analytics.cluster.AgglomerativeCluster","size":3938,"imports":["flare.animate.Transitioner","flare.vis.data.DataList","flare.util.math.IMatrix","flare.analytics.cluster.MergeEdge","flare.analytics.cluster.HierarchicalCluster","flare.vis.data.Data"]},
{"name":"flare.analytics.cluster.CommunityStructure","size":3812,"imports":["flare.analytics.cluster.HierarchicalCluster","flare.animate.Transitioner","flare.vis.data.DataList","flare.analytics.cluster.MergeEdge","flare.util.math.IMatrix"]},
{"name":"flare.analytics.cluster.HierarchicalCluster","size":6714,"imports":["flare.vis.data.EdgeSprite","flare.vis.data.NodeSprite","flare.vis.data.DataList","flare.vis.data.Tree","flare.util.Arrays","flare.analytics.cluster.MergeEdge","flare.util.Sort","flare.vis.operator.Operator","flare.util.Property","flare.vis.data.Data"]},
```

The important elements are the `name` and `imports` keys.  In the current implementation, `size` is ignored.  Note the dots in the node names, these are used to do the clustering.  For example these first three nodes would all appear grouped together in the graph as they all start with `flare.analytics.cluster`. You can have multiple levels of clustering (hierarchical clustering) using different depths in the naming convention.  Any text after the final dot will be rendered as the node label in the graph.

## Output options

#### Within RStudio

When running (recent versions of) RStudio, the default behaviour is for the plot to render in the Viewer pane.  You should not specify the width and height parameters, as these will override the dynamic resizing behaviour.

For example, the following code would generate a plot that dynamically resizes to fit in the Viewer pane:

```s
ws_graph = watts.strogatz.game(1, 50, 4, 0.05)
edgebundle(ws_graph,tension = 0.1,fontsize = 20)
```

#### Stand alone web page

You can open the graph in a web browser from RStudio using the "Show in new window" icon.  If you would like to save the webpage to share with others, the best option is to use the `saveEdgebundle` function:

```s
g = edgebundle(ws_graph,tension = 0.1,fontsize = 20,width=600,height=600)
saveEdgebundle(g,file = "ws_graph.html")
```

This will create a fully self contained html file that renders reliably in most browsers.

#### Rmarkdown document

Simply set the code chunk argument `results='asis'`.

#### Shiny application

Using the `shinyedge` function, you can interactively adjust the font size, height/width and tension then export the graph to a self contained html file.  The input to the `shinyedge` function is a JSON file, igraph object or symmetric matrix (the same as the `edgebundle` function).

```s
g1 = watts.strogatz.game(1, 100, 4, 0.05)
shinyedge(g1)
```

If you are building your own Shiny app, you can use the standard output and render functions: `edgebundleOutput` and `renderEdgebundle`.

## Saving and sharing

If you would like to save an image (png, jpeg or tiff) you can use the export drop down menu from the RStudio viewer pane.  To save as a pdf, the easiest option is to view the graph in a web browser then print to pdf.
