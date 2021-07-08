# Branching-Out-Graph-Theory-Fundamentals

## Modeling a Network

To model a network in Python, first start by defining a node, which requires an id. In this case, the node wants to have the attributes name, party, and state. These are defined in a separate class method so that if they are not available the node can still be initiated. The graph object itself is a simple container containing a node object and an edge object. The node object is structured as a dictionary to allow lookups, and an extension to this model could be do redefine the edge object as a dictionary of source, target keys. This would allow lookups on edges. The toy model is then created by calling the Graph class.

```python
# define node object
class Node:
    def __init__(self, id):
        self.id = id
        self.has_attrs = False

    def __str__(self):
        return self.id

    def __repr__(self):
        return self.id

    def bulk_set_attrs(self, name, party, state):
        self.name = name
        self.party = party
        self.state = state
        self.has_attrs = True
        return self

# define graph object to serve as a toy database
class Graph:
    def __init__(self):
        self.nodes = {}
        self.edges = []

# start the graph
G = Graph()
```

The edge list generated in Appendix D was saved to a csv file called “fncosponsors.csv”. This then accessed and a list of raw edges created. The script then iterates through the edge list and does threw operations: 1) if the source node is not in the node dictionary, it adds the node to the node dictionary, along with the node attributes assigned in the edge list. 2) if the target node is not in the node dictionary, the target node and attributes are added. 3) the edge is added to the edge list using a reference to the source and target nodes in the node dictionary. 

```python
# open the edge list and create a list of edges
with open('fncosponsors.csv') as edge_list:
    edges = [edge.replace('\n', '').split(',') for edge in edge_list]

# iterate through the edge list and add edges to our dictionary. add nodes to our node list if they do not already exist, with attributes
for edge in edges[1:]:
    if edge[0] not in G.nodes:
        G.nodes[edge[0]] = Node(edge[0]).bulk_set_attrs(name=edge[1], party=edge[2], state=edge[3])
    if edge[4] not in G.nodes:
        G.nodes[edge[4]] = Node(edge[4]).bulk_set_attrs(name=edge[5], party=edge[6], state=edge[7])
    source = G.nodes[edge[0]]
    target = G.nodes[edge[4]]
    G.edges.append((source, target, edge[8]))

The toy model can now be queried. The first query asks for the largest edge, indicating the largest number of co-sponsorships.

# ask the graph object which edge is the largest, and print a response
result = sorted(G.edges, key=lambda x: x[2], reverse=True)[0]
print(f'{result[0].name} ({result[0].party[:1]}-{result[0].state}) and {result[1].name} ({result[1].party[:1]}-{result[1].state}) shared {result[2]} cosponsors')
print('-' * 70)
```
```
Output:
Joseph I. Lieberman (D-CT) and Susan Collins (R-ME) shared 6 cosponsors
----------------------------------------------------------------------
```

The next query asks for a list of all Senators that cosponsored a bill with Senator Lieberman.

```python
# ask the graph object for a list of who else cosponsored a bill with Senator Lieberman
result = [y[1] for y in filter(lambda x: x[0].name == 'Joseph I. Lieberman', G.edges)] + [y[0] for y in filter(lambda x: x[1].id == 'S210', G.edges)]
print('All Senators that cosponsored a bill with Senator Lieberman')
print([x.name for x in result])
print('-' * 70)
```

```
Output:
All Senators that cosponsored a bill with Senator Lieberman
['Susan Collins', 'Thomas R. Carper', 'Bob Casey', 'Daniel K. Akaka', 'Bill Nelson', 'Jon Tester', 'Mark Pryor', 'Chris Coons', 'Richard J. Durbin', 'Jon Kyl', 'Scott P. Brown', 'Benjamin L. Cardin', 'Jeanne Shaheen', 'Mary L. Landrieu', 'Richard M. Burr', 'Sherrod Brown', 'Amy Klobuchar', 'Mike Lee', 'John Barrasso', 'James M. Inhofe', 'Johnny Isakson', 'Bob Menendez', 'Jeff Sessions', 'Kay Bailey Hutchison', 'Debbie Stabenow', 'Dianne Feinstein', 'Al Franken', 'Marco Rubio', 'Lindsey Graham', 'Pat Roberts', 'Michael B. Enzi', 'John Cornyn', 'Michael Bennet', 'Claire McCaskill', 'David Vitter', 'Mark Begich', 'Kay Hagan', 'Lamar Alexander', 'Charles E. Grassley', 'Carl Levin', 'Tom Harkin', 'Patrick J. Leahy', 'John McCain', 'Richard G. Lugar', 'Orrin G. Hatch', 'Barbara A. Mikulski']
----------------------------------------------------------------------
```

To compute a metrics, the graph is converted to a networkx object to save time reimplementing network algorithms. This is done by creating modified lists out of the node and edge objects and passing them to networkx. Note that networkx can use custom defined objects, like the Node class, as nodes.

```python
# compute node statistics using networkx
import networkx as nx

# convert our toy model into a networkx graph
Gnx = nx.Graph()
Gnx.add_edges_from([(s, t, {'cosponsors': int(v)}) for s, t, v in G.edges])
Gnx.add_nodes_from([node for node in G.nodes.values()])
G = Gnx

Betweenness centrality can then be computed using the convenience method in networkx. Two examples show how to find the node with the highest betweenness centrality and how to calculate the average centrality by party.

# which node is the most central (betweenness)?
cent = nx.betweenness_centrality(G, normalized=True)
cent = ([(i, v) for i, v in cent.items() if not isinstance(i, tuple)])
cent.sort(reverse=True, key=lambda x: x[1])
print(f'Node {cent[0][0].id}, {cent[0][0].name} ({cent[0][0].party[:1]}-{cent[0][0].state}) has the highest betweenness centrality of {cent[0][1]}')
print('-' * 70)

# which party had the highest average betweenness centrality?
party = ([(i.party, v) for i, v in cent])
for p in ('Democrat', 'Republican', 'Independent'):
    tmp = filter(lambda x: x[0] == p, party)
    tmp = [v for i, v in tmp]
    print(f'The average centrality of {p} is {round(sum(tmp) / len(tmp), 4)}')
print('-' * 70)
```

```
Output:
Node S307, Sherrod Brown (D-OH) has the highest betweenness centrality of 0.043802194470251514
----------------------------------------------------------------------
The average centrality of Democrat is 0.0143
The average centrality of Republican is 0.0046
The average centrality of Independent is 0.0052
----------------------------------------------------------------------
```
 
## Visualizing a Network
This example shows how to visualize a network stored as a csv edgelist. The same edge list from Appendix D is used, but it Is ingested directly to networkx instead of passing through a toy model. Most of the code below related to the visual 

```python
# Load edge list from csv and
# visualize the network using Bokeh
import pandas as pd
import networkx as nx
from bokeh.plotting import figure, from_networkx
from bokeh.models import (Circle, MultiLine)
from bokeh.io import show
from bokeh.palettes import Spectral11, Greys3
from bokeh.models import HoverTool, TapTool, EdgesAndLinkedNodes, NodesAndLinkedEdges, NodesOnly

# inititalize plot
plot = figure(x_range=(-4, 4), y_range=(-4, 4), toolbar_location="right", sizing_mode="scale_both")

# remove grid lines and axis
plot.xgrid.grid_line_color = None
plot.ygrid.grid_line_color = None
plot.axis.visible = False
```

Node and edge attributes are calculated before creating the graph, this is author preference

```python
# initialize networkx graph from csv edgelist
df = pd.read_csv('fncosponsors.csv')
df.columns = ['source', 'source_name', 'source_party', 'source_state', 'target', 'target_name', 'target_party', 'target_state', 'cosponsors']
df.fillna('', inplace=True)
df['edge_fill'] = df['cosponsors'].apply(lambda w: ((w - df['cosponsors'].min()) / (df['cosponsors'].max() - df['cosponsors'].min()) + .05))
G = nx.from_pandas_edgelist(df, 'source', 'target', ['cosponsors', 'edge_fill'], nx.Graph)

# tooltips
node_hover = HoverTool(line_policy='interp', tooltips=[("Name", "@name"), ('Party', '@party'), ('State', '@state')])
plot.add_tools(node_hover, TapTool())

Dictionaries can be created that map attributes to colors. Load these into the network as additional attributes to designate styling by node.

# create custom color mappings
party_map = {'Republican': Spectral11[9], 'Democrat': Spectral11[1], 'Independent': Spectral11[0]}
```

Node attributes are set using a loop over the edge list. The source node and target node data are validated and appended. This implementation assumes that every node attribute instance is identical.

```python
# set node attributes
node_atrs = {}
for i, row in df.iterrows():
    if row['source_party'] != '':
        node_atrs[row['source']] = {'name': row['source_name'], 'party': row['source_party'], 'state': row['source_state'], 'node_color': party_map[row['source_party']]}
    if row['target_party'] != '':
        node_atrs[row['target']] = {'name': row['target_name'], 'party': row['target_party'], 'state': row['target_state'], 'node_color': party_map[row['target_party']]}
nx.set_node_attributes(G, node_atrs)

# create layout - customization and interactive functions
layout = from_networkx(G, nx.spring_layout, scale=1, center=(0, 0), weight='cosponsors')
layout.node_renderer.glyph = Circle(size=15, fill_color='node_color')
layout.node_renderer.selection_glyph = Circle(size=15, fill_color='node_color')
layout.node_renderer.hover_glyph = Circle(size=15, fill_color=Spectral11[7])
layout.edge_renderer.glyph = MultiLine(line_color=Greys3[0], line_alpha='edge_fill', line_width=3)
layout.edge_renderer.selection_glyph = MultiLine(line_color=Spectral11[7], line_width=3)
layout.edge_renderer.hover_glyph = MultiLine(line_color=Spectral11[6], line_width=3)
layout.selection_policy = NodesAndLinkedEdges()
layout.inspection_policy = NodesOnly()
plot.renderers.append(layout)

show(plot)
```
