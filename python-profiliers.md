---
description: Checking the performance of python scripts
---

# Python profiliers

Install GraphViz: [http://www.graphviz.org/download/](http://www.graphviz.org/download/)

Install Grof2dot:: [https://github.com/jrfonseca/gprof2dot](https://github.com/jrfonseca/gprof2dot)

Run profile on the code.

```python
python -m cProfile -o myLog.profile <myScript.py> arg1 arg2 ...
```

Run `gprof2dot` to convert the call profile into a dot file

```python
gprof2dot -f pstats myLog.profile -o callingGraph.dot
```

Open with graphViz to visualize the graph

![Example graph image](.gitbook/assets/image%20%283%29.png)

