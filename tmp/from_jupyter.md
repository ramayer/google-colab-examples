```python
import pyspark

MAX_MEMORY = "8g"  # 24 gives OOM here.

spark = (pyspark.sql.SparkSession.builder.appName("MyApp") 
    .config("spark.jars.packages", "io.delta:delta-core_2.12:0.8.0") 
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") 
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") 
    .config("spark.executor.memory", MAX_MEMORY) 
    .config("spark.driver.memory", MAX_MEMORY) 
    .enableHiveSupport() 
    .getOrCreate()        
    )
spark
```





    <div>
        <p><b>SparkSession - hive</b></p>

<div>
    <p><b>SparkContext</b></p>

    <p><a href="http://192.168.0.71:4041">Spark UI</a></p>

    <dl>
      <dt>Version</dt>
        <dd><code>v3.1.1</code></dd>
      <dt>Master</dt>
        <dd><code>local[*]</code></dd>
      <dt>AppName</dt>
        <dd><code>MyApp</code></dd>
    </dl>
</div>

    </div>





```python
import subprocess
subprocess.run("wget -nc https://datasets.imdbws.com/name.basics.tsv.gz",shell=True)
#dbutils.fs.mkdirs('/tmp/imdb/')
#dbutils.fs.cp('file:///databricks/driver/name.basics.tsv.gz','dbfs:/FileStore/imdb/name.basics.tsv.gz')

df = spark.read.option("delimiter", "\t").option('header',True).csv('name.basics.tsv.gz')
display(df.limit(5).toPandas())
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primaryName</th>
      <th>birthYear</th>
      <th>deathYear</th>
      <th>primaryProfession</th>
      <th>knownForTitles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0000001</td>
      <td>Fred Astaire</td>
      <td>1899</td>
      <td>1987</td>
      <td>soundtrack,actor,miscellaneous</td>
      <td>tt0050419,tt0053137,tt0072308,tt0031983</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm0000002</td>
      <td>Lauren Bacall</td>
      <td>1924</td>
      <td>2014</td>
      <td>actress,soundtrack</td>
      <td>tt0037382,tt0117057,tt0075213,tt0038355</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm0000003</td>
      <td>Brigitte Bardot</td>
      <td>1934</td>
      <td>\N</td>
      <td>actress,soundtrack,music_department</td>
      <td>tt0054452,tt0057345,tt0049189,tt0056404</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm0000004</td>
      <td>John Belushi</td>
      <td>1949</td>
      <td>1982</td>
      <td>actor,soundtrack,writer</td>
      <td>tt0080455,tt0072562,tt0077975,tt0078723</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0000005</td>
      <td>Ingmar Bergman</td>
      <td>1918</td>
      <td>2007</td>
      <td>writer,director,actor</td>
      <td>tt0050976,tt0050986,tt0060827,tt0069467</td>
    </tr>
  </tbody>
</table>
</div>



```python
%pip install --upgrade pip 
%pip install -q ordered_set
```

    Requirement already satisfied: pip in /home/ron/.python/venvs/3.8/lib/python3.8/site-packages (21.0.1)
    Note: you may need to restart the kernel to use updated packages.
    Note: you may need to restart the kernel to use updated packages.



```python
import re
import colorsys
import random
import json
from ordered_set import OrderedSet

node_colors = {}

def randcolor():
  rgb = colorsys.hls_to_rgb(random.random(),0.8,random.random()*.2 + .8)
  return "#" + ''.join([f'{int(255*x):02x}' for x in rgb])

def get_color_for_node(node):
  if node_colors.get(node): return node_colors.get(node);
  color = randcolor()
  node_colors[node] = color
  return color

def show_linkchart(dots,engine='neato'):
    jscript = '''
    var graphviz = d3.select("#graph").graphviz()
        .width("100%")
        .height(600)
        .zoomScaleExtent([0.01,100])
        .fit(1)
        .engine(engine).transition(function() {{
            return d3.transition()
                .delay(100)
                .duration(1000);
        }}).renderDot(d0);

    function render() {
      var dot = dots[dotIndex];
      graphviz
        .renderDot(dot)
        .on("end", function () {
            dotIndex = (dotIndex + 1) % dots.length;
        });
   }
  '''
    h=f'''<!DOCTYPE html>
<meta charset="utf-8">
<body>
<script src="https://d3js.org/d3.v5.min.js"></script>
<script src="https://unpkg.com/@hpcc-js/wasm@0.3.11/dist/index.min.js"></script>
<script src="https://unpkg.com/d3-graphviz@3.0.5/build/d3-graphviz.js"></script>
<button onClick="dotIndex=0;console.log(graphviz); graphviz.resetZoom();render()">less</button>
<button onClick="render()">more</button>
<div id="graph" style="text-align: center; width:100%; height:604px; border: 1px solid #dddddd;"></div>
<script>
var dots = {dots};
var engine = '{engine}';
var dotIndex = 1;
d0 = dots[0];
{jscript}
</script>
'''
    return h
  #with open("/dbfs/FileStore/ramayer/tmp/1.html",'w') as f: f.write(h)

#nodes,edges = query_links('alvin lovett')
#links = query_links('anthony reed')
#links = query_by_sql("select q,u from tmp_flattened_interesting_search_terms where q='anthony reed'")
#dot = to_dot(links)
#print(dot)



#l0 = query_by_sql('''select q,u from tmp_flattened_interesting_search_terms where q = 'alvin lovett' limit 3''')
#l1 = query_by_sql('''select q,u from tmp_flattened_interesting_search_terms where q = 'alvin lovett' limit 5''')
#l2 = query_by_sql('''select q,u from tmp_flattened_interesting_search_terms where q = 'alvin lovett' limit 99''')
#displayHTML(show_linkchart2([l0,l1,l2],engine='neato'))

```


```python
import math
import graphviz
import itertools
def make_dot(n):
  nodes = [i for i in range(n)]
  random.seed(0) 
  edges = [(random.randint(0, int(n/10)),n) for n in range(n)]
  d = graphviz.Digraph(filename='rank_same.gv')
  d.attr(rankdir='LR', size='8,5',splines="spline",ranksep="0.25",overlap="prism",nodesep="0.25")
  #	graph [ranksep=0.25, overlap=prism, nodesep=0.25, splines=true];
  #  node [style="filled"; dir="none"];
  
  edges_out = {k:len(list(g)) for k,g in itertools.groupby(sorted(edges,key=lambda x:x[0]),lambda x:x[0])}
  edges_in  = {k:len(list(g)) for k,g in itertools.groupby(sorted(edges,key=lambda x:x[1]),lambda x:x[1])}
  for n in nodes:
    d.node(str(n),shape="octagon", style='filled', color=get_color_for_node(n))
  for k,g in itertools.groupby(sorted(edges),lambda x:x[0]):
    edges_in_group = [x for x in g]
    sq = math.ceil(math.sqrt(len(edges_in_group)))
    
    #sq = len(edges_in_group)
    #sq=10
    for e in edges_in_group:
      busyness = min(edges_in.get(e[0],0) + edges_out.get(e[0],0), edges_in.get(e[1],0) + edges_out.get(e[1],0))
      sq = busyness
      if random.random() > 0.5:
        d.edge(str(e[0]), str(e[1]),dir="none",len=str(random.randint(1,sq)),minlen=str(random.randint(1,sq)))
      else:
        d.edge(str(e[1]), str(e[0]),dir="none",len=str(random.randint(1,sq)),minlen=str(random.randint(1,sq)))
      
  #d = d.unflatten(5,4,5)
  return str(d)

dots = [make_dot(n*5) for n in range(1,5)]

```


```python
# this fails
# HTML(src_html)

from IPython.core.display import HTML
import base64

src_html = show_linkchart(dots,engine='dot')
b64 = base64.b64encode(src_html.encode('utf-8'))
src = f"data:text/html;base64,{b64.decode('utf-8')}"
HTML(f'<iframe style="width:100%;height:600px" src="{src}">link</a>')



```




<iframe style="width:100%;height:600px" src="data:text/html;base64,PCFET0NUWVBFIGh0bWw+CjxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KPGJvZHk+CjxzY3JpcHQgc3JjPSJodHRwczovL2QzanMub3JnL2QzLnY1Lm1pbi5qcyI+PC9zY3JpcHQ+CjxzY3JpcHQgc3JjPSJodHRwczovL3VucGtnLmNvbS9AaHBjYy1qcy93YXNtQDAuMy4xMS9kaXN0L2luZGV4Lm1pbi5qcyI+PC9zY3JpcHQ+CjxzY3JpcHQgc3JjPSJodHRwczovL3VucGtnLmNvbS9kMy1ncmFwaHZpekAzLjAuNS9idWlsZC9kMy1ncmFwaHZpei5qcyI+PC9zY3JpcHQ+CjxidXR0b24gb25DbGljaz0iZG90SW5kZXg9MDtjb25zb2xlLmxvZyhncmFwaHZpeik7IGdyYXBodml6LnJlc2V0Wm9vbSgpO3JlbmRlcigpIj5sZXNzPC9idXR0b24+CjxidXR0b24gb25DbGljaz0icmVuZGVyKCkiPm1vcmU8L2J1dHRvbj4KPGRpdiBpZD0iZ3JhcGgiIHN0eWxlPSJ0ZXh0LWFsaWduOiBjZW50ZXI7IHdpZHRoOjEwMCU7IGhlaWdodDo2MDRweDsgYm9yZGVyOiAxcHggc29saWQgI2RkZGRkZDsiPjwvZGl2Pgo8c2NyaXB0Pgp2YXIgZG90cyA9IFsnZGlncmFwaCB7XG5cdG5vZGVzZXA9MC4yNSBvdmVybGFwPXByaXNtIHJhbmtkaXI9TFIgcmFua3NlcD0wLjI1IHNpemU9IjgsNSIgc3BsaW5lcz1zcGxpbmVcblx0MCBbY29sb3I9IiNmZWQwOTkiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxIFtjb2xvcj0iI2ZlOTllNSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDIgW2NvbG9yPSIjYTBmN2YyIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0MyBbY29sb3I9IiNiN2Y2YTEiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ0IFtjb2xvcj0iIzlhZmU5OSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDAgLT4gMCBbZGlyPW5vbmUgbGVuPTMgbWlubGVuPTRdXG5cdDEgLT4gMCBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDIgLT4gMCBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDAgLT4gMyBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDQgLT4gMCBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG59JywgJ2RpZ3JhcGgge1xuXHRub2Rlc2VwPTAuMjUgb3ZlcmxhcD1wcmlzbSByYW5rZGlyPUxSIHJhbmtzZXA9MC4yNSBzaXplPSI4LDUiIHNwbGluZXM9c3BsaW5lXG5cdDAgW2NvbG9yPSIjZmVkMDk5IiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0MSBbY29sb3I9IiNmZTk5ZTUiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQyIFtjb2xvcj0iI2EwZjdmMiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDMgW2NvbG9yPSIjYjdmNmExIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0NCBbY29sb3I9IiM5YWZlOTkiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ1IFtjb2xvcj0iI2U3ZmM5YiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDYgW2NvbG9yPSIjYTBkZWY3IiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0NyBbY29sb3I9IiNmY2QwOWIiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ4IFtjb2xvcj0iI2E5ZjdhMCIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDkgW2NvbG9yPSIjZjNmZDlhIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0MCAtPiAwIFtkaXI9bm9uZSBsZW49NSBtaW5sZW49M11cblx0MCAtPiAxIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MiAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MyAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0NCAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0NSAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA2IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA3IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA4IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA5IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cbn0nLCAnZGlncmFwaCB7XG5cdG5vZGVzZXA9MC4yNSBvdmVybGFwPXByaXNtIHJhbmtkaXI9TFIgcmFua3NlcD0wLjI1IHNpemU9IjgsNSIgc3BsaW5lcz1zcGxpbmVcblx0MCBbY29sb3I9IiNmZWQwOTkiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxIFtjb2xvcj0iI2ZlOTllNSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDIgW2NvbG9yPSIjYTBmN2YyIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0MyBbY29sb3I9IiNiN2Y2YTEiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ0IFtjb2xvcj0iIzlhZmU5OSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDUgW2NvbG9yPSIjZTdmYzliIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0NiBbY29sb3I9IiNhMGRlZjciIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ3IFtjb2xvcj0iI2ZjZDA5YiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDggW2NvbG9yPSIjYTlmN2EwIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0OSBbY29sb3I9IiNmM2ZkOWEiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxMCBbY29sb3I9IiNmZWFkOTkiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxMSBbY29sb3I9IiNjNmY1YTIiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxMiBbY29sb3I9IiNhN2ExZjYiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxMyBbY29sb3I9IiNmNWVjYTIiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQxNCBbY29sb3I9IiNmYmNhOWMiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQwIC0+IDAgW2Rpcj1ub25lIGxlbj05IG1pbmxlbj0xMl1cblx0MCAtPiAxIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49Ml1cblx0MiAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MyAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0NCAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0NSAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA2IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA3IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA4IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiA5IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiAxMCBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDAgLT4gMTIgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQxMyAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MTEgLT4gMSBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDEgLT4gMTQgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxufScsICdkaWdyYXBoIHtcblx0bm9kZXNlcD0wLjI1IG92ZXJsYXA9cHJpc20gcmFua2Rpcj1MUiByYW5rc2VwPTAuMjUgc2l6ZT0iOCw1IiBzcGxpbmVzPXNwbGluZVxuXHQwIFtjb2xvcj0iI2ZlZDA5OSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDEgW2NvbG9yPSIjZmU5OWU1IiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0MiBbY29sb3I9IiNhMGY3ZjIiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQzIFtjb2xvcj0iI2I3ZjZhMSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDQgW2NvbG9yPSIjOWFmZTk5IiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0NSBbY29sb3I9IiNlN2ZjOWIiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ2IFtjb2xvcj0iI2EwZGVmNyIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDcgW2NvbG9yPSIjZmNkMDliIiBzaGFwZT1vY3RhZ29uIHN0eWxlPWZpbGxlZF1cblx0OCBbY29sb3I9IiNhOWY3YTAiIHNoYXBlPW9jdGFnb24gc3R5bGU9ZmlsbGVkXVxuXHQ5IFtjb2xvcj0iI2YzZmQ5YSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDEwIFtjb2xvcj0iI2ZlYWQ5OSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDExIFtjb2xvcj0iI2M2ZjVhMiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDEyIFtjb2xvcj0iI2E3YTFmNiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDEzIFtjb2xvcj0iI2Y1ZWNhMiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE0IFtjb2xvcj0iI2ZiY2E5YyIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE1IFtjb2xvcj0iI2Y5OWNmYiIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE2IFtjb2xvcj0iI2ZhOWRmNyIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE3IFtjb2xvcj0iI2EwZWJmNyIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE4IFtjb2xvcj0iI2RmZmI5YyIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDE5IFtjb2xvcj0iI2Y0OWVmOSIgc2hhcGU9b2N0YWdvbiBzdHlsZT1maWxsZWRdXG5cdDAgLT4gMCBbZGlyPW5vbmUgbGVuPTQgbWlubGVuPTEyXVxuXHQxIC0+IDAgW2Rpcj1ub25lIGxlbj01IG1pbmxlbj0yXVxuXHQwIC0+IDIgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDMgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDQgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDUgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDYgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDcgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDggW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQwIC0+IDkgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQxMCAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MTIgLT4gMCBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDAgLT4gMTMgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQxNSAtPiAwIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MCAtPiAxNyBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDE4IC0+IDAgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQxIC0+IDExIFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cblx0MTQgLT4gMSBbZGlyPW5vbmUgbGVuPTEgbWlubGVuPTFdXG5cdDEgLT4gMTYgW2Rpcj1ub25lIGxlbj0xIG1pbmxlbj0xXVxuXHQxIC0+IDE5IFtkaXI9bm9uZSBsZW49MSBtaW5sZW49MV1cbn0nXTsKdmFyIGVuZ2luZSA9ICdkb3QnOwp2YXIgZG90SW5kZXggPSAxOwpkMCA9IGRvdHNbMF07CgogICAgdmFyIGdyYXBodml6ID0gZDMuc2VsZWN0KCIjZ3JhcGgiKS5ncmFwaHZpeigpCiAgICAgICAgLndpZHRoKCIxMDAlIikKICAgICAgICAuaGVpZ2h0KDYwMCkKICAgICAgICAuem9vbVNjYWxlRXh0ZW50KFswLjAxLDEwMF0pCiAgICAgICAgLmZpdCgxKQogICAgICAgIC5lbmdpbmUoZW5naW5lKS50cmFuc2l0aW9uKGZ1bmN0aW9uKCkge3sKICAgICAgICAgICAgcmV0dXJuIGQzLnRyYW5zaXRpb24oKQogICAgICAgICAgICAgICAgLmRlbGF5KDEwMCkKICAgICAgICAgICAgICAgIC5kdXJhdGlvbigxMDAwKTsKICAgICAgICB9fSkucmVuZGVyRG90KGQwKTsKCiAgICBmdW5jdGlvbiByZW5kZXIoKSB7CiAgICAgIHZhciBkb3QgPSBkb3RzW2RvdEluZGV4XTsKICAgICAgZ3JhcGh2aXoKICAgICAgICAucmVuZGVyRG90KGRvdCkKICAgICAgICAub24oImVuZCIsIGZ1bmN0aW9uICgpIHsKICAgICAgICAgICAgZG90SW5kZXggPSAoZG90SW5kZXggKyAxKSAlIGRvdHMubGVuZ3RoOwogICAgICAgIH0pOwogICB9CiAgCjwvc2NyaXB0Pgo=">link</a>




```python

```


```python

```


```python

```
