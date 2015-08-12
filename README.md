# PyGraphistry: Explore Relationships

PyGraphistry is a bindings library to extract, transform, and load data in the [Graphistry's](http://www.graphistry.com) data explorer. Try the demo:

<table style="width:100%;">
  <tr valign="top">
    <td>Friendship communities on Facebook. <em>Click to open live!</em> (source: <a href="http://snap.stanford.edu">SNAP</a>)<br><a href="http://proxy-staging.graphistry.com/graph/graph.html?dataset=Facebook&debug=true&info=true&play=0&mapper=opentsdb&menu=false&static=true&contentKey=Facebook_readme&center=false&left=-28057.922443107804&right=19343.789165388305&top=-13990.35481117573&bottom=12682.885549380659#"><img src="http://i.imgur.com/CvO12an.png" title="Click to open."></a>
  </tr>
</table>

### PyGraphistry is...

- **Fast & Gorgeous** Our data explorer connects to Graphistry's GPU cluster show hundreds of thousand of nodes and edges in your browser. You can cluster, filter, and inspect large amounts of data at interactive speed.

-  **Notebook Friendly** PyGraphistry plays well with [IPython](http://ipython.org): You can process and visualize data directly within your notebooks.

- **Science Ready** PyGraphistry works out-of-the-box with popular data science libraries. It is also very easy to use. To create the visualization shown above, download  [this dataset](https://www.dropbox.com/s/csy1l8e3uv600mj/facebook_combined.txt?dl=1) of Facebook communities from [SNAP](http://snap.stanford.edu) and use your favorite library to load it.

  - [Pandas](http://pandas.pydata.org)

     ```python
     df = pandas.read_csv('facebook_combined.txt', sep=' ', names=['src', 'dst'])
     graphistry.bind(source='src', destination='dst').plot(df)
     ```

  - [Igraph](http://igraph.org)

     ```python
     ig = igraph.read('facebook_combined.txt', format='edgelist', directed=False)
     graphistry.bind(source='src', destination='dst').plot(ig)
     ```

  - [NetworkX](https://networkx.github.io)

     ```python
     g = networkx.read_edgelist('facebook_combined.txt')
     graphistry.bind(source='src', destination='dst', node='nodeid').plot(g)
     ```

### Gallery

<table>
    <tr valign="top">
        <td width="33%">Twitter Botnet<br><a href="http://i.imgur.com/qm5MCqS.jpg"><img width="266" src="http://i.imgur.com/qm5MCqS.jpg"></a></td>
        <td width="33%">Edit Wars on Wikipedia<br><a href="http://i.imgur.com/074zFve.png"><img width="266" src="http://i.imgur.com/074zFve.png"></a></td>
        <td width="33%">Uber Trips in SF<br><a href="http://http://i.imgur.com/GdT4yV6.jpg"><img width="266" src="http://i.imgur.com/GdT4yV6.jpg"></a></td>
    </tr>
    <tr valign="top">
        <td width="33%">Attackers Port Scanning a Network<br><a href="http://i.imgur.com/vKUDySw.png"><img width="266" src="http://i.imgur.com/vKUDySw.png"></a></td>
        <td width="33%">Interactions Between Proteins (<a href="http://thebiogrid.org">BioGRID</a>)<br><a href="http://i.imgur.com/nrUHLFz.png"><img width="266" src="http://i.imgur.com/nrUHLFz.png"></a></td>
        <td width="33%">Programming Languages Adoption<br><a href="http://i.imgur.com/0T0EKmD.png"><img width="266" src="http://i.imgur.com/0T0EKmD.png"></a></td>
    </tr>
</table>

## Installation

You need [Python](https://www.python.org) 2.7 or 3.4. The simplest way to install PyGraphistry is pip:

- With Pandas only: `pip install graphistry`
- With Pandas, IGraph, and NetworkX: `pip install "graphistry[all]"`

##### API Key
You need and API key to connect to our GPU cluster. We ask for API keys to make use our servers are not melting :) To get your own, email us at [pygraphistry@graphistry.com](mailto:pygraphistry@graphistry.com). Register you key after the `import graphistry` statement and you are good to go:

```python
import graphistry
graphistry.register(key='<Your key>')
```

##### Working IPython (Jupyter) Notebooks

We recommend [IPython](http://ipython.org) notebooks to interleave code and visualizations.

- Install IPython:`pip install "ipython[notebook]"`
- Launch notebook server: `ipython notebook`

## Tutorial: Les Misérables

For this example, we use [Pandas](http://pandas.pydata.org) to load/process data and [Igraph](http://igraph.org) to run a community detection algorithm. You can download the [IPython notebook](https://www.dropbox.com/s/n35ahbhatshrau6/MiserablesDemo.ipynb?dl=1) containing this example.

#### Loading the Dataset
Let's load the characters from [Les Misérables](http://en.wikipedia.org/wiki/Les_Misérables). Our [dataset is a CSV file](http://gist.github.com/thibaudh/3da4096c804680f549e6/) that looks like this:

| source        | target        | value  |
| ------------- |:-------------:| ------:|
| Cravatte |	Myriel | 1| Valjean	| Mme.Magloire | 3| Valjean	| Mlle.Baptistine | 3

*Source* and *target* are character names, and the *value* column counts the number of time they meet. Parsing the data is a one-liner with Pandas:

```python
import pandas
links = pandas.read_csv('./lesmiserables.csv')
```

#### Quick Visualization
The PyGraphistry can plot graphs directly from Pandas dataframes, IGraph or NetworkX graphs. Calling *plot* uploads the data to our visualization servers and return an URL to an embeddable webpage containing the visualization.

To create a graph, we bind *source* and *destination* to the columns indicating the start and end nodes of each edges.

```python
import graphistry
graphistry.register(key='YOUR_API_KEY_HERE')

plotter = graphistry.bind(source="source", destination="target")
plotter.plot(links)
```

You should see a beautiful graph like this one:
![Graph of Miserables](http://i.imgur.com/dRHHTyK.png)

### Adding Labels

Let's add label to edges showing how many time each pair of characters met. We create a new column called *label* in *links* containing the text of the label and bind *edge_label* to it.

```python
links["label"] = links.value.map(lambda v: "#Meetings: %d" % v)
plotter = plotter.bind(edge_label="label")
plotter.plot(links)
```

### Controling Node Size and Color
We are going to use [Igraph](http://igraph.org/python/) to size nodes based on their [PageRank](http://en.wikipedia.org/wiki/PageRank) score and color them using their [community](https://en.wikipedia.org/wiki/Community_structure). If Igraph is not already installed, fetch it with `pip install igraph-python`. (Warning: `pip install igraph` will install the wrong package!)

We start by converting our edge dateframe to an Igraph. The plotter can do the conversion for us using the *source* and *destination* bindings. By computing PageRank and community clusters, we create two new node attributes (*pagerank* & *community*).

```python
ig = plotter.pandas2igraph(links)
ig.vs['pagerank'] = ig.pagerank()
ig.vs['community'] = ig.community_infomap().membership

plotter.bind(point_color='community', point_size='pagerank').plot(ig)
```

![Second Graph of Miserables](http://i.imgur.com/P7fm5sn.png)

## Going Further: Marvel Comics

This is a more complex example: we link together Marvel characters who co-star in the same comic. The dataset is split in three files:

- [appearances.txt](https://www.dropbox.com/s/yz78yy58m1mh8l2/appearances.txt?dl=1)
- [characters.txt](https://www.dropbox.com/s/7zodqsvqa9j29bb/characters.txt?dl=1)
- [comics.txt](https://www.dropbox.com/s/x1o30enl5abdpnm/comics.txt?dl=1)

Find out who is the most popular Marvel hero! Run the code in [the Marvel Demo notebook](https://www.dropbox.com/s/mzzq1mvpdwwmes1/MarvelTutorial.ipynb?dl=1) to browse the entire Marvel universe.

![Marvel Universe](http://i.imgur.com/0rgPLg7.png)

## API Reference

Have a look at the full [API](http://graphistry.com/api0.3.html#python) for more information on sizes, colors, palettes etc.

<!---

### Cheat Sheet
In a nutshell, `plot` *mandatory* arguments are:

- `edges` *pandas.DataFrame*: The edge dataframe.
- `source` *string*: The column of `edges` containing the start of each edge.
- `destination` *string*: The column of `edges` containing the end of each edge.

This is enough to define a graph.
##### Edges
We control the visual attributes of edges with the following *optional* arguments. Each of them refers to the name of a column of `edges`.

- `edge_color` *string*
- `edge_title` *string*
- `edge_label` *string*
- `edge_weight` *string*

##### Nodes
To control node visual attributes, we pass two more arguments:

- `nodes` *pandas.DataFrame*: The node dataframe.
- `node` *string*: The column of `nodes` that contains node identifiers (these are the same ids used in the `source` and `destination` columns of `edges`).

then we bind columns of `node` using:

- `point_title` *string*
- `point_label` *string*
- `point_size` *string*
- `point_color` *string*

-->


