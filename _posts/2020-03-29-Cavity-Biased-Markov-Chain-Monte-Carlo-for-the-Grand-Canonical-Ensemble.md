---
published: true
images:
  - url: /blog/img/muVT_DensityConvergence.png
---
<style TYPE="text/css">
code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}
</style>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [['$','$'], ['\\(','\\)']],
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'] // removed 'code' entry
    }
});
MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS_HTML-full"></script>

<img alt="Interpolation" src="/blog/img/muVT_DensityConvergence.png"> 

Semi-interactive code for fluid simulations in the μVT ensemble according to Mezei's Cavity-Biased algorithm, which improves insertions in high-density systems.

The environment consists of a fixed-size simulation box centered in the origin, filled with spherical molecules based on the density of the system. Their interaction is according to either a Lennard-Jones or Square-Well potential (implementation of additional potentials is straightforward) in the reduced unit's system.

New states are generated either by insertion, removal or displacement of molecules, according to the probabilities in the upcoming scheme:

![Algorithm](img/Algorithm.png)

Code has been designed with succesful parameters as compared with official reported results. This includes a 20σ ✕ 20σ ✕ 20σ simulation box with 2 ✕ ⌈ 0.5 ✕ V ⌉ ✕ 10E4 relaxation steps and 2.5 ✕ ⌈ 0.5 ✕ V ⌉ ✕ 10E5 equilibrium steps, for a total of 25,000 samples.

A simple and semi-interactive execution has been designed for those not familiar with the Julia language, where the chemical potential and temperature are taken as parameters, i. e., a system with μ* = -4.151 and T* = 1.5:

    julia GrandCanonical.jl -4.151 1.5

This generates output files with the energy, density and radial distribution function evolution of the system along the simulation, as well as their corresponding plots.

![Density Convergence](img/muVT_DensityConvergence.png)

Code found at [my personal github repository.](https://github.com/omaraalvarez/GrandCanonical)