---
title: A Tale of Two Stars 
published: true
classes: wide
---

{% capture fig_img %}
![Foo]({{ "/assets/images/sun_and_proxima.jpg" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Photo by Michele Diodati/Medium.</figcaption>
</figure>

It happened inside a Python universe. Two stars, `sun` and `proxima_centauri`, wanted to hang out
together in a parallel universe.
```python
class Universe(object):
    def __init__(self):
        pass 
        
class Star(object):
    def __init__(self, star_name, universe):
        self.name = star_name
        self._universe = universe

universe = Universe()
sun = Star(star_name='sun',
             universe=universe)
proxima_entauri = Star(star_name='Proxima Centauri',
             universe=universe)
```
First, they tried to pickle themselves seperately. They couldn't find each other.
```python
sun_parallel = pickle.loads(pickle.dumps(sun))
proxima_entauri_parallel = pickle.loads(pickle.dumps(proxima_entauri))

>>>print('universe of parallel sun:', id(sun_parallel._universe))
>>>print('universe of parallel proxima entauri:', id(proxima_entauri_parallel._universe))
universe of parallel sun: 139927156407168
universe of parallel proxima entauri: 139927156407072
```
Then they decided to do it together. They reunited.
```python
sun_parallel, proxima_entauri_parallel = pickle.loads(pickle.dumps([sun, proxima_entauri])) 

>print('universe of parallel sun:', id(sun_parallel._universe))
>print('universe of parallel proxima entauri:', id(proxima_entauri_parallel._universe))
universe of parallel sun: 140287996428144
universe of parallel proxima entauri: 140287996428144
```
It's a true story about `AtomGroup`.

