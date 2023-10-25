Mini Data Analysis Milestone 2
================

*To complete this milestone, you can either edit [this `.rmd`
file](https://raw.githubusercontent.com/UBC-STAT/stat545.stat.ubc.ca/master/content/mini-project/mini-project-2.Rmd)
directly. Fill in the sections that are commented out with
`<!--- start your work here--->`. When you are done, make sure to knit
to an `.md` file by changing the output in the YAML header to
`github_document`, before submitting a tagged release on canvas.*

# Welcome to the rest of your mini data analysis project!

In Milestone 1, you explored your data. and came up with research
questions. This time, we will finish up our mini data analysis and
obtain results for your data by:

- Making summary tables and graphs
- Manipulating special data types in R: factors and/or dates and times.
- Fitting a model object to your data, and extract a result.
- Reading and writing data as separate files.

We will also explore more in depth the concept of *tidy data.*

**NOTE**: The main purpose of the mini data analysis is to integrate
what you learn in class in an analysis. Although each milestone provides
a framework for you to conduct your analysis, it’s possible that you
might find the instructions too rigid for your data set. If this is the
case, you may deviate from the instructions – just make sure you’re
demonstrating a wide range of tools and techniques taught in this class.

# Instructions

**To complete this milestone**, edit [this very `.Rmd`
file](https://raw.githubusercontent.com/UBC-STAT/stat545.stat.ubc.ca/master/content/mini-project/mini-project-2.Rmd)
directly. Fill in the sections that are tagged with
`<!--- start your work here--->`.

**To submit this milestone**, make sure to knit this `.Rmd` file to an
`.md` file by changing the YAML output settings from
`output: html_document` to `output: github_document`. Commit and push
all of your work to your mini-analysis GitHub repository, and tag a
release on GitHub. Then, submit a link to your tagged release on canvas.

**Points**: This milestone is worth 50 points: 45 for your analysis, and
5 for overall reproducibility, cleanliness, and coherence of the Github
submission.

**Research Questions**: In Milestone 1, you chose two research questions
to focus on. Wherever realistic, your work in this milestone should
relate to these research questions whenever we ask for justification
behind your work. In the case that some tasks in this milestone don’t
align well with one of your research questions, feel free to discuss
your results in the context of a different research question.

# Learning Objectives

By the end of this milestone, you should:

- Understand what *tidy* data is, and how to create it using `tidyr`.
- Generate a reproducible and clear report using R Markdown.
- Manipulating special data types in R: factors and/or dates and times.
- Fitting a model object to your data, and extract a result.
- Reading and writing data as separate files.

# Setup

Begin by loading your data and the tidyverse package below:

``` r
library(datateachr) # <- might contain the data you picked!
library(tidyverse)
```

# Task 1: Process and summarize your data

From milestone 1, you should have an idea of the basic structure of your
dataset (e.g. number of rows and columns, class types, etc.). Here, we
will start investigating your data more in-depth using various data
manipulation functions.

### 1.1 (1 point)

First, write out the 4 research questions you defined in milestone 1
were. This will guide your work through milestone 2:

<!-------------------------- Start your work below ---------------------------->

1.  *How is the mean radius of nuclei related to the diagnosis?*
2.  *Which factors has the most significant effect on the diagnosis?*
3.  *Is there any difference in the average symmetry_mean between the
    malignant cancer and benign cancer?*
4.  *Is there any difference in the average compactness_worst between
    the malignant cancer and benign cancer?*
    <!----------------------------------------------------------------------------->

Here, we will investigate your data using various data manipulation and
graphing functions.

### 1.2 (8 points)

Now, for each of your four research questions, choose one task from
options 1-4 (summarizing), and one other task from 4-8 (graphing). You
should have 2 tasks done for each research question (8 total). Make sure
it makes sense to do them! (e.g. don’t use a numerical variables for a
task that needs a categorical variable.). Comment on why each task helps
(or doesn’t!) answer the corresponding research question.

Ensure that the output of each operation is printed!

Also make sure that you’re using dplyr and ggplot2 rather than base R.
Outside of this project, you may find that you prefer using base R
functions for certain tasks, and that’s just fine! But part of this
project is for you to practice the tools we learned in class, which is
dplyr and ggplot2.

**Summarizing:**

1.  Compute the *range*, *mean*, and *two other summary statistics* of
    **one numerical variable** across the groups of **one categorical
    variable** from your data.
2.  Compute the number of observations for at least one of your
    categorical variables. Do not use the function `table()`!
3.  Create a categorical variable with 3 or more groups from an existing
    numerical variable. You can use this new variable in the other
    tasks! *An example: age in years into “child, teen, adult, senior”.*
4.  Compute the proportion and counts in each category of one
    categorical variable across the groups of another categorical
    variable from your data. Do not use the function `table()`!

**Graphing:**

6.  Create a graph of your choosing, make one of the axes logarithmic,
    and format the axes labels so that they are “pretty” or easier to
    read.
7.  Make a graph where it makes sense to customize the alpha
    transparency.

Using variables and/or tables you made in one of the “Summarizing”
tasks:

8.  Create a graph that has at least two geom layers.
9.  Create 3 histograms, with each histogram having different sized
    bins. Pick the “best” one and explain why it is the best.

Make sure it’s clear what research question you are doing each operation
for!

**Question 1**

*How is the mean radius of nuclei related to the diagnosis?*

Compute the *range*, *mean*, and median and standard deviation of
`radius_mean` across the groups of `diagnosis`.

``` r
dt <- cancer_sample %>%
 group_by(diagnosis) %>% summarise(
   min=min(radius_mean),
   max=max(radius_mean),
   mean=mean(radius_mean),
   median=median(radius_mean),
   sd=sd(radius_mean),
   n=n()
 )
dt
```

    ## # A tibble: 2 × 7
    ##   diagnosis   min   max  mean median    sd     n
    ##   <chr>     <dbl> <dbl> <dbl>  <dbl> <dbl> <int>
    ## 1 B          6.98  17.8  12.1   12.2  1.78   357
    ## 2 M         11.0   28.1  17.5   17.3  3.20   212

Make a graph where it makes sense to customize the alpha transparency.

``` r
ggplot(cancer_sample, aes(x=radius_mean, fill=diagnosis)) + 
  geom_density(alpha=0.5) +
  labs(x="Mean radius", y="Density",
       fill="Diagnosis")
```

![](mini-project-2_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

**Question 2** *Which factors has the most significant effect on the
diagnosis?*

Create a categorical variable with 3 or more groups from an existing
numerical variable.

``` r
cancer_sample <- cancer_sample %>% 
  mutate(area_mean_group=cut(area_mean, 
  breaks = quantile(cancer_sample$radius_mean, c(0, 0.25, 0.75, 1)), 
  labels = c("Low", "Middle", "High")))
```

Create a graph of histogram, make one of the axes logarithmic, and
format the axes labels so that they are “pretty” or easier to read.

``` r
ggplot(cancer_sample, aes(x=area_mean)) + 
  geom_histogram(color="white", bins=30) +
  labs(x="Mean Area(log)",
       title="Histogram of the mean area(log)", 
       y="Frequency") +
  scale_x_log10() 
```

![](mini-project-2_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

**Question 3** *Is there any difference in the average symmetry_mean
between the malignant cancer and benign cancer?*

Compute the *range*, *mean*, and median and standard deviation of
`symmetry_mean` across the groups of `diagnosis`.

``` r
df <- cancer_sample %>%
 group_by(diagnosis) %>% summarise(
   min=min(symmetry_mean),
   max=max(symmetry_mean),
   mean=mean(symmetry_mean),
   median=median(symmetry_mean),
   sd=sd(symmetry_mean),
   n=n()
 )
df
```

    ## # A tibble: 2 × 7
    ##   diagnosis   min   max  mean median     sd     n
    ##   <chr>     <dbl> <dbl> <dbl>  <dbl>  <dbl> <int>
    ## 1 B         0.106 0.274 0.174  0.171 0.0248   357
    ## 2 M         0.131 0.304 0.193  0.190 0.0276   212

Create a graph that has at least two geom layers.

``` r
ggplot(df, aes(x=diagnosis, y=mean, fill=diagnosis)) +
  geom_col(show.legend = F) +
  geom_errorbar(aes(ymin=mean-sd, ymax=mean+sd), width=0.5) +
  labs(title="Average mean symmetry of nuclei present in sample image",
       y="Average value")
```

![](mini-project-2_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

**Question 4**

*Is there any difference in the average compactness_worst between the
malignant cancer and benign cancer?*

Compute the *range*, *mean*, and median and standard deviation of
`compactness_worst` across the groups of `diagnosis`.

``` r
df <- cancer_sample %>%
 group_by(diagnosis) %>% summarise(
   min=min(compactness_worst),
   max=max(compactness_worst),
   mean=mean(compactness_worst),
   median=median(compactness_worst),
   sd=sd(compactness_worst),
   n=n()
 )
df
```

    ## # A tibble: 2 × 7
    ##   diagnosis    min   max  mean median     sd     n
    ##   <chr>      <dbl> <dbl> <dbl>  <dbl>  <dbl> <int>
    ## 1 B         0.0273 0.585 0.183  0.170 0.0922   357
    ## 2 M         0.0513 1.06  0.375  0.356 0.170    212

Create a graph that has at least two geom layers.

``` r
ggplot(df, aes(x=diagnosis, y=mean, fill=diagnosis)) +
  geom_col(show.legend = F) +
  geom_errorbar(aes(ymin=mean-sd, ymax=mean+sd), width=0.5) +
  labs(title="Average largest compactness of nuclei",
       y="Average value")
```

![](mini-project-2_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

### 1.3 (2 points)

Based on the operations that you’ve completed, how much closer are you
to answering your research questions? Think about what aspects of your
research questions remain unclear. Can your research questions be
refined, now that you’ve investigated your data a bit more? Which
research questions are yielding interesting results?

Based on the results above, we found that the malignant cancer has a
higher mean radius of nuclei, higher largest compactness of nuclei and
higher mean symmetry of nuclei present in sample image. Which are closer
to answering the research question 1, question 3 and question 4. But we
are not able to answer question 2, we can refine the question 2 to: Is
the smoothness_mean positively associated with the compactness_mean?

# Task 2: Tidy your data

In this task, we will do several exercises to reshape our data. The goal
here is to understand how to do this reshaping with the `tidyr` package.

A reminder of the definition of *tidy* data:

- Each row is an **observation**
- Each column is a **variable**
- Each cell is a **value**

### 2.1 (2 points)

Based on the definition above, can you identify if your data is tidy or
untidy? Go through all your columns, or if you have \>8 variables, just
pick 8, and explain whether the data is untidy or tidy.

Based on the definition above, I think our data is tidy. Because each
row represents an observation, which in this case is a cancer sample.
Each column represents a variable, such as ID, diagnosis, radius_mean,
texture_mean, perimeter_mean, area_mean, smoothness_mean,
compactness_mean, and so on. The cells within the table contain the
corresponding values for each variable.

### 2.2 (4 points)

Now, if your data is tidy, untidy it! Then, tidy it back to it’s
original state.

If your data is untidy, then tidy it! Then, untidy it back to it’s
original state.

Be sure to explain your reasoning for this task. Show us the “before”
and “after”.

Untidy the data from long wide format to long format.

``` r
data(cancer_sample)
cancer_sample_undity <- cancer_sample %>% 
  pivot_longer(cols=radius_mean:fractal_dimension_worst,
               names_to="metric", values_to="value")
head(cancer_sample_undity)
```

    ## # A tibble: 6 × 4
    ##       ID diagnosis metric              value
    ##    <dbl> <chr>     <chr>               <dbl>
    ## 1 842302 M         radius_mean        18.0  
    ## 2 842302 M         texture_mean       10.4  
    ## 3 842302 M         perimeter_mean    123.   
    ## 4 842302 M         area_mean        1001    
    ## 5 842302 M         smoothness_mean     0.118
    ## 6 842302 M         compactness_mean    0.278

Tidy it back to it’s original state.

``` r
cancer_sample_undity %>% 
  pivot_wider(names_from="metric", values_from = "value") %>%
  head()
```

    ## # A tibble: 6 × 32
    ##         ID diagnosis radius_mean texture_mean perimeter_mean area_mean
    ##      <dbl> <chr>           <dbl>        <dbl>          <dbl>     <dbl>
    ## 1   842302 M                18.0         10.4          123.      1001 
    ## 2   842517 M                20.6         17.8          133.      1326 
    ## 3 84300903 M                19.7         21.2          130       1203 
    ## 4 84348301 M                11.4         20.4           77.6      386.
    ## 5 84358402 M                20.3         14.3          135.      1297 
    ## 6   843786 M                12.4         15.7           82.6      477.
    ## # ℹ 26 more variables: smoothness_mean <dbl>, compactness_mean <dbl>,
    ## #   concavity_mean <dbl>, concave_points_mean <dbl>, symmetry_mean <dbl>,
    ## #   fractal_dimension_mean <dbl>, radius_se <dbl>, texture_se <dbl>,
    ## #   perimeter_se <dbl>, area_se <dbl>, smoothness_se <dbl>,
    ## #   compactness_se <dbl>, concavity_se <dbl>, concave_points_se <dbl>,
    ## #   symmetry_se <dbl>, fractal_dimension_se <dbl>, radius_worst <dbl>,
    ## #   texture_worst <dbl>, perimeter_worst <dbl>, area_worst <dbl>, …

### 2.3 (4 points)

Now, you should be more familiar with your data, and also have made
progress in answering your research questions. Based on your interest,
and your analyses, pick 2 of the 4 research questions to continue your
analysis in the remaining tasks:

I will choose the following two task:

1.  *Is there any difference in the average symmetry_mean between the
    malignant cancer and benign cancer?*

2.  *Is there any difference in the average compactness_worst between
    the malignant cancer and benign cancer?*

Explain your decision for choosing the above two research questions.

<!--------------------------- Start your work below --------------------------->

I choose the two questions because we can answer these questions based
on the calculation and visualization above.
<!----------------------------------------------------------------------------->

Now, try to choose a version of your data that you think will be
appropriate to answer these 2 questions. Use between 4 and 8 functions
that we’ve covered so far (i.e. by filtering, cleaning, tidy’ing,
dropping irrelevant columns, etc.).

(If it makes more sense, then you can make/pick two versions of your
data, one for each research question.)

<!--------------------------- Start your work below
--------------------------->

``` r
df <- cancer_sample %>% 
  select(symmetry_mean, compactness_worst, diagnosis)
```

# Task 3: Modelling

## 3.0 (no points)

Pick a research question from 1.2, and pick a variable of interest
(we’ll call it “Y”) that’s relevant to the research question. Indicate
these.

<!-------------------------- Start your work below ---------------------------->

**Research Question**: *Is there any difference in the average
compactness_worst between the malignant cancer and benign cancer?*

**Variable of interest**: compactness_worst

<!----------------------------------------------------------------------------->

## 3.1 (3 points)

Fit a model or run a hypothesis test that provides insight on this
variable with respect to the research question. Store the model object
as a variable, and print its output to screen. We’ll omit having to
justify your choice, because we don’t expect you to know about model
specifics in STAT 545.

- **Note**: It’s OK if you don’t know how these models/tests work. Here
  are some examples of things you can do here, but the sky’s the limit.

  - You could fit a model that makes predictions on Y using another
    variable, by using the `lm()` function.
  - You could test whether the mean of Y equals 0 using `t.test()`, or
    maybe the mean across two groups are different using `t.test()`, or
    maybe the mean across multiple groups are different using `anova()`
    (you may have to pivot your data for the latter two).
  - You could use `lm()` to test for significance of regression
    coefficients.

<!-------------------------- Start your work below ---------------------------->

``` r
model <- lm(compactness_worst~diagnosis, df)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = compactness_worst ~ diagnosis, data = df)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.32351 -0.08157 -0.01507  0.05113  0.68318 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 0.182673   0.006723   27.17   <2e-16 ***
    ## diagnosisM  0.192152   0.011014   17.45   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.127 on 567 degrees of freedom
    ## Multiple R-squared:  0.3493, Adjusted R-squared:  0.3481 
    ## F-statistic: 304.3 on 1 and 567 DF,  p-value: < 2.2e-16

<!----------------------------------------------------------------------------->

## 3.2 (3 points)

Produce something relevant from your fitted model: either predictions on
Y, or a single value like a regression coefficient or a p-value.

- Be sure to indicate in writing what you chose to produce.
- Your code should either output a tibble (in which case you should
  indicate the column that contains the thing you’re looking for), or
  the thing you’re looking for itself.
- Obtain your results using the `broom` package if possible. If your
  model is not compatible with the broom function you’re needing, then
  you can obtain your results by some other means, but first indicate
  which broom function is not compatible.

<!-------------------------- Start your work below ---------------------------->

``` r
broom::tidy(model)
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic   p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)    0.183   0.00672      27.2 9.79e-105
    ## 2 diagnosisM     0.192   0.0110       17.4 7.07e- 55

<!----------------------------------------------------------------------------->

# Task 4: Reading and writing data

Get set up for this exercise by making a folder called `output` in the
top level of your project folder / repository. You’ll be saving things
there.

## 4.1 (3 points)

Take a summary table that you made from Task 1, and write it as a csv
file in your `output` folder. Use the `here::here()` function.

- **Robustness criteria**: You should be able to move your Mini Project
  repository / project folder to some other location on your computer,
  or move this very Rmd file to another location within your project
  repository / folder, and your code should still work.
- **Reproducibility criteria**: You should be able to delete the csv
  file, and remake it simply by knitting this Rmd file.

<!-------------------------- Start your work below ---------------------------->

``` r
write.csv(dt, file="output/table.txt")
```

<!----------------------------------------------------------------------------->

## 4.2 (3 points)

Write your model object from Task 3 to an R binary file (an RDS), and
load it again. Be sure to save the binary file in your `output` folder.
Use the functions `saveRDS()` and `readRDS()`.

- The same robustness and reproducibility criteria as in 4.1 apply here.

<!-------------------------- Start your work below ---------------------------->

``` r
saveRDS(model, file="model.RDS")
model <- readRDS("model.RDS")
```

<!----------------------------------------------------------------------------->

# Overall Reproducibility/Cleanliness/Coherence Checklist

Here are the criteria we’re looking for.

## Coherence (0.5 points)

The document should read sensibly from top to bottom, with no major
continuity errors.

The README file should still satisfy the criteria from the last
milestone, i.e. it has been updated to match the changes to the
repository made in this milestone.

## File and folder structure (1 points)

You should have at least three folders in the top level of your
repository: one for each milestone, and one output folder. If there are
any other folders, these are explained in the main README.

Each milestone document is contained in its respective folder, and
nowhere else.

Every level-1 folder (that is, the ones stored in the top level, like
“Milestone1” and “output”) has a `README` file, explaining in a sentence
or two what is in the folder, in plain language (it’s enough to say
something like “This folder contains the source for Milestone 1”).

## Output (1 point)

All output is recent and relevant:

- All Rmd files have been `knit`ted to their output md files.
- All knitted md files are viewable without errors on Github. Examples
  of errors: Missing plots, “Sorry about that, but we can’t show files
  that are this big right now” messages, error messages from broken R
  code
- All of these output files are up-to-date – that is, they haven’t
  fallen behind after the source (Rmd) files have been updated.
- There should be no relic output files. For example, if you were
  knitting an Rmd to html, but then changed the output to be only a
  markdown file, then the html file is a relic and should be deleted.

Our recommendation: delete all output files, and re-knit each
milestone’s Rmd file, so that everything is up to date and relevant.

## Tagged release (0.5 point)

You’ve tagged a release for Milestone 2.

### Attribution

Thanks to Victor Yuan for mostly putting this together.