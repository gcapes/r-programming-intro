---
title: "Unused end of carpentry ep 02"
teaching: 30
exercises: 0
questions:
- "How do I make a function?"
objectives:
- "Define a function that takes arguments."
keypoints:
- "Define a function using `name <- function(...args...) {...body...}`."
---

(This episode is derived from episode 2 of [Software Carpentry's Programming with R course](https://github.com/swcarpentry/r-novice-inflammation/))






I

## Testing and documenting


Once we start putting things in functions so that we can re-use them, we need to start testing that those functions are working correctly.  


To see how to do this, let's write a function to center a dataset around a particular value:


~~~
center <- function(data, desired) {
  new_data <- (data - mean(data)) + desired
  return(new_data)
}
~~~
{: .r}

We could test this on our actual data, but since we don't know what the values ought to be, it will be hard to tell if the result was correct.
Instead, let's create a vector of 0s and then center that around 3.
This will make it simple to see if our function is working as expected:


~~~
z <- c(0, 0, 0, 0)
z
~~~
{: .r}



~~~
[1] 0 0 0 0
~~~
{: .output}



~~~
center(z, 3)
~~~
{: .r}



~~~
[1] 3 3 3 3
~~~
{: .output}

That looks right, so let's try center on our real data. We'll center the inflammation data from day 4 around 0:


~~~
dat <- read.csv(file = "data/inflammation-01.csv", header = FALSE)
centered <- center(dat[, 4], 0)
head(centered)
~~~
{: .r}

It's hard to tell from the default output whether the result is correct, but there are a few simple tests that will reassure us:


~~~
# original min
min(dat[, 4])
# original mean
mean(dat[, 4])
# original max
max(dat[, 4])
# centered min
min(centered)
# centered mean
mean(centered)
# centered max
max(centered)
~~~
{: .r}

That seems almost right: the original mean was about ` round(mean(dat[, 4]), 2)`, so the lower bound from zero is now about ` -round(mean(dat[, 4]), 2)`.
The mean of the centered data is ` mean(centered)`.
We can even go further and check that the standard deviation hasn't changed:


~~~
# original standard deviation
sd(dat[, 4])
# centered standard deviation
sd(centered)
~~~
{: .r}

Those values look the same, but we probably wouldn't notice if they were different in the sixth decimal place.
Let's do this instead:


~~~
# difference in standard deviations before and after
sd(dat[, 4]) - sd(centered)
~~~
{: .r}

Sometimes, a very small difference can be detected due to rounding at very low decimal places.
R has a useful function for comparing two objects allowing for rounding errors, `all.equal`:


~~~
all.equal(sd(dat[, 4]), sd(centered))
~~~
{: .r}

It's still possible that our function is wrong, but it seems unlikely enough that we should probably get back to doing our analysis.
We have one more task first, though: we should write some [documentation]({{ page.root }}/reference#documentation) for our function to remind ourselves later what it's for and how to use it.

A common way to put documentation in software is to add [comments]({{ page.root }}/reference/#comment) like this:


~~~
center <- function(data, desired) {
  # return a new vector containing the original data centered around the
  # desired value.
  # Example: center(c(1, 2, 3), 0) => c(-1, 0, 1)
  new_data <- (data - mean(data)) + desired
  return(new_data)
}
~~~
{: .r}

> ## Writing Documentation
>
> Formal documentation for R functions is written in separate `.Rd` using a
> markup language similar to [LaTeX][]. You see the result of this documentation
> when you look at the help file for a given function, e.g. `?read.csv`.
> The [roxygen2][] package allows R coders to write documentation alongside
> the function code and then process it into the appropriate `.Rd` files.
> You will want to switch to this more formal method of writing documentation
> when you start writing more complicated R projects.
{: .callout}

[LaTeX]: http://www.latex-project.org/
[roxygen2]: http://cran.r-project.org/web/packages/roxygen2/vignettes/rd.html





> ## Functions to Create Graphs
>
> Write a function called `analyze` that takes a filename as a argument
> and displays the three graphs produced in the [previous lesson][01] (average, min and max inflammation over time).
> `analyze("data/inflammation-01.csv")` should produce the graphs already shown,
> while `analyze("data/inflammation-02.csv")` should produce corresponding graphs for the second data set.
> Be sure to document your function with comments.
>
> > ## Solution
> > ~~~
> > analyze <- function(filename) {
> >   # Plots the average, min, and max inflammation over time.
> >   # Input is character string of a csv file.
> >   dat <- read.csv(file = filename, header = FALSE)
> >   avg_day_inflammation <- apply(dat, 2, mean)
> >   plot(avg_day_inflammation)
> >   max_day_inflammation <- apply(dat, 2, max)
> >   plot(max_day_inflammation)
> >   min_day_inflammation <- apply(dat, 2, min)
> >   plot(min_day_inflammation)
> > }
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

> ## Rescaling
>
> Write a function `rescale` that takes a vector as input and returns a corresponding vector of values scaled to lie in the range 0 to 1.
> (If $L$ and $H$ are the lowest and highest values in the original vector, then the replacement for a value $v$ should be $(v-L) / (H-L)$.)
> Be sure to document your function with comments.
>
> Test that your `rescale` function is working properly using `min`, `max`, and `plot`.
>
> > ## Solution
> > ~~~
> > rescale <- function(v) {
> >   # Rescales a vector, v, to lie in the range 0 to 1.
> >   L <- min(v)
> >   H <- max(v)
> >   result <- (v - L) / (H - L)
> >   return(result)
> > }
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

[01]: {{ page.root }}/01-starting-with-data/



### Defining Defaults

We have passed arguments to functions in two ways: directly, as in `dim(dat)`, and by name, as in `read.csv(file = "data/inflammation-01.csv", header = FALSE)`.
In fact, we can pass the arguments to `read.csv` without naming them:


~~~
dat <- read.csv("data/inflammation-01.csv", FALSE)
~~~
{: .r}



~~~
Warning in file(file, "rt"): cannot open file 'data/inflammation-01.csv':
No such file or directory
~~~
{: .error}



~~~
Error in file(file, "rt"): cannot open the connection
~~~
{: .error}

However, the position of the arguments matters if they are not named.


~~~
dat <- read.csv(header = FALSE, file = "data/inflammation-01.csv")
~~~
{: .r}



~~~
Warning in file(file, "rt"): cannot open file 'data/inflammation-01.csv':
No such file or directory
~~~
{: .error}



~~~
Error in file(file, "rt"): cannot open the connection
~~~
{: .error}



~~~
dat <- read.csv(FALSE, "data/inflammation-01.csv")
~~~
{: .r}



~~~
Error in read.table(file = file, header = header, sep = sep, quote = quote, : 'file' must be a character string or connection
~~~
{: .error}

To understand what's going on, and make our own functions easier to use, let's re-define our `center` function like this:


~~~
center <- function(data, desired = 0) {
  # return a new vector containing the original data centered around the
  # desired value (0 by default).
  # Example: center(c(1, 2, 3), 0) => c(-1, 0, 1)
  new_data <- (data - mean(data)) + desired
  return(new_data)
}
~~~
{: .r}

The key change is that the second argument is now written `desired = 0` instead of just `desired`.
If we call the function with two arguments, it works as it did before:


~~~
test_data <- c(0, 0, 0, 0)
center(test_data, 3)
~~~
{: .r}



~~~
[1] 3 3 3 3
~~~
{: .output}

But we can also now call `center()` with just one argument, in which case `desired` is automatically assigned the default value of `0`:


~~~
more_data <- 5 + test_data
more_data
~~~
{: .r}



~~~
[1] 5 5 5 5
~~~
{: .output}



~~~
center(more_data)
~~~
{: .r}



~~~
[1] 0 0 0 0
~~~
{: .output}

This is handy: if we usually want a function to work one way, but occasionally need it to do something else, we can allow people to pass an argument when they need to but provide a default to make the normal case easier.

The example below shows how R matches values to arguments


~~~
display <- function(a = 1, b = 2, c = 3) {
  result <- c(a, b, c)
  names(result) <- c("a", "b", "c")  # This names each element of the vector
  return(result)
}

# no arguments
display()
~~~
{: .r}



~~~
a b c 
1 2 3 
~~~
{: .output}



~~~
# one argument
display(55)
~~~
{: .r}



~~~
 a  b  c 
55  2  3 
~~~
{: .output}



~~~
# two arguments
display(55, 66)
~~~
{: .r}



~~~
 a  b  c 
55 66  3 
~~~
{: .output}



~~~
# three arguments
display (55, 66, 77)
~~~
{: .r}



~~~
 a  b  c 
55 66 77 
~~~
{: .output}

As this example shows, arguments are matched from left to right, and any that haven't been given a value explicitly get their default value.
We can override this behavior by naming the value as we pass it in:


~~~
# only setting the value of c
display(c = 77)
~~~
{: .r}



~~~
 a  b  c 
 1  2 77 
~~~
{: .output}

> ## Matching Arguments
>
> To be precise, R has three ways that arguments are supplied
> by you are matched to the *formal arguments* of the function definition:
>
> 1. by complete name,
> 2. by partial name (matching on initial *n* characters of the argument name), and
> 3. by position.
>
> Arguments are matched in the manner outlined above in *that order*: by
> complete name, then by partial matching of names, and finally by position.
{: .callout}

With that in hand, let's look at the help for `read.csv()`:


~~~
?read.csv
~~~
{: .r}

There's a lot of information there, but the most important part is the first couple of lines:


~~~
read.csv(file, header = TRUE, sep = ",", quote = "\"",
         dec = ".", fill = TRUE, comment.char = "", ...)
~~~
{: .r}

This tells us that `read.csv()` has one argument, `file`, that doesn't have a default value, and six others that do.
Now we understand why the following gives an error:


~~~
dat <- read.csv(FALSE, "data/inflammation-01.csv")
~~~
{: .r}



~~~
Error in read.table(file = file, header = header, sep = sep, quote = quote, : 'file' must be a character string or connection
~~~
{: .error}

It fails because `FALSE` is assigned to `file` and the filename is assigned to the argument `header`.

> ## A Function with Default Argument Values
>
> Rewrite the `rescale` function so that it scales a vector to lie between 0 and 1 by default, but will allow the caller to specify lower and upper bounds if they want.
> Compare your implementation to your neighbor's: do the two functions always behave the same way?
>
> > ## Solution
> > ~~~
> > rescale <- function(v, lower = 0, upper = 1) {
> >   # Rescales a vector, v, to lie in the range lower to upper.
> >   L <- min(v)
> >   H <- max(v)
> >   result <- (v - L) / (H - L) * (upper - lower) + lower
> >   return(result)
> > }
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

