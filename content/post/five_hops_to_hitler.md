---
title: "Five Hops To Hitler"
date: "2016-03-15"
description: "According to a fellow student, a general assumption exists, stating that almost no article on Wikipedia is more than five degrees seperated from the article about Adolf Hitler. To verify this assumption, I have analyzed the German Wikipedia and computed the shortest distance between all vertices and the vertex representing the article about Adolf Hitler. The result shows that 97.1% of all articles have a path length of ≤ 5."
categories:
  - "Graph Theory"
  - "Research"
tags:
  - "Random"
---

**According to a friend, a common assumption exists, stating that almost no article on Wikipedia is more than five degrees separated from the article about Adolf Hitler. To verify this assumption, I have analyzed the German Wikipedia and computed the shortest distance between all vertices and the vertex representing the article about Adolf Hitler. The result shows that 97.1% of all articles have a path length of ≤ 5.**

## I. Introduction

> "No Wikipedia article is more than five degrees seperated from the article about Hitler."<br>
> &mdash; Diana

This hypothesis was mentioned by my friend Diana during lunch at the TUM cafeteria. Remembering the ’Six Degrees of Seperation’ [1] and the ’Bacon Number’ [2], it sparked my interest in finding evidence for or against her statement.

To simplify computation, for this study I’ve reduced the dataset to article names and ingoing and outgoing internal links. Based on this extremely flat, but broad information structure we can already start answering some interesting questions when it comes to relatedness and importance of articles from the corpus.

We can for example compute the indegrees and outdegrees of nodes we’re interested in or determine the shortest path between nodes. To find evidence for the stated hypotheses, I’ve looked at the shortest path from all vertices in the graph to the vertex representing the article about ’Adolf Hitler’.

## II. Data and Methods
Wikipedia itself is a conceptually perfect source to answer questions like ’How far away is X from Y?’. The concept of distance can be treated in different ways. One of the simplest ways is to treat the corpus of articles as an unweighted, directed graph structure. The vertices in our graph represent the articles and internal links are represented by the directed edges in between. The direction of edges follows the direction of hyperlinks.

### A. Dataset
I’ve conducted the evaluation on the German edition Wikipedia as of December 2015. The dataset was acquired from a Wikipedia full [article dump](https://dumps.wikimedia.org/dewiki/20151201/dewiki-20151201-pages- articles.xml.bz2) and post processed with [Graphipedia](https://github.com/mirkonasato/graphipedia), a tool to prepare Wikipedia dumps for import into a [Neo4j](http://www.neo4j.com) database.

The imported dataset contains 3,169,885 pages connected by 48,993,269 links. 6,466,688 broken links have been removed before and are not included in the tested dataset.

### B. Computation

For computing the results, Neo4j, a database designed for storing and querying graph structures has been used. Neo4j implements its own shortest path algorithm. The following query was used to compute the results.

```
MATCH (a:Page), (b:Page {title:’Adolf Hitler’}), p=shortestPath ((a)−[*..15]−>(b))
RETURN size(nodes(p)), count(*);
```

Neo4j uses a Breadth-first search (BFS) algorithm to determine the shortest path when analyzing a directed, unweighted graph. BFS has a worst-case runtime complexity of `O(V + E)` Where V is the total number of vertices and E is the total number of edges. The BFS for this study was configured to not include &#8467; > 14.

## III. Results

From the total of 3,169,885 vertices, 97.1% have &#8467; ≤ 5 to reach the article about ’Adolf Hitler’. For distribution see the following figures.

![distribution](/images/posts/hops.png)

<table>
    <tr>
        <td>Path length &#8467;&nbsp;&nbsp;&nbsp;&nbsp;</td>
		<td>Vertices V(&#8467;)</td>
    </tr>
    <tr>
        <td>1</td>
		<td>7,139</td>
    </tr>
    <tr>
        <td>2</td>
		<td>805,640</td>
    </tr>
    <tr>
        <td>3</td>
		<td>1,596,007</td>
    </tr>
    <tr>
        <td>4</td>
		<td>648,934</td>
    </tr>
    <tr>
        <td>5</td>
		<td>20,357</td>
    </tr>
    <tr>
        <td>6</td>
		<td>189</td>
    </tr>
</table>

## IV. Conclusion

The resulting numbers show, that 97.1% of German Wikipedia articles are less than 6 hops away from the article about ’Adolf Hitler’. Therefore we can conclude that the stated hypothesis is confirmed and the ’urban myth’ can be considered a fact.

Further interesting questions around this topic can be certainly asked. Especially since we’re looking at a topic that’s culturally extremely relevant for the community that created and maintains the analyzed Wikipedia corpus, it might be of interest to compare results with Wikipedia editions in other languages.

<small>____

[1] J. Guare, Six Degrees of Separation: A Play, Vintage Books, New York, 1990.<br/>
[2] A. Ruthven, (April 7, 1994). Kevin Bacon is the Center of the Universe. https://groups.google.com/d/topic/rec.arts.movies/-qNue6RwTn8/discussion. Retrieved December 12, 2015.
</small>
