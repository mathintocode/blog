---
published: true
images:
  - url: /blog/img/interpolation_1.png
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

<img alt="Interpolation" src="/blog/img/interpolation_1.png"> 

<div style="text-align: justify">Whenever one has a set of discrete data points given by sampling or experimentation, they correspond to the values of a certain function. Since we don't know the complete set of values, one needs to <b>interpolate</b> in order to estimate the value of the function between the known data.For this, it is required a spline which is functioned defined piecewise by polynomials of order $k \in N$ differentiable $k-1$ times in $[a,b]$: $S(x) \in C^{k-1}[a,b]$. <br><br>
This spline must satisfy three basic conditions:<br>
  1. Continous function: $S_{i-1,i}(x_i) = S_{i,i+1}(x_i)  \forall  i$<br>
  2. Same slope: $S'_{i-1,i}(x_i) = S'_{i,i+1}(x_i)  \forall  i$<br>
  3. Same concavity: $S''_{i-1,i}(x_i) = S''_{i,i+1}(x_i) = K_i \forall  i$<br><br>
The polynomials can be of any order but for $k = 1$ the second and third condition aren't completely satisfied. The same goes for $k = 2$ and the third condition. Naturally, for the imposed conditions it's required $k \geq 3$, but for this specific case, we'll stick with this lowest possible order.<br>
Therefore, for third order:<br>
$S(x)$: piecewise <b>cubic</b> polynomial.<br>
$S'(x)$: piecewise <b>quadratic</b> polynomial.<br>
$S''(x)$: piecewise <b>lineal</b> polynomial.<br>
Since no information is known related to $S(x)$, one starts by putting forward a Lagrange interpolation for the second derivative:<br>
$$S''_{i,i+1}(x) = K_i \frac{x-x_{i+1}}{x_i - x_{i+1}} + K_{i+1} \frac{x-x_{i}}{x_{i+1} - x_i}$$<br>
Integrating twice:
$$S_{i,i+1}(x) = \frac{K_i}{6} \frac{(x-x_{i+1})^3}{x_i-x_{i+1}} + \frac{K_{i+1}}{6} \frac{(x-x_i)^3}{x_{i+1}-x_i} + C_1x + C_2$$
Redefining the constants $C_1 = \alpha - \beta$ and $C_2 = -\alpha x_{i+1} + \beta x_i$ and giving it a more symmetric structure:
$$S_{i,i+1}(x) = \frac{K_i}{6} \frac{(x-x_{i+1})^3}{x_i-x_{i+1}} + \frac{K_{i+1}}{6} \frac{(x-x_i)^3}{x_{i+1}-x_i} + \alpha (x-x_{i+1}) + \beta (x-x_i)$$<br>
Given the boundary condition $S_{i,i+1}(x_i) = y_i \Rightarrow \alpha = \frac{y_i}{x_i-x_{i+1}} - \frac{K_i}{6}(x_i-x_{i+1})$<br>
Given the boundary condition $S_{i,i+1}(x_{i+1}) = y_{i+1} \Rightarrow \beta = \frac{y_{i+1}}{x_{i+1}-x_i} - \frac{K_{i+1}}{6}(x_{i+1}-x_i)$<br>
This is the function that'll interpolate the data, but the values for $K_i, K_{i+1}$ are still unknown.
<b>$$S_{i,i+1}(x) = \frac{K_i}{6} (\frac{(x-x_{i+1})^3}{x_i-x_{i+1}} -(x-x_{i+1})(x_i-x_{i+1})) - \frac{K_{i+1}}{6} (\frac{(x-x_i)^3}{x_i-x_{i+1}} - (x-x_i)(x_i-x_{i+1})) + \frac{y_i(x-x_{i+1})-y_{i+1}(x-x_i)}{x_i-x_{i+1}}$$</b>
{% highlight julia linenos %}

using Plots
using LinearAlgebra

{% endhighlight %}

{% highlight julia linenos %}

function sorting_xy(x, y)
    size(x) == size(y) ? tuple_n = size(x) : error("Arrays must be the same size.")
    N = tuple_n[1]
    for i = N:-1:1, j = 1:i-1
        if x[j] > x[j+1]
            Γ, Λ = x[j], y[j]
            x[j], y[j] = x[j+1], y[j+1]
            x[j+1], y[j+1] = Γ, Λ
        end
    end
end

{% endhighlight %}

{% highlight julia linenos %}

function natural_cubic_spline(x::Array{Float64}, y::Array{Float64}, η)
    tuple_n = size(x)
    N = tuple_n[1]
    k = Array{Float64}(undef, N)
    k[1] = k[end] = 0
    A = Matrix(zeros(Float64, N-2, N-2))
    B = Array(zeros(Float64, N-2))
    for i = 2:N-1
        B[i-1] = 6( (y[i-1] - y[i])/(x[i-1] - x[i]) - (y[i] - y[i+1])/(x[i] - x[i+1]) )
        if i != N-1
            A[i-1, i-1] = 2 * (x[i-1] - x[i+1])
            A[i-1, i] = A[i, i-1] = x[i] - x[i+1]
        else
            A[i-1, i-1] = 2 * (x[i-1] - x[i+1])
        end
    end
    SymTridiagonal(A)
    k_aux = inv(A) * B
    for i = 1:N-2
        k[i+1] = k_aux[i]
    end
    Δ = abs((x[end]-x[1])/η)
    ξ = x[1]+Δ
    i = 1
    r = 1
    x_interpolation = Array(zeros(Float64, convert(Int64, η)))
    y_interpolation = Array(zeros(Float64, convert(Int64, η)))
    while ξ <= x[end]
        if ξ >= x[i+1]
            i += 1
        end
        x_interpolation[r] = ξ
        α = (k[i]/6)*(((ξ-x[i+1])^3)/(x[i]-x[i+1]) - (ξ - x[i+1])*(x[i]-x[i+1]))
        β = (k[i+1]/6)*(((ξ-x[i])^3)/(x[i]-x[i+1]) - (ξ - x[i])*(x[i] - x[i+1]))
        γ = (y[i]*(ξ-x[i+1])-y[i+1]*(ξ-x[i]))/(x[i]-x[i+1])
        y_interpolation[r] = α - β + γ
        ξ += Δ
        r += 1
    end
    if x_interpolation[end] == 0.0
        resize!(x_interpolation, convert(Int64, η)-1)
        resize!(y_interpolation, convert(Int64, η)-1)
    end
    scatter!((x_interpolation, y_interpolation), 
    		color = "black", 
            markersize = 1)
end

{% endhighlight %}

{% highlight julia linenos %}

function Cubic_Interpolation(points::Int64, η)
    x = randn(points)
    y = randn(points)
    if issorted(x) == false
        sorting_xy(x,y)
    end
    p = scatter((x,y), 
    			xlabel = "x", 
                ylabel = "f(x)", 
                color = "red", 
                markersize = 6, 
                legend = false, 
                ylim = [minimum(y)-1,maximum(y)+1])
    natural_cubic_spline(x, y, points*η)
end

{% endhighlight %}
