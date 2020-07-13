---
title: Current Implementation of Universe Pickling
published: true
classes: wide
---

{% capture fig_img %}
![Foo]({{ "/assets/images/universe_2.jpg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Photo by Billy Huyn.</figcaption>
</figure>

To achieve a flawless implementation of parallelism, in the first stage of GSoC project,
we aimed at `Universe` serialization.

`Universe` is serialized by serializing each necessary compartment---
`_topology`, `_trajectory`, `anchor_name`; a new `Universe` is
reconstructed with these elements. 

Here is how a `Universe` can be (un)pickled with its current state saved:

```python
    u = MDAnalysis.Universe(topology, trajectory)
    u.trajectory[5]
    u_pickled = pickle.loads(pickle.dumps(u))
    u_pickled.ts.frame == u.trajectory.ts.frame == 5
```

With a picklable `Universe` at hand, we can delve into parallel analysis.
This is a very simple example:

```python
    def cog(u, ag, frame_id):
        u.trajectory[frame_id]
        return ag.center_of_geometry()

    u = MDAnalysis.Universe(GRO, XTC)
    ag = u.atoms[2:5]

    p = multiprocessing.Pool(2)
    result = np.array([p.apply(cog, args=(u, ag, i))
                       for i in range(u.n_frames)])
    p.close()
```

Note to make sure `Atomgroup` finds its anchored `Universe` after pickling,
the `Universe` has to be pickled first.

Below is some notes for the developers to implement serialization to the new readers in the future.

To make sure every Trajectory reader can be successfully
serialized, we implement picklable I/O classes (see :ref:`implemented-fileio`).
When the file is pickled, filename and other necessary attributes of the open 
file handle are saved. On unpickling, the file is opened by filename.
This means that for a successful unpickle, the original file still has to
be accessible with its filename. To retain the current frame of the trajectory,
`_read_frame(previous frame)` will be called during unpickling.

- File Access

If the new reader uses `util.anyopen()` (e.g. PDB), the reading handler
can be pickled without modification.
If the new reader uses I/O classes from other package (e.g. GSD), and cannot
be pickled natively, create a new picklable class inherited from 
the file class in that package (e.g. GSDPicklable), adding `__getstate__`,
`__setstate__` functions to allow file handler serialization.

- To seek or not to seek

Some I/O class supports `seek` and `tell` functions to allow the file 
to be pickled with an offset. It is normally not needed for MDAnalysis with
random access. But if error occurs, find a way to make the offset work.

- Miscellaneous

If pickle still fails due to some unpicklable attributes, try to find a way
to pickle those, or write custom `__getstate__` and `__setstate__`
methods for the reader.

If the new reader is written in Cython, read `lib.formats.libmdaxdr` and
`lib.formats.libdcd` files as references.

- Tests

If the test for the new reader uses `BaseReaderTest`, whether
the current timestep information is saved, and whether its relative
position is maintained, i.e. next() read the right next timestep,
are already tested.

If the new reader accesses the file with `util.anyopen`, add necessary
testes inside ``parallelism/test_multiprocessing.py`` for the reader.

If the new reader accessed the file with new picklable I/O class,
add necessary tests inside ``utils/test_pickleio.py`` for I/O class,
``parallelism/test_multiprocessing.py`` for the reader.
