---
title: How Many Stars Are There in The Universe?
published: true
classes: wide
---

{% capture fig_img %}
![Foo]({{ "/assets/images/universe.jpg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Photo by Snapwire.</figcaption>
</figure>

The `Universe` is the fundamental data structure of MDAnalysis. It contains all the topology and trajectory data of a simulation system. Normally, a `Universe` can be created from files:

```python
universe  = mda.Universe(topology, trajectory)
```

where `topology` must be specified to access 'atom'-wise information of the system; `trajectory` , on the other hand, can be either loaded from a trajectory file, deduced from `topology` (e.g. when it is a pdb file), or absent at all. If `in_memory = True` , the trajectory will be loaded directly into memory with `MemoryReader`, while in default, it will be parsed as a corresponding reader for different types of trajectories.

Let's have a look at the stars in a `Universe` after it is created.

```python
>>> u = mda.Universe(GRO, XTC)
>>> print(u.__dict__)

{'_cache': {},
 '_anchor_name': None,
 '_anchor_uuid': UUID('a1b4e353-aeaf-4830-8de7-cadadcfc35da'),
 'atoms': <AtomGroup with 47681 atoms>,
 'residues': <ResidueGroup with 11302 residues>,
 'segments': <SegmentGroup with 1 segment>,
 'filename': 'data/adk_oplsaa.gro',
 '_kwargs': {'transformations': None,
  'guess_bonds': False,
  'vdwradii': None,
  'anchor_name': None,
  'is_anchor': True,
  'in_memory': False,
  'in_memory_step': 1,
  'format': None,
  'topology_format': None,
  'all_coordinates': False},
 '_topology': <MDAnalysis.core.topology.Topology at 0x7febf011a690>,
 '_class_bases': {MDAnalysis.core.groups.GroupBase: MDAnalysis.core.
groups._TopologyAttrContainer,
  MDAnalysis.core.groups.AtomGroup: MDAnalysis.core.groups.
_TopologyAttrContainer,
  MDAnalysis.core.groups.ResidueGroup: MDAnalysis.core.groups.
_TopologyAttrContainer,
  MDAnalysis.core.groups.SegmentGroup: MDAnalysis.core.groups.
_TopologyAttrContainer,
  MDAnalysis.core.groups.ComponentBase: MDAnalysis.core.groups.
_TopologyAttrContainer,
MDAnalysis.core.groups.Atom: MDAnalysis.core.groups._TopologyAttrContainer,
MDAnalysis.core.groups.Residue: MDAnalysis.core.groups._TopologyAttrContainer,
MDAnalysis.core.groups.Segment: MDAnalysis.core.groups._TopologyAttrContainer},
'_classes': {MDAnalysis.core.groups.AtomGroup: 
MDAnalysis.core.groups.AtomGroup,
MDAnalysis.core.groups.ResidueGroup: MDAnalysis.core.groups.ResidueGroup,
MDAnalysis.core.groups.SegmentGroup: MDAnalysis.core.groups.SegmentGroup,
MDAnalysis.core.groups.Atom: MDAnalysis.core.groups.Atom,
MDAnalysis.core.groups.Residue: MDAnalysis.core.groups.Residue,
MDAnalysis.core.groups.Segment: MDAnalysis.core.groups.Segment},
 '_trajectory': <XTCReader data/adk_oplsaa.xtc with 10 frames of 47681 atoms>}
```

`__dict__` is an important attribute in terms of pickling. As we will cover more in the following **parallelism** blog, an object in python has to be pickled/unpickled when it is transferred across different threads. In default, some normal data types can be easily pickled (or serialized), e.g. integers, strings, lists, etc.; while for user-defined classes, instances of such classes whose `__dict__` or the result of calling `__getstate__()` has to be picklable can be pickled.

So, what's the current status of the picklibility of a `Universe` ?

```python
>>> for star in u.__dict__:
>>> try:
>>>     pickle.dumps(u.__dict__[star])
>>>     print('{:20} can be pickled'.format(star))
>>> except:
>>>     print('{:20} cannot be pickled'.format(star))

_cache               can be pickled
_anchor_name         can be pickled
_anchor_uuid         can be pickled
atoms                can be pickled
residues             cannot be pickled
segments             cannot be pickled
filename             can be pickled
_kwargs              can be pickled
_topology            can be pickled
_class_bases         cannot be pickled
_classes             cannot be pickled
_trajectory          can be pickled
```

Looks not bad, huh? We just need to fix a few attributes—if they are fixable—to make `Universe` pickable, but do we really need all these class attributes? In fact, upon pickling, a special function `__reduce__` is being called to declare how an object should be pickled. For our `Universe`, `topology` and `trajectory` might be quite enough for the reconstruction of the same `Universe`. Another caveat is that not all the `trajectory` readers can be pickled in default:

- [can't pickle PDBReader on python 3 #1981](https://github.com/MDAnalysis/mdanalysis/issues/1981)
- [ChainReader cannot be pickled #1940](https://github.com/MDAnalysis/mdanalysis/issues/1940)

Overall, in this project, I will implement the pickling support to create a parallel `Universe` with the same stars, well, for parallel analysis.

## Reference 
- [https://docs.python.org/3/library/pickle.html](https://docs.python.org/3/library/pickle.html)
- [https://www.mdanalysis.org/UserGuide/index.html](https://www.mdanalysis.org/UserGuide/index.html)
