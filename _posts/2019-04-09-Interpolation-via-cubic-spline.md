---
published: true
images:
  - url: /blog/img/interpolation_1.png
---
<img alt="Interpolation" src="/blog/img/interpolation_1.png"> 

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