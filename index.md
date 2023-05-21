# Web Scraping with R Studio: A gentle introduction

**Welcome to this short tutorial on Web Scraping!**

## A Case Study: The Wind Energy Across Europe 
We want to explore the relational network that the European organizational actors involveed the wind energy sector eventually build within the digital environment.
In order to do so, we have identified a promising source of information, the Wind Energy Europe, wich collect more than 600 organization involved in wind energy project.

Our formal research question is:
***RQ: Which organizations build the densest relational network in the digital environment?***

Let's try to address it through R-based tools.
You can use the both your local R Studio or an [R Studio Cloud Project](https://posit.cloud/) to start your scraping task!

## The sampling procedure
Let’s start by finding and downloading the Excel file of your group here:
[group_blue.xlsx](https://github.com/nucciodinome/Web-Scraping-Text-Mining-with-RStudio/files/11525141/group_blue.xlsx)
[group_red.xlsx](https://github.com/nucciodinome/Web-Scraping-Text-Mining-with-RStudio/files/11525145/group_red.xlsx)
[group_green.xlsx](https://github.com/nucciodinome/Web-Scraping-Text-Mining-with-RStudio/files/11525144/group_green.xlsx)

Now, you can open R Studio and import the file.
As you can see, the dataset is a list of URLs, named SEED, which refer to the websites of 50 members of Wind Energy Europe.

Given RQ, you should first identify what data you need to track relationships between organizations.
As we know, information about the relationships between two websites is encoded as hyperlinks, which, in the HTML language are reported with the "a" tag, while the target URL of the link is made explicit with the "href" attribute.
We can assume, moreover, that not all website links are on the homepage: many of them will be found on the internal pages of the website (e.g., "about" page, "partners" page, "clients" page, etc.).

Let's assume, then, that we also want to investigate the internal pages which are directly linked to the homepage, or, in other words, at 1 degree of separation from the homepage (let's call them second-level web pages).

We can then start to build our procedure aimed at extracting the data we need and then identify an R script for web scraping.

Probably, your procedure will look like this:

- [x] Starting from the list of 50 SEEDs...
- [ ] __STEP I__: Extract all the _Second Level web pages_ in order to...
- [ ] __STEP II__: Collect all _external links_ (which refer to other organizations' websites connected to the SEEDs


### STEP Ia: Extract Second Level web pages with Rvest
Having the SEEDs it should be easy to extract all the internal links which refer to second level web pages.
*N.b. The :x: are the fields you need to fill in.*



#### Import R packages
Let's try the R package "Rvest." (and some other additional packages)

```markdown

install.packages(rvest)
install.packages( (stringr)
install.packages( (dplyr)

#then activate them:

library(rvest)
library(stringr)
library(dplyr)

```

#### Run Rvest script for a given URL within the SEED list
A simple script like this should do the trick, let’s try with the first SEED of our list:
```markdown
  page = read_html(group X$SEED[1])
  a_ref <- page %>% html_nodes("X")
  links <- a_ref %>% html_attr("X")
```

#### Loop over the SEED list

```markdown
#Create an empty vector
links=c()

#loop the extraction of links for all the SEEDs
 for (i in X) {  
page = read_html(X)
  	a_ref <- page %>% html_nodes("X")
  	link_seed <- a_ref %>% html_attr("X")
	links <- c(links, link_seed)
 }
 #Inspect the result: does it look satisfying in order to proceed to the next step? 
Look at the extracted URLs...

links
 
```

### STEP Ib: Extract Second Level web pages with Rcrawler
Perhaps, there are other ways to proceed in this case.
Let's try the "Rcrawler" package.

#### Install Rcrawler

```markdown
install.packages(Rcrawler)
library(Rcrawler)
```

#### Use LinkExtractor function
The linkExtractor function, available in the "Rcrawler" package allows you to simplify the process of link extraction: let’s try!

```markdown
#create an empty vector
all_links=c()

#loop search for second level web pages
for(i in X) {  
  tryCatch({
scraped<-LinkExtractor(url=i,lev= 1, ExternalLInks = FALSE, IndexErrPages= c(200,300,403,404),RenderingDelay=3, Timeout = 6)
  	links <-scraped$InternalLinks
  	all_links <- c(all_links, links)
  	print(i)
  	}, error=function(e){cat("ERROR :",conditionMessage(e), "\n")
  })
}
#Take a look at the results:
all_links
```

Maybe we should filter for duplicates URLs

```markdown
all_links<-unique(all_links)
```

Seems to be far better. We can proceed to STEP II.


### STEP II: Extract External links from Second Level web pages
Now we have a complete list of second-level URLs for each of our SEEDs.
We can then replicate the previous script to extract all the external links for second level web page. Remember to enable External Links extraction.

```markdown
#create the empty vectors
out_links=c()
int_links=c()

#loop search for external links: 500URLs are enough for this exercise
for(i in all_links) {  
  tryCatch({
    scraped <-LinkExtractor(url=i,lev= 1, ExternalLInks = TRUE, IndexErrPages=c(200,300,401,403,404))
    links <- scraped$ExternalLinks
    out_links <- c(out_links, links)
    print(i)
    for (j in links){
      int_links<-c(int_links, i)
    }
  }, error=function(e){cat("ERROR :",conditionMessage(e), "error_occurred")
  })
}
```

## Addressing RQ: Building the Hyperlink Network
Now you should have some nice data. We should be able to address RQ.
Take another look at it:
***RQ: Which organizations build the densest relational network in the digital environment?***

There are many methods to address this question… let's try to look at it from a network perspective.
By transposing our data into a graph, it should be easy to represent organizations as nodes, and the links as edges between them.

The "relational ability" of organizations, in this perspective, can be operationalized as the number of external links produced by each of the organizations' websites: the outdegree centrality of nodes!

### Data cleaning and normalization
Now we have two vectors: “int_links”, which contains the URLs of our organizations, and “out_links”, which contains the target URLs of the links. Let’s merge them in a data.frame:

```markdown
edges <- data.frame(int_links, out_links) 

#Take a look at it:
edges
```

### Normalizing URL addresses via urltools
Do you see any problem with the data?
Because of the procedure used, our organizations are fragmented into a multitude of URLs (see $int_links column). There are also many links of little interest in the $out_link column, such as those directed to social networks (Facebook, Twitter, Linkedin, etc.) and other external services (e.g., Google).
Before drawing the network, we should normalize and clean the data.
At first, we need to “normalize” the data by reconciling each URL with its domain.
The package "urltools" can help us.

#### Install urltools

```markdown
install.packages(urltools)
library(urltools)
```

#### Normalize URLs
A short script will provide us the information we need: 

```markdown
in_domain <-suffix_extract(domain((edges$X)))
out_domain <-suffix_extract(domain((edges$X)))
edges$source <-in_domain$domain
edges$target <-out_domain$domain
edges$int_links<-NULL
edges$out_links<-NULL
```

#### Filter for useless URLs
We can also use a script that filters results based on text (REGEX), in order to remove useless URLs from the $target column.

```markdown
#Take a look at most frequent target URLs:
sort(table(edges$X), decreasing=TRUE)[1:20]

#Filter the $target URLs (replace the X with keywords contained in the URLs you would like to remove):
filter <- paste(c("X", " X", " X", " X", " X"),collapse = '|')
edges_clean <- edges[!grepl(filter,as.character(edges$ X)),]
rownames(edges_clean) <- NULL  
```


### Estimating network and centralities
Now we can use Igraph to build the network:

#### Building a network in Igraph
```markdown
install.packages(Igraph)
library(Igrapg)

g<- graph_from_data_frame(edges_clean, directed = TRUE, vertices = NULL)

#remove self-loops
gs<-simplify(g)
```

#### Estimating nodes centrality measures
We can calculate the in- and out-degree centrality indices as follows:

```markdown
Indegree <- degree(gs, mode="in")
Outdegree <- degree(gs, mode="out")
```

#### Network visualization

```markdown
viz<-layout_(gs, with_fr(grid = "auto", niter=5000))
plot(gs, layout = viz, edge.arrow.size=.2, vertex.size= 10, 
vertex.color="lightblue", vertex.frame.color="gray", vertex.label.color="black", vertex.label.cex= .8, vertex.label.dist=3)
```

### Results
***So, which organizations builds the densest relational network in the digital environment??***
```markdown
sort(Outdegree,decreasing=TRUE)[1:10]
```



## Support or Contact

Having trouble with example code? Some new questions or ideas you want to explore?
Contact me at _nuccio.ludovico@gmail.com_
