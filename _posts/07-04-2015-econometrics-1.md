---
layout: post
title: Econometrics in Julia I
author: Patrick Kofod Mogensen
excerpt: Applications and theory in emperical analysis of problems in economics is called econometrics. Let us see how Julia can be used for econometric analysis. In this first part, we will begin by handling data.
---

# Econometrics in Julia I
There is a lot of software out there for doing econometrics. Linear regression models, generalized linear models (probit and logit), survival analysis, treatment effects, and so on; there's software for it all. Popular choices are Stata, SAS, and R, even EViews and Excel can be used (shivers). With languages like Julia, Python, or R, you get access to statistical and econometric tools, but you also get the features from a general purpose programming language.

I am not going to convince you that Julia is fast. The benchmarks are out there. If you are here, I assume you use, or want to use, Julia - for example for doing econometrics. The obvious question is: can you do all the things you used to do in the other programs? Yes, and no. The relevant packages are not as mature and plenty as those for R. That being said, there are actually rather many packages already available. We will see some, and you will be able to find more in [JuliaStats](https://github.com/JuliaStats). If you need an R-package, there is [RJulia](https://github.com/armgong/RJulia), which links Julia and R.

## Loading data
At the heart of applied statistics and econometrics we find the data. We need to load the data into memory somehow to analyze it. You should download the well-known Engel (1857) data set on food expenditure in Belgish households [(download link)](/files/engel.csv). Navigate to the folder where the .csv-file is saved, and start the Julia REPL (interactive shell). To get started, we need to make sure, that the package DataFrames is available. 

### DataFrames
Julia is a static typed language. This means that a vector of floating point valued numbers cannot contain integers or characters. It is important for Julia that types don't (can't) change, because it uses it to optimize performance of the code. This is relevant for data analysis, because if we were simply to represent our data in a matrix (two-dimensional array), we would have to do one of two things. We could simply store the data in a matrix of floating point numbers, but this would require conversion of integers to floats, and how would strings fit in? Alternatively, we could make a new kind of _array_ which enables each column (variable) to have its own type. Another problem is missing values. If a vector or matrix can only contain one type of content, then how can we represent values side-by-side with missing values? The latter problem is solved in [DataArrays.jl](https://github.com/JuliaStats/DataArrays.jl). This package defines a new type of arrays, which can hold both data entries  (floats or integers for example), and missing values (the NA value). This is used in [DataFrames.jl](https://github.com/JuliaStats/DataFrames.jl) to handle multiple types at once.

To install DataFrames.jl, simply use the [package manager (Pkg)](https://github.com/JuliaLang/julia/blob/master/base/pkg.jl).

{% highlight Julia %}
julia> Pkg.add("DataFrames")
{% endhighlight %}

This grabs DataFrames.jl from [github](http://github.com), and also makes sure that all dependencies are installed.

We can now read our .csv-file into memory. Before we use the `readtable()` function in DataFrames.jl, let us use `readcsv()` from base, so we can see the difference.

{% highlight Julia %}
julia> readcsv("engel.csv")
236x3 Array{Any,2}:
    "id"      "income"     "foodexp"
   1.0     420.158      255.839     
   2.0     541.412      310.959     
   3.0     901.157      485.68      
   ⋮                                
 233.0     581.36       468.001     
 234.0     743.077      522.602     
 235.0    1057.68       750.32      
{% endhighlight %}

We see that the id-column is a float, but in the original file it was an integer. We also see that the first row contains the strings: "id", "income", and "foodexp". This is not how we want to represent our data when doing analyzes, so let us try `readtable()`, which is defined in DataFrames.jl.


{% highlight Julia %}
julia> using DataFrames

julia> df = readtable("engel.csv")
235x3 DataFrame
| Row | id  | income  | foodexp |
|-----|-----|---------|---------|
| 1   | 1   | 420.158 | 255.839 |
| 2   | 2   | 541.412 | 310.959 |
| 3   | 3   | 901.157 | 485.68  |
⋮
| 233 | 233 | 581.36  | 468.001 |
| 234 | 234 | 743.077 | 522.602 |
| 235 | 235 | 1057.68 | 750.32  |
{% endhighlight %}

We see that the strings in the first line are used as column (or variable) names, and the values are read correctly as integers and floats. `df` is of type `DataFrame`. If we're only interested in `foodexp`, we can index into it as follows.

{% highlight Julia %}
julia> df[:foodexp]
235x1 DataFrame
| Row | foodexp |
|-----|---------|
| 1   | 255.839 |
| 2   | 310.959 |
| 3   | 485.68  |
⋮
| 233 | 468.001 |
| 234 | 522.602 |
| 235 | 750.32  |
{% endhighlight %}

To access multiple variables, use an array as the index like this: `df[[:id, :foodexp]]`.

Great, so now we have some data. What can we do with it? Well, we would probably start off by looking at some descriptive statistics. 

{% highlight Julia %}
julia> describe(df)
id
Min      1.0
1st Qu.  59.5
Median   118.0
Mean     118.0
3rd Qu.  176.5
Max      235.0
NAs      0
NA%      0.0%

income
Min      377.058368850099
1st Qu.  638.8757884435065
Median   883.984916757004
Mean     982.473043993119
3rd Qu.  1163.986672067545
Max      4957.81302447901
NAs      0
NA%      0.0%

foodexp
Min      242.32020192074
1st Qu.  429.688763177651
Median   582.54125094185
Mean     624.1501113133554
3rd Qu.  743.8814317451735
Max      2032.67919020832
NAs      0
NA%      0.0%
{% endhighlight %}

We get the usual quartiles, min, max, and mean. For `id` the information is not super informative, but we see that food expenditure seems to be around 2/3 of the budget on average. We also see that the range is larger for income than for food expenditure. Using `describe()`, we get output in our terminal.

### Columnwise calculations
Say we wanted to save the means. Then we could use the `colwise()` function. `colwise()` takes a function and a DataFrame, performs the function on each column (variable), and returns an array of the output. Let us calculate means, medians, and standard deviations for all three variables (even `id`).

{% highlight julia %}
julia> colwise(mean,df)
3-element Array{Any,1}:
 [118.0]  
 [982.473]
 [624.15] 

julia> colwise(median,df)
3-element Array{Any,1}:
 [118.0]  
 [883.985]
 [582.541]

julia> colwise(std,df)
3-element Array{Any,1}:
 [67.9828]
 [519.231]
 [276.457]
{% endhighlight %}

The output is consistent with `describe()` above. An important point of using DataFrames, as mentioned earlier, is that we have missing values (`NA`s). Describe told us, that we had 0 `NA`s, but if we had any, it would be able to handle them (ignore them) when calculating the statistics. 

### The `RDatasets` package
The [RDatasets.jl](https://github.com/johnmyleswhite/RDatasets.jl) package contains a lot of _classic_ data sets, for example the Engel-data set from above. To install the package we use `Pkg`.

{% highlight julia %}

Pkg.add("Rdataset")

{% endhighlight %}

The data sets were originally collected by [Vincent Arelbundock](https://github.com/vincentarelbundock) from various R-packages. To access a data set we use `dataset`, and specify the R-package name and the specific data set. To be clear, we do the following to load the Engel data set, as it was shipped in the [`quantreg`](http://cran.r-project.org/web/packages/quantreg/index.html) package.

{% highlight julia %}
julia> using RDatasets

julia> engel = dataset("quantreg", "engel")
235x2 DataFrame
| Row | Income  | FoodExp |
|-----|---------|---------|
| 1   | 420.158 | 255.839 |
| 2   | 541.412 | 310.959 |
| 3   | 901.157 | 485.68  |
⋮
| 233 | 581.36  | 468.001 |
| 234 | 743.077 | 522.602 |
| 235 | 1057.68 | 750.32  |

{% endhighlight %}

Let us get a bit more interesting data set in memory, and save Engel for quantile regression in another post. We will load the "Housing" data from the "Ecdat" package (econometrics data sets), which is from 
 Anglin, P.M. and R. Gencay (1996) &ldquo;Semiparametric estimation of a hedonic price function&rdquo;, _Journal of Applied Econometrics_, 11(6), 633-648. I've omitted the REPL output from `dataset()`. Let's also use `describe()` to see what's in there.

{% highlight julia %}
julia> Housing = dataset("Ecdat", "Housing")
546x12 DataFrame
| Row | Price  | LotSize | Bedrooms | Bathrms | Stories | Driveway | RecRoom | FullBase | Gashw | AirCo | GaragePl | PrefArea |
|-----|--------|---------|----------|---------|---------|----------|---------|----------|-------|-------|----------|----------|
| 1   | 42000  | 5850    | 3        | 1       | 2       | "yes"    | "no"    | "yes"    | "no"  | "no"  | 1        | "no"     |
| 2   | 38500  | 4000    | 2        | 1       | 1       | "yes"    | "no"    | "no"     | "no"  | "no"  | 0        | "no"     |
| 3   | 49500  | 3060    | 3        | 1       | 1       | "yes"    | "no"    | "no"     | "no"  | "no"  | 0        | "no"     |
⋮
| 544 | 103000 | 6000    | 3        | 2       | 4       | "yes"    | "yes"   | "no"     | "no"  | "yes" | 1        | "no"     |
| 545 | 105000 | 6000    | 3        | 2       | 2       | "yes"    | "yes"   | "no"     | "no"  | "yes" | 1        | "no"     |
| 546 | 105000 | 6000    | 3        | 1       | 2       | "yes"    | "no"    | "no"     | "no"  | "yes" | 1        | "no"     |

julia> describe(Housing)
Price
Min      25000.0
1st Qu.  49125.0
Median   62000.0
Mean     68121.59706959708
3rd Qu.  82000.0
Max      190000.0
NAs      0
NA%      0.0%

LotSize
Min      1650.0
1st Qu.  3600.0
Median   4600.0
Mean     5150.2655677655675
3rd Qu.  6360.0
Max      16200.0
NAs      0
NA%      0.0%

Bedrooms
Min      1.0
1st Qu.  2.0
Median   3.0
Mean     2.9652014652014653
3rd Qu.  3.0
Max      6.0
NAs      0
NA%      0.0%

Bathrms
Min      1.0
1st Qu.  1.0
Median   1.0
Mean     1.2857142857142858
3rd Qu.  2.0
Max      4.0
NAs      0
NA%      0.0%

Stories
Min      1.0
1st Qu.  1.0
Median   2.0
Mean     1.8076923076923077
3rd Qu.  2.0
Max      4.0
NAs      0
NA%      0.0%

Driveway
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2

RecRoom
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2

FullBase
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2

Gashw
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2

AirCo
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2

GaragePl
Min      0.0
1st Qu.  0.0
Median   0.0
Mean     0.6923076923076923
3rd Qu.  1.0
Max      3.0
NAs      0
NA%      0.0%

PrefArea
Length  546
Type    Pooled ASCIIString
NAs     0
NA%     0.0%
Unique  2
{% endhighlight %}

Once again we have no `NA`s, but now we also have `Pooled ASCIIString` variables. For example, `FullBase` is such a variable which describes of there is a full basement or not. As the output tells us, there are two unique values, and they are "yes" or "no". 

### The `by()` function
We saw that the mean price was around 68121.60, but how does this vary with some of the features we have in the data? What is the premium of having a full basement, and what is the premium of having air conditioning? If we simply want to calculate the conditional average prices, we can use the `by()` function.

{% highlight julia %}
julia> by(Housing, :FullBase, df->mean(df[:Price]))
2x2 DataFrame
| Row | FullBase | x1      |
|-----|----------|---------|
| 1   | "no"     | 64477.6 |
| 2   | "yes"    | 74894.5 |

julia> by(Housing, :AirCo, df->mean(df[:Price]))
2x2 DataFrame
| Row | AirCo | x1      |
|-----|-------|---------|
| 1   | "no"  | 59884.9 |
| 2   | "yes" | 85880.6 |
{% endhighlight %}

Alternatively, we can use the `do` block form. This time we add a name to the mean, and also calculate the standard deviation. Notice Julia fully supports UTF-8 characters. In Ubuntu I've mapped the "Windows Button + Space" to cycle between my local keyboard layout and Greek keyboard layout. 

{% highlight julia %}
julia> by(housing, :FullBase) do df
                   DataFrame(μ = mean(df[:Price]), σ  = std(df[:Price]))
               end
2x3 DataFrame
| Row | FullBase | μ       | σ       |
|-----|----------|---------|---------|
| 1   | "no"     | 64477.6 | 26281.0 |
| 2   | "yes"    | 74894.5 | 26219.9 |
{% endhighlight %}

### Further indexing
What if we want to create a DataFrame conditional on some statement? Say we only want house sales with full basements. This is done as in the following example.

{% highlight julia %}
julia> Housing[Housing[:FullBase] .== "yes",[:Price,:Bedrooms]]
191x2 DataFrame
| Row | Price | Bedrooms |
|-----|-------|----------|
| 1   | 42000 | 3        |
| 2   | 66000 | 3        |
| 3   | 66000 | 3        |
⋮
| 189 | 66000 | 3        |
| 190 | 58900 | 2        |
| 191 | 71000 | 3        |
{% endhighlight %}

I only picked out the sales price and number of bedrooms, but you can include all columns by using `:` instead of the array `[:Price, :Bedrooms]`. 

## The end and the future
To make this small note digestible, I will stop here. This little post was supposed to be a little introduction to working with data sets in Julia, and in a future post, I will try to get some actual econometrics done, and we will also look at plotting data.

There is a small caveat that you should be aware of. Some of the details such as the `NA` value is going to change in Julia v0.4. In addition, the types `CategoricalVariable` and `OrdinalVariable` will be introduced as a replacement for the `Pooled` types. See more at [John Myles White's blog](http://www.johnmyleswhite.com/notebook/2014/11/29/whats-wrong-with-statistics-in-julia/).