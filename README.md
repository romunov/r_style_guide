# R code style guide

During your work, you may or may not have had to collaborate with other programmers or god forbid, researchers for whom programming is (was) a hobby (such as myself). These collaborators or even yourself may have their own coding style which praises the god of chaos, making the code very executable by the computer and very unreadable by other humans or non-humans such as oneself in the future. This is a style guide I've devised based on about 8+ years of experience with R and also influenced by Python style guide (PEP).

I fully recognize that any style can make code more readable, irrespective of the style (but must include some sensibly sprinkled carriage returns and spaces) as long as it is consistent. Note that style is by definition heavily based on opinion. When I say that something should (not) be in some particular way, it is my opinion. It should not be taken as fact as I've read of no studies how a certain style of writing increases any kind of performance. That being said, here's approximately what works for me.

### Document your code _well_
Your future self will thank you for it. I find it helpful if I sketch out the algorithm or analysis with comments before hand and then fill in the blanks with code. Write sentences with capital letters, commas, full stops... You know, like a literate person. Place a space between a hash and your text if comment is stand-alone or two spaces and a hash for inline comments. If writing in-line comments (e.g. `# two spaces` below), capitalization, punctuations and the rest can be avoided. Feel free to use these comments to yell or berate your future self.

```
# Assign 3 to a variable.
x <- 3.14  # I could have used the built-in constant, but I feel frisky today
```

If you are planning on ever putting your code into a package (and you probably should), I may suggest adding  [roxygen2](https://cran.r-project.org/web/packages/roxygen2/vignettes/roxygen2.html) "shebang" for title, description, parameter description, output, exports, author and so on. This will document what the function does, what are the inputs, outputs, any quirks it might had AND will be almost ready for being packaged into a package.

A minimal entry would look something along these lines:

```

#' Fetch threshold values (TH) from some database.
#'
#' @param x A slice of `fishbone` object.
#' @param stat Which statistic to fetch, e.g. "RelativeLow", "Disbalance", "Stutter", "LowCount".
#' @param locus Character. For which locus to fetch statistics?
#' @author Roman Luštrik (roman.lustri@@biolitika.si)
#' @return Defined statistic from defined locus on a given marker.

fetchTH <- function(x, stat, locus) {
  out <- x[x$Marker %in% locus, stat, with = FALSE]
  if (length(out) != 1) stop(sprintf("Unable to fetch %s for locus %s.", stat, locus))
  out
}
```

### Use <strike>version control</strike> git, dammit
Kidding aside, you should version all your code. Even if you are coding by yourself, you should use one of the online communities, such as GitHub (you are here) and [BitBucket](https://bitbucket.org/). If you are coding by yourself, you will have an archive and backup and  this will be a _muss haben_ for any collaboration. Sending code via e-mail, Dropbox or any other archaic means is probably something to be avoided. To get you started, here's a [book on Git](https://git-scm.com/book/en/v2).

### Functions and functionNames
Function names should be descriptive and not too long, written as one word, no spaces. Use camel case with first letter not capitalized. Function name is followed by assign sign wrapped in spaces (` <- `), call to function `function()`, followed by function body within curly braces. Opening curly brace is in the same line as function name, closing curly brace is in its own (last) like unless used in an anonymous function. See below for general structure. If you are using [RStudio](https://dailies.rstudio.com/), it should add appropriate number of spaces to code within the function body. If not, make it two spaces.

```
descriptiveFunctionName <- function(x) {  # OK
  out <- x^2
  return(out)
}
```

```
# BAD on several levels. Do not use functions without input, avoid global assignment,
# place curly braces after function(...). Just think what you would write in Java...
# and then don't do that.
func1 <- function()
{
  x <<- y^2
}
```

```
sapply(1:5, FUN = function(x) {
  x^2
})  # this is OK because it's an anonymous function
```

### Function argument names
As with function names, keep function argument names descriptive, short and simple. If you need some sort of a delimiter, use underscore (`_`). This way, when used in function body, a satellite could tell that it's an argument that has been passed into the function. Add spaces between arguments and equal signs. If there are many arguments, break them into lines so that it will fit on a contemporary screen of average resolution. Yes, it can be more than 80 characters, this is not 1984.

```
moduloChecker <- function(x, y, expect_type = 0) {
  modulo <- x %% y
  if (modulo == expect_type) {
    return(TRUE)
  } else {
    return(FALSE)
  }
}
```

### Calling functions
When calling a function, there should be no space between function name and open bracket. Use spaces between arguments and equal signs. When function has more than one argument, first argument should be position matched, all others should be matched by name.

```
n.mean <- sapply(xy, FUN = mean, simplify = FALSE)  # OK
n.mean<-sapply(xy,FUN=mean,simplify=F)  # BAD: use spaces, write out FALSE

slices <- lapply(xy, MARGIN = 1, FUN = extractDailyPoints)  # OK
slices <- lapply(xy, 1,extractDailyPoints)  # BAD: inconsistent spaces, use arg. names
```

### Variable names
This is the hardest part of programming for me. Inventing informative and short names for variables is something I spend a lot of time doing. I give myself extra points for puns. Avoid capitalization. For delimiters use dot (`.`) and _not_ underscores (reserved for function argument names) or camelCase (reserved for function names). Add spaces around assignment ` <- `. Equal sign ` = ` is OK, but whatever. I only use it to pass parameters to functions.

Avoid naming your variables as existing (base) function names, such as `df` (density function of F distribution), `data`, `library`, `sum`, `c`, `range`, `pi`... When you are creating a minimal reproducible example or some throwaway code, you can use something quick, such as `xy` or whatever works on your keyboard.

Be explicit, use `FALSE` and `TRUE` instead of `F` and `T`.

```
if (slice == TRUE) return("OK")  # OK
if (slice==T) return("OK")  # NO: use TRUE and spaces
if (slice) return("OK")  # OK: implicit use of TRUE/FALSE is OK

parent <- xy[(xy$reference_sample %in% input$animals), ]  # OK
Parent <- xy[(xy$reference_sample %in% input$animals), ]  # BAD: capitalized letter has no extra information

num.offspring <- xy[xy$mother %in% i | xy$father %in% i, ]  # OK
numOffspring <- xy[xy$mother %in% i | xy$father %in% i, ]  # BAD: camelCase reserved for functions

par.t <- gather(par, -offspring, key = parent, value = value)  # OK
par_t <- gather(par, -offspring, key=parent, value=value)  # BAD: underscore reserved for function args, missing spaces

xy <- data.frame(names = letters[1:5], values = runif(5))  # OK
c <- data.frame(names = letters[1:5], values = runif(5))  # BAD: do not use reserved words or base function names
```

Make constants in all capital letters.
```
NUM.CORES <- parallel::detectCores()  # number of cores used in simulation
N <- 5  # number of simulated animals
```

### Working with data.frames
When constructing data.frames, use descriptive column names that are easy to type. Don't use a lot of capitalization, especially not for the first letter. While it is possible to use spaces in column names, don't. Using a decent IDE, you should be able to auto-complete column/variable names and punching a shift into the mix is just an unnecessary hassle. If there are many columns, break them into lines.

```
xy <- data.frame(num_samples = 1:5, pop_size = runif(5, min = 100, max = 500),
                 area_size = 500, use_rgeos = TRUE)
```
When subsetting from a data.frame or a similar object (e.g. `matrix`), use space between rows and columns.

```
xy[, c("area_size", "pop_size")]
```

and not

```
xy[,c("area_size", "pop_size")]  # BAD
xy[xy$num_samples<3,"pop_size"]  # TERRIBLE, we are not friends anymore
```

The extra space makes it apparent that columns are being extracted explicitly.

### Workflow control
When using `if` and `else`, use the below structure. It is OK to have things in one line if the operation is simple. Otherwise split body between curly braces into several lines. The curly brace should be on the same line as the function name. The extra line otherwise created by a single `{` lends little to readability.

```
if (control.input) counter <- 0  # OK

# This is OK.
if (input.reset) {
  counter <- 0
  counter <- x + counter
} else {
  counter <- x + counter
}

# This is BAD. Open curly brace should be in the same line as `if`.
if (input.reset)
{
  counter <- 0
  counter <- x + counter
}
```

### Use of pipes
I do not advocate the use of pipes. The vast majority of my programming time is spent thinking, not writing so I see little benefit of "quickly" moving result from one variable to another. Readability of piped code above "classical approach" could be debated until either or all parties are senile or deceased.

### Structuring ggplot code
When writing ggplot code, it may look like piping, it may sound like piping, but it's OK. `ggplot2` implements a kickass concept where you are adding _complex layers_ to a figure. This enables you to think of the figure in more scientific terms and removes the hurdle of thinking about bureaucracy. To this end, I suggest putting every conceptual layer into its own line. Break it into multiple lines if contents of the layer becomes too long for comfort.

```
# OK
ggplot(mtcars, aes(x = mpg, y = wt, color = qsec)) +
  theme_bw() +
  theme(axis.text.x = element_text(size = 12, angle = 90, vjust = 0.35),
        legend.position = "none") +
  geom_point() +
  geom_smooth(method = "lm") +
  facet_wrap(~ vs)
```

I have seen code like this and would rather go blind than to see it again.
```
# TERRIBLE
ggplot(mtcars,aes(x=mpg,y=wt,color=qsec))+theme_bw()+theme(axis.text.x=element_text(size=12,angle=90,vjust=0.35),legend.position="none")+geom_point()+geom_smooth(method="lm")+coord_flip()+facet_wrap(~vs)
```

If you are having problems running the code (and need it in one-two lines), you are not using an IDE or are using it wrong. In Rstudio, pressing CTRL+r or CTRL+Enter will run the whole chunk of code for you.
