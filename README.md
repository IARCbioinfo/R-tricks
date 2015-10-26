![R Logo](https://www.r-project.org/Rlogo.png)

# Tip and tricks for R
- [Usefull tools](https://github.com/IARC-bioinfo/R-tricks#usefull-tools)
- [Tip and tricks](https://github.com/IARC-bioinfo/R-tricks#tip-and-tricks)
 - [Use Rscript to run R from bash](https://github.com/IARC-bioinfo/R-tricks#use-rscript-to-run-r-from-bash)
 - [Running bash commands from R](https://github.com/IARC-bioinfo/R-tricks#running-bash-commands-from-r)
 - [Building an argument section in R](https://github.com/IARC-bioinfo/R-tricks#building-an-argument-section-for-your-r-script)

## Usefull tools
- [R](https://www.r-project.org)
- [RStudio](https://www.rstudio.com) 
- [Revolution Analytics](http://www.revolutionanalytics.com) is a R fork made to be faster.

## Modern R
There are new ways to use R for data processing workflow, in particular using combination of these packages: [`dplyr`](https://github.com/hadley/dplyr), [`magrittr`](https://github.com/smbache/magrittr), [`tidyr`](https://github.com/hadley/tidyr) and [`ggplot2`](https://github.com/hadley/ggplot2). A few nice tutorial/introductions:
- http://zevross.com/blog/2015/01/13/a-new-data-processing-workflow-for-r-dplyr-magrittr-tidyr-ggplot2/
- http://blog.rstudio.org/2014/07/22/introducing-tidyr/
- https://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html
- http://www.r-statistics.com/2014/08/simpler-r-coding-with-pipes-the-present-and-future-of-the-magrittr-package/

## Tip and tricks

### Use Rscript to run R from bash
R comes with `Rscript`, a command line tool that can run R scripts or R commands directly from the bash shell.

The most compact way to run it is with the `-e` option containing directly the R expression to evaluate. For example the following command will output 10 random numbers:
```bash
Rscript -e 'res=runif(10);cat(res,"\n")'
```

Of course this is only usefull for very short commands. An alternative is to write a R script, for example if you create a file called `test.r` containing:
```R
res=runif(10)
cat(res,"\n")
```
You can run it using `Rscript test.r`. And even better, if you add an initial [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line `#!/usr/bin/env Rscript` in the script above and make it executable with `chmod +x test.r`, you can directly launch your R script with `./test.r`!

Now a common thing to have in scripts is command line parameters. The R function `commandArgs()` returns the command line arguments as a vector of strings (so you will need to convert them to numeric in some cases). By default the first element of the vector is the name of the Rscript executable, the second if the first argument and so on. Most of the time you won't need it so you should rather use `commandArgs(trailingOnly = TRUE)` instead (or the compact version `commandArgs(T)`) to have the first element being the first argument and so on. You can easily check if the command line argument is missing: R puts an `NA`. For example, our script outputing random numbers will look like this if we want to output 10 numbers by default or to have this number as a command line argument:
```R
#!/usr/bin/env Rscript
N=as.numeric(commandArgs(TRUE)[1])
if (is.na(N)) N=10
res=runif(N)
cat(res,"\n")
```
You can run it using:
```bash
./test.r 5
0.2852298 0.9366892 0.752774 0.4416602 0.9603793 
```
or
```bash
./test.r 
0.3131752 0.3540976 0.07491065 0.125842 0.2947516 0.203168 0.8390772 0.6115891 0.323192 0.783478 
```
If you have a second argument it way look like:
```R
#!/usr/bin/env Rscript
N1=as.numeric(commandArgs(TRUE)[1])
N2=as.numeric(commandArgs(TRUE)[2])
res1=runif(N1)
res2=rnorm(N2)
cat(res1,"\n")
cat(res2,"\n")
```
And running:
```bash
./test.r 4 8
0.6631743 0.2670673 0.5929007 0.8545739 
-0.6988854 -0.4150706 1.0834 -0.002987133 -2.552233 -0.6456261 0.7652581 0.7687048 
```
outputs 4 uniform numbers and 8 normally distributed numbers. Note that in this case it's harder to check the presence or absence of arguments.

Also note that [`funr`](https://github.com/sahilseth/funr) is an interesting tool providing shell access to all R functions.

### Running bash commands from R
Now the opposite: you need to run a bash command from y R script. In this section all commands should be run in R unless specified. The easiest way is to use the `system()` function:
```R
system('ls -l | wc -l')
```
By default the `system()` function returns an error code (0 for success and 127 for failure). If you want to get the result of the command back in a R variable, you can use the `intern = TRUE` option. In this case it will return a character vector with one string element for each output line. Again, use `as.numeric()` to transform the output to a number if needed. For example this command will give you the number of files in the current directory in the variable `num_files` (note that there are easier and cleaner ways of doing that in R directly, that's just a toy example):
```R
num_files=as.numeric(system('ls -l | wc -l',intern = TRUE))
```

If the output is more than a single line and you want to load it in a data frame, you can use the function `pipe()` instead of system. For example the following **bash command** returns the names of the files and their sizes in the current directory:
```bash
ls -l | tail -n +2 | awk '{print $9"\t"$5}'
```
Now in R, you can use `read.table` and `pipe` together to call the command and put the result in a data frame `file_sizes`:
```R
file_sizes=read.table(pipe('ls -l | tail -n +2 | awk \'{print $9"\t"$5}\''))
```
Note that the `'` chars in the bash command need to be escaped with `\'` in R because the command itself has to be a string in R delimited by `'`.


### Building an argument section for your R script

This is an example of how having a beautiful argument section in the head of your R script, with possible optional arguments

```R
#! /usr/bin/Rscript

## Collect arguments
args <- commandArgs(TRUE)

## Parse arguments (we expect the form --arg=value)
parseArgs <- function(x) strsplit(sub("^--", "", x), "=")
argsL <- as.list(as.character(as.data.frame(do.call("rbind", parseArgs(args)))$V2))
names(argsL) <- as.data.frame(do.call("rbind", parseArgs(args)))$V1
args <- argsL
rm(argsL)

## Give some value to options if not provided 
if(is.null(args$opt_arg1)) {args$opt_arg1="default_option1"}
if(is.null(args$opt_arg2)) {args$opt_arg2="default_option1"} else {args$opt_arg2=as.numeric(args$opt_arg2)}

## Default setting when no all arguments passed or help needed
if("--help" %in% args | is.null(args$arg1) | is.null(args$arg2)) {
  cat("
      The R Script arguments_section.R
      
      Mandatory arguments:
      --arg1=type           - description
      --arg2=type           - description
      --help                - print this text
      
      Optionnal arguments:
      --opt_arg1=String          - example:an absolute path, default:default_option1
      --opt_arg2=Value           - example:a threshold, default:10

      WARNING : here put all the things the user has to know
      
      Example:
      ./arguments_section.R --arg1=~/Documents/ --arg2=10 --opt_arg2=8 \n\n")
  
  q(save="no")
}

cat("first mandatory argument : ", args$arg1,"\n",sep="")
cat("second mandatory argument : ", args$arg2,"\n",sep="")
cat("first optional argument : ", args$opt_arg1,"\n",sep="")
cat("second optional argument : ", args$opt_arg2,"\n",sep="")
```
