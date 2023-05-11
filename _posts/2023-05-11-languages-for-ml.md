---
title: "Language(s) for ML?"
categories:
  - Pondering
tags:
  - Machine learning
  - Programming languages
---

I've recently been trying to help my son, Patrick, with a programming task. He's working on a numerical simulation: for the purposes of this post, think of it as simulating the motion of particles in a box.

He's naturally starting with what he knows: Python with `numpy` for vectors and matrix operations. It turns out this is close to the standard toolkit for machine learning (Python + PyTorch or TensorFlow). Python is great for quickly writing code to implement his ideas. Now he's ready to go from prototyping to running larger simulations, and the question I've been pondering is how far it makes sense to go with pure Python, and what alternatives are worth exploring.

![100% Python](/assets/images/100percent-python.png)

We've looked at various ways to speed up pure Python code:
- [PyPy](https://www.pypy.org/) can make a lot of pure Python code faster with minimal effort for the developer. Unfortunately, the integration with `numpy` isn't seamless (see their [faq](https://doc.pypy.org/en/latest/faq.html#what-about-numpy-numpypy-micronumpy) for details), and for our case the recommended approach made the simulation slower rather than faster. 
- The Python `multiprocessing` module makes it quite easy to distribute tasks across a pool of worker processes. The simulation is [embarrassingly parallelizable](https://en.wikipedia.org/wiki/Embarrassingly_parallel), so this scales up well to the number of CPU cores available. Developers need to be wary of global variables and side effects to get maximum benefit, and there is a communication / startup cost of creating the pool, but on the whole `multiprocessing` has worked well.
- I tried [joblib](https://joblib.readthedocs.io/en/latest/) as an alternative to `multiprocessing`. One thing that appealed to me is running multithreaded instead of multiprocess with a simple change to the backend. Unfortunately, it was slower for us in both configurations: there is a large array being passed to the workers, which is documented as [up to 100x slower](https://joblib.readthedocs.io/en/latest/parallel.html#serialization-processes) for multiprocessing. And with the `threading` backend, we hit [bottlenecks with the GIL](https://wiki.python.org/moin/GlobalInterpreterLock).
- [Numba](https://numba.pydata.org/) promises to make Python code fast just by sprinkling a few `@njit` decorators through the code. This looks like exactly what we wanted, and for simple cases it did provide some speedups, but things [got complicated fast](https://numba.readthedocs.io/en/stable/user/performance-tips.html), and we didn't make it to the bottom of the rabbit hole.

There are obviously many more techniques possible with pure Python code. For example, Patrick spent some time getting his code running on CUDA, but since it isn't pure linear algebra, the results weren't great.

![ML with mixed languages](/assets/images/ml-mixed-languages.png)

This led me to look at some real ML projects to understand how they are structured. The above breakdown is from the strongest open source Go engine I've found, [KataGo](https://github.com/lightvector/KataGo) (which I'll have more to say about in future posts). Here Python is used for training and to orchestrate multiple interacting processes, and the main application is written in fast C++ to make use of the model(s), even on humble laptops with no dedicated hardware to accelerate ML.

Unfortunately, this doesn't help Patrick much: he's a mathetmatics major who has never taken a computer science unit, so writing another implementation of his simulation in C++ is out of the question for now (and I'm not volunteering!).

Here are some other possibilities I'm continuing to explore:
- the problem statement for the [Mojo programming language](https://www.modular.com/mojo) looks like exactly what we want: 
  > Mojo combines the usability of Python with the performance of C, unlocking unparalleled programmability of AI hardware and extensibility of AI models.
  
  It's a superset of Python that can exploit type information and has a vectorized backend built on LLVM's MLIR. Unfortunately, it's limited in various ways and currently only available in a walled garden.
- [Cython](https://cython.org/) can translate Python to C/C++ code that can be compiled and used as a module. It works best if types are converted to a Cython-specific format, which unfortunately makes it difficult to experiment with.
- [Pythran](https://pythran.readthedocs.io/) is an ahead of time compiler for a subset of Python, meant to efficiently compile scientific programs. We haven't yet found a subset of Patrick's code that Pythran can understand.
- The [Nim programming language](https://nim-lang.org/) has some features I like (as does [Zig](https://ziglang.org/), for that matter), but it would be a stretch coming from Python and the array handling doesn't seem complete.
- The most promising candidate for now looks like the [Julia Programming Language](https://julialang.org/). It has a [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) and dynamic typing, and baked in support for functional programming over multi-dimensional arrays.

### Why does any of this matter?

I think what Patrick is facing now is a microcosm of the challenges facing many ML projects: how to go from a working prototype to a production-ready implementation. It seems important to have a "good enough" solution that doesn't require every ML application to be implemented twice (often by programmers with disjoint skill sets).