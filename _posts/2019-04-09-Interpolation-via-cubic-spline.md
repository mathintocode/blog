---
published: true
images:
  - url: /blog/img/interpolation_1.png
---
<img alt="Interpolation" src="/blog/img/interpolation_1.png"> 

{% highlight julia linenos %}

using Plots
using LinearAlgebra
using DelimitedFiles

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
    scatter!((x_interpolation, y_interpolation), color = "black", markersize = 1)
end

{% endhighlight %}