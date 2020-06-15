---
title: Composition Over Inheritance
published: true
classes: wide
---

{% capture fig_img %}
![Foo]({{ "/assets/images/classes.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption><a href="https://drive.google.com/file/d/1LkVBcaEvNITimifbqVtFKXnbDGdOh5gz/view?usp=sharin">Class inheritance of Readers in MDAnalysis</a>.</figcaption>
</figure>

First two weeks of GSoC project have been fun and fruitful. I submitted my first PR, and then I submitted my second...well...it basically teared down the backbone of the former—Composition over Inheritance!

You see, in MDAnalysis, all types of Readers are based on  `ProtoReader`. One of the major problems upon pickling a trajectory is to pickle a trajectory file, which is normally read by `util.anyopen` in the MDAnalysis.lib. This function utilizes either `bz2.open`, `gzip.open` , or `open` as a file handler to read compressed/uncompressed files into a `FileIO` based object, that **cannot be pickled**, which leaves natively pickle a trajectory impossible. 

One of the solutions is to tweak with `__getstate__` and `__setstate__` functions for each trajectory reader—telling the computer to recreate a file handler each time we pickle/unpickle a trajectory. We can give readers such functionality by letting them **inherit** from a pickle-implemented class (PR #[2704](https://github.com/MDAnalysis/mdanalysis/pull/2704)).

The problem is, files are born different—even though they are all bits at the bottom level. Some files are saved in text , while others are saved in binary. We may all have ***vi*** a `gro` file successfully, and got beaten down by all the gibberish when accidentally open an `xtc` file. The binary format gives compressing an astronomical trajectory file some advantages (Behold! A new `TNG`format is coming as one of the GSoC projects this year), but it also leaves unifying pickling process for all these readers improbable (because text and binary are read in different modes). There's more for the future—imagining some day exists a new reader that needs two file inputs. Then we have to create a new inheritance class to accommodate such a new pickle process.

The remedy for that is composition. Object composition is a design pattern that by assembling or composing objects,  new and more complex functionality can be reached. In this case, if we take a step back from the readers and focus on picklability of the file handlers, we can later teach each reader how to pickle themselves. (PR #[2723](https://github.com/MDAnalysis/mdanalysis/pull/2723))

The picklibility of a file handler have been changed back and forth. In python 2, they cannot be pickled; in python 3.2, they can, and was later forbidden explicitly in Issue [10180](https://bugs.python.org/issue10180). The major issue is that back in the day, it gives nonsensical result upon unpickling. Besides, "pickle cannot will the connection for file object to exist when you unpickle your object, and the process of creating that connection goes beyond what pickle can automatically do for you." This is of course, one of the issues we need to consider if we try to run parallel analysis across nodes—making sure every node has access to the file with the same file path. But right now we can just assume it to be holistic, and create a new `FileIO` class that can be pickled. Things went quite well! The whole pickling process becomes much neater, more flexible and easier to adjust.

For the next two weeks before the first evaluation, I will keep up with the [timeline](https://yuxuanzhuang.github.io/Proposed-Timeline-For-Serialization-of-Universes/): write detailed docs and tests for PR #2723 and merge into the origin PR, write comprehensive tests for picklibility, and memory leakage to make sure every reader works in this way, and also conduct some easy parallel analysis to ultilize this new functionality as a showcase.

Thanks all my mentors for their revision on my messy code. In particular, I want to thank @kain88-de for the kind advice and discussion upon this composition over inheritance topic, and all the `git` remedies!

## Git Lesson Learned

If one wants to time machine back to before some specific git command (e.g. you mess up with the PR by `rebase`).

- git reflog                       # find the latest SHA before that command
- git branch backup      # just in case reference to start over.
- git reset --hard <SHA>
- git remote update      # fetch remote changes without local updates
- git merge mda_origin/develop      # or whatever name you give to the branch you want to merge
- git push origin HEAD --force-with-lease

## Reference

- [https://bugs.python.org/issue10180](https://bugs.python.org/issue10180)
- [https://docs.python.org/3/library/io.html](https://docs.python.org/3/library/io.html)
- Design Patterns: Elements of Reusable Object-Oriented Software
