--- 
title: Proposed Timeline for Serialzation of Universes 
published: true
classes: wide
---

{% capture fig_img %}
![Foo]({{ "/assets/images/spacex.jpg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption><a href="https://unsplash.com/@spacex?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Photo by SpaceX on Unsplash</a>.</figcaption>
</figure>

With recent crushing of the exascale barrier, researchers are handling tons of molecular dynamics (MD) data every day. Implementation of parallelism of MDAnalysis plays a pivotal role in accelerating the analysis process in such turnaround. By serializing Universes, the core of MDAnalysis, analysis modules can then be utilized in parallel, and be sent over to a distributed computing framework, e.g. Dask, multiprocessing, or MPI.

I will list the proposed timeline and deliverables below for the GSoC project, Serialization of Universes:

## **June 1, 2020 - June 29, 2020**

- Merger PR 2140, and make sure it works for all the Readers
- Raise quick-merged PR (documents what fails) & new PRs for individual tests (xfail))
- Better structurize the inheritance of pickle functionality
    - Understand lib.format files (cython)
    - Document what works/ not works
- Write comprehensive test for picklibility and memory leakage
- Write 'easy' analysis methods (not AnalysisBase based) with parallel support
    - Be sure no mem leakage
    - Multiprocessing
- Documentation for basic functionality
    - documents what should be done for the new format

## June 29, 2020 - July 3, 2020

- **Evaluation I**
- Deliverables

	```python
	u = mda.Universe(ANY_TOP, ANY_TRAJ)
	pickle.dumps(pickle.loads(u)) == u

	def Analysis_method(u, ts)
	# In joblib/multiprocessing
	result = Parallel(n_jobs=num_cores)(delayed(Analysis_method)(u, ts) 
						for ts in range(0,n_frames))
	```
    - Tests
    - Documentations


## July 3, 2020 - July 27, 2020

- Buffer for some unsolved problems in June
- Code cleanup for the works that have been done
- Add support for on-the-fly transformation/auxiliary
- Prototype Parallelism into AnalysisBase

## July 27, 2020 - July 31, 2020

- **Evaluation II**
- Deliverables
    - On-the-fly transformation test
    - On-the-fly transformation documentation/example
    - Prototype for

        ```python
        Class AnalysisMethod(AnalysisBase)
        # In joblib/multiprocessing
        AnalysisMethod(u, n_jobs=num_cores, *arg).run()
        ```

## July 31, 2020 - August 24, 2020

- Implement AnalysisBase
- Test different distributed clusters (with Dask and joblib)
- Tests and documentation
- Run benchmarks
- Write a blogpost about this new functionality (parallel analysis) and some tutorials
- Minimize the time and mem usage during pickling (Stretch goal)

## August 24, 2020 - August 31, 2020

- **Final evaluation**
