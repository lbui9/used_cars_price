# Assignment 9: How much for that car?

> ### Install `plotly`
>
> Before running the set-up code chunk for this assignment, make sure you have installed the `plotly` library by running this line of code in the RStudio Console: `install.packages("plotly")`

## Instructions

In this assignment you will develop a model to study the effect of different variables on price of used cars.

**Make sure to answer these questions using full sentences and paragraphs, and proper spelling and punctuation.**

Read the [About the dataset](#about-the-dataset) section to get some background information on the dataset that you'll be working with.
Each of the below [exercises](#exercises) are to be completed in the provided spaces within your starter file `car_prices.Rmd`.
Then, when you're ready to submit, follow the directions in the **[How to submit](#how-to-submit)** section below.


## About the dataset

This dataset was sourced from [Kulper (2008)][kulper-2008], who collected data on car prices from Kelly Blue Book for hundreds of used GM cars sold in 2005. When you run the setup code chunk, the dataset will be loaded into you environment and store in a variable called `car_prices`.

| Variable | Description |
| -------- | ----------- |
| Price    | Suggested retail price of the used 2005 GM car. |
| Mileage  | number of miles the car has been driven |
| Make     | manufacturer of the car such as Saturn, Pontiac, and Chevrolet |
| Model    | specific models for each car manufacturer such as Ion, Vibe, Cavalier |
| Trim (of car) | specific type of car model such as SE Sedan 4D, Quad Coupe 2D |
| Type     | body type such as sedan, coupe, etc. |
| Cylinder | number of cylinders in the engine |
| Liter    | a more specific measure of engine size |
| Doors    | number of doors |
| Cruise   | indicator variable representing whether the car has cruise control (1 = cruise) |
| Sound    | indicator variable representing whether the car has upgraded speakers (1 = upgraded) |
| Leather  | indicator variable representing whether the car has leather seats (1 = leather) |



## Exercises

**You may find it helpful to refer back to our earlier modeling assignment (Assignment 5 on modeling blood pressure) where we first encountered many of the methods we will use in this assignment.**

**You must answer written questions with full sentences (and paragraphs where appropriate).**

1.  Run the set-up code block at the start of the `car_prices.Rmd` answer file to load the `car_prices` dataset.

    The `car_prices` dataset contains both continuous and categorical variables.
    
    i. There are 2 continuous variables in this dataset. `Price` is one of them. What is the other? (Hint: there are many numerical variables, but most of them are categorical. Which of the numerical variables can contain any number, instead of one of a fixed set of numbers.)
    
    ii. A third variable, `Liter`, may look like it is continuous, but it is actually categorical. Let's create some scatter plots to look at the relationship between `Price` (the response variable, y) and two possible explanatory variables (`Liter` and the variable that you identified in part i.).
    
      To do this, follow these steps:
        
      * Pipe the dataset to the `pivot_longer()` function, where you should pivot the `Liter` variable and the continuous variable you just identified (not `Price`). This will transform these two columns into names and values columns (you can either name these something appropriate or just use the default names).
      
      * Then pipe this pivoted dataframe to ggplot, where you should make a scatter plot showing `Price` on the y-axis and the value column on the x-axis, and use `facet_wrap` to create subplots for the two pivoted columns by faceting over the key column (supply the `scales = "free_x"` argument to the `facet_wrap` function to scale each plot appropriately).
        
      Here is a code template to guide you:
        
      ```r
      car_prices %>%
        pivot_longer(...) %>%
        ggplot() +
        geom_point(mapping = aes(x = ..., y = ...)) +
        facet_wrap(~..., scales = "free_x") +
        labs(title="...")
      ```

      > When trying to create a chain of multiple functions like this, it's a good idea to go one step at a time and make sure the output of each step looks correct before writing the code for the next step in the chain. For example, make sure that `pivot_longer()` is working correctly before adding in the code to create the graph.
      
      You should note that while `Price` is correlated with `Liter`, all points occur at certain values of `Liter` (unlike the continuous variable in the other subplot). This makes `Liter` a categorical variable.
      
      However, because `Liter` is numeric and because it is correlated with `Price`, *we will treat `Liter` as a continuous variable* in our model for the purposes of this assignment.
    
    Commit your work at the end of this exercise.

2. Using the `lm` function, create a multivariate model with:
    * `Price` as the response variable (y)
    * `Liter` and the continuous variable that you identified in Exercise 1 as the two explanatory variables (x)
    
    This should be a "parallel slopes" model with no interaction term (i.e. you should separate the explanatory variables with a `+` symbol).
    
    Store this model in a new variable called `continuous_model`.
    
    Then report the model coefficients (i.e. slope and intercept) using the `tidy` function, and the R<sup>2</sup> value using the `glance` function.
    
    What does the R<sup>2</sup> value tell us about how good this model is at explaining variation in `Price`?
      
    > #### (Optional reading) But what about the p-value?
    >
    > The `glance` function reports a p-value for our linear regression model. You might be wondering what this means, what our null & alternative hypotheses are, and whether a p-value of < 0.05 means something "significant".
    >
    > Unfortunately, the p-value of a linear model does not tell us anything about how well our model fits the data. The null hypothesis of a linear regression is that there is no relationship between the explanatory and response variables. In other words, the null hypothesis is a horizontally flat line, and the p-value is the probability that the data come from such a model. Thus a significant p-value (p < 0.05) means that we can reject this null hypothesis and fit a linear model, but says nothing about how good our fitted linear model actually is (because the p-value is for the horizontal line, not for our actual model).
    >
    > We will learn about linear model p-values in the Advanced Inference module.
    
    Commit your work.
    
3. Let's try to visualize what our model looks like by creating a 3D scatter plot (unfortunately we cannot include it in our knitted PDF).
    
    Copy and paste this code into a new code chunk in your `car_prices.Rmd` file and run it (you may have to first install the `plotly` library by running this line in the RStudio Console: `install.packages("plotly")`):
    
    ```r
    # predict model plane over values
    lit <- unique(car_prices$Liter)
    mil <- unique(car_prices$Mileage)
    grid <- with(car_prices, expand.grid(lit, mil))
    d <- setNames(data.frame(grid), c("Liter", "Mileage"))
    vals <- predict(continuous_model, newdata = d)
    
    # form surface matrix and give to plotly
    m <- matrix(vals, nrow = length(unique(d$Liter)), ncol = length(unique(d$Mileage)))
    p <- plot_ly() %>%
      add_markers(
        x = ~car_prices$Mileage, 
        y = ~car_prices$Liter, 
        z = ~car_prices$Price, 
        marker = list(size = 1)
        ) %>%
      add_trace(
        x = ~mil, y = ~lit, z = ~m, type="surface", 
        colorscale=list(c(0,1), c("yellow","yellow")),
        showscale = FALSE
        ) %>%
      layout(
        scene = list(
          xaxis = list(title = "mileage"),
          yaxis = list(title = "liters"),
          zaxis = list(title = "price")
        )
      )
    if (!is_pdf) {p}
    ```
    
    The output should be a 3D plot! Each variable in the model forms an exis of the plot. The data is shown as blue points, and the model is shown as a yellow plane (the flat surface). This plane tells us the predicted price for any value of the 2 explanatory variables.
    
    By examining and rotating the 3D plot, how well does the model seem to fit the data? From the 3D graph, can you tell if the model is meeting the 3 assumptions of the linear model. Is it easier to see what's going on with a 2D univariate model or a 3D multivariate model?
    
    > ### The problem with dimensions
    > 
    > A univariate linear model (1 response & 1 explanatory variable), occupies a 2-D space in which the linear model is a straight line.
    >
    > For every additional explanatory variable that we add to the model, the number of dimensions also increases. With 3 dimensions we can still just about visualize the original data (although not as simply as with 2D!). But if we added another explanatory variable, we would need a 4 dimensional space (aka. hyperdimensional space) to plot the model (and the model would be a 4-D "plane", also known as a "hyperplane").
    >
    > Because we are not able to visualize anything over 3 dimensions, we need to create 2-D plots to inspect the performance of the model and see whether it meets the 3 assuptions of the linear model. No matter how many variables our model has, these 3 plots:
    > 
    > * observed vs. predicted
    > * residuals vs. observed
    > * Q-Q plot
    >
    > will always have 2 dimensions, which is why they are so useful.
    
    Commit your work.
    
4. Using the model from Exercise 2, `continuous_model`, calculate the predicted y values and residuals for every data point in the `car_prices` dataset. To do this, you will need to use the `add_predictions` and `add_residuals` functions that you learned about in the first modeling assignment.
    
    You should save the output dataframe (containing the new columns `pred` and `resid`) as a new variable called `continuous_df`.
    
    Commit your work.
      
5. Create an *observed vs. predicted plot* using `continuous_df`. Recall that this plots the actual y values (the "observed" y values, i.e. the real `Price` values) on the y-axis, versus the model's predicted y values (i.e. the `pred` column) on a scatter plot (using the `geom_point()` function).
    
    Don't forget to add a reference line to the scatter plot using `geom_abline`. In an observed vs predicted plot, the reference line has a slope of 1 and an intercept of 0. This is because if our model was perfect, then the predicted y values would be exactly equal to the observed y values, and so all the points would lie on a line with a slope of 1 that goes through the origin (0,0). `geom_abline` can be used with `slope` and `intercept` arguments - as these are fixed values, they do not need to go inside an aesthetic function.
    
    Also remember to add appropriate axis labels and a title. 
    
    Here is a code template to get you started:
    
    ```r
    continuous_df %>%
      ggplot() +
      geom_point(mapping = aes(x = ..., y = ...)) +
      geom_abline(slope = ..., intercept = ...) +
      labs(...)
    ```
    
    What does this graph tells about the linear model's assumption of linearity?
    
    Commit your work.
       
6. Create a *residual vs. predicted plot*. Recall that this plots the residuals (the `resid` column) on the y-axis and the predicted y values (i.e. the `pred` column) on the x-axis.
    
    Don't forget to add a reference line. In this case, the reference line is a horizontal line at *y = 0*.
      
    Also remember to add appropriate axis labels and a title. Here is a code template to get you started:
    
    ```r
    continuous_df %>%
      ggplot() +
      geom_point(...) +
      geom_hline(...) +
      labs(...)
    ```
     
    What does this graph tells about the linear model's assumption of constant variability in the residuals?
    
    Commit your work.
    
7. Create a *Q-Q plot* using the `geom_qq` and `geom_qq_line` geom functions (and remember to add an appropriate title - you can leave the axis labels as their default values for this graph). 
    
    Recall that the Q-Q plot shows where the residuals for each data point would be distributed if they were drawn from a theoretical normal distribution, versus the actual distribution of residuals. The `sample` argument in the `aes()` function should be the `resid` column.
    
    You can leave the default title and axis labels. Here is a code template to get you started:
    
    ```r
    continuous_df %>%
      ggplot() +
      geom_qq(aes(sample = ...)) +
      geom_qq_line(aes(sample = ...))
    ```
      
    What does this graph tell you about the linear model's assumption that the residuals are nearly normally distributed?
    
    Commit your work.

8. 
    We might be able to improve our model by adding more explanatory variables. However the remaining variables are cateogrical, and many of them are not numerical. This presents 2 problems:
    
    * Visualizing the relationship between a continuous response and a categorical (non-numeric) explanatory variable.
    
    * Adding a non-numeric variable as a predictor in our model.
    
    In this exercise, we will look at the first problem: visualization.
    
    When we have a categorical explanatory variable, it is often a good idea to use *box plots* to visualize the distribution in each category.
    
    Copy this code into your Rmd file to create box plots of the differences in price between different values of the `Make` variable:  
    
    ```r
    car_prices %>%
      ggplot() +
      geom_boxplot(aes(x = Make, y = Price)) +
      labs(x = "Make of car", title = "Effect of make of car on price")
    ```
    
    It will be easier to see what is happening if we order the categories. To do so, replace `x=Make` with `x = reorder(Make, Price, FUN=median)`. This will order the categories of `Make` by the median of `Price` in each category.
    
    How should you interpret a box plot? (Note that the box plot in this image has been rotated 90 degrees compared to your box plot - the "x axis" here corresponds to `Price` in your plot)
    
    <img src="https://github.com/mason-cds-intro-comput-sci/assignment-9-car-prices-starter/blob/master/img/boxplot.png?raw=true" width="550">
    
    * The box in the center shows the middle 50% of the data points (also known as the "interquartile range", IQR).
    
    * The line in the middle of the box is the median.
    
    * Any data point that has a value of `Price` that is more than <i>1.5 * IQR</i>  is classified as an outlier and drawn as a circle.
    
    Using the boxplot of the `Make` and `Price` variables that you created in your Rmd file, *answer the following questions*:
    
    i. Which make of car has the lowest median price?
    
    ii. Which make of car has the greatest interquartile range of prices?
    
    iii. Which makes of cars have outliers?
    
    Commit your work.
    
9.  Create box plots of the remaining categorical variables in the `car_prices` dataset (you can omit `Liter`, which we are treating as continuous). There are two methods to do this (you can pick either):

    * The labor intensive method is to create a separate graph for every variable.
    
    * A more elegant solution is to use the `pivot_longer` function to collect all the categorical variables into *names* and *values* columns, and then facet over the newly created key column. Here is a code template to get you started:
    
        ```r
        car_prices %>%
          pivot_longer(
            cols = ..., 
            names_to="name", 
            values_to="value", 
            values_transform = list(value = 'factor')
            ) %>%
          ggplot() +
          geom_boxplot(aes(x = ..., y = ...)) +
          facet_wrap(~name, scales = "free_x") +
          labs(title = "...")
        ```
        
        * Remember to reorder the box plots by the median of `Price`, as we did in the previous exercise.
        
        * You should also increase `fig.width` for the code chunk to increase the graph size (a value of 9 is usually good), and you may also wish to add the `fig.asp` option to change the aspect ratio (try a value of 1.5 or 2, and then adjust until it looks good when you knit to a PDF).
        
        * You may wish to rotate the overlapping x-axis labels by adding this function to your graph code:
        
            ```r
            ... +
              theme(axis.text.x = element_text(angle = 45))
            ```
            
            However there several of the boxplots will have too many labels to visualizae properly - that's fine here, you can let those labels overlap in this exercise.
        
        * Note that we need to specify a type of data to convert all the values into (the categorical aka *factor* data type), since the columns that we are pivoting contain different types of data. 
    
    Commit your work.

10. From these box plots, we can see that some of the categorical variables seem to have much more effect on car price (i.e. the box plots are more differentiated in these variables).

    In particular, the `Make`, `Type`, and `Cylinder` variables look like promising candidates to add to our model. However, first we need to make a small change to the `Cylinder` column.
    
    In our first model, we treated `Liter` as a continuous variable, even though it was actually categorical. We could do the same with the `Cylinder` variable, but in the box plot you created in the previous question it does not look like there is a linear relationship between `Cylinder` and `Price`. Therefore we will tell R that `Cylinder` is not a continuous number, but actually a categorical number (which R calls a *factor*).
    
    To do this, and the following code chunk to your RMarkdown file and run it to create the new `cars_factor_df`:
    
    ```{r}
    cars_factor_df <- car_prices %>%
      mutate(Cylinder = as.factor(Cylinder))
    ```
    
    i. Now create a linear model using 5 explanatory variables: `Mileage`, `Liter`, `Cylinder`, `Make`, & `Type`. Make sure to use the new `cars_factor_df` dataframe in the `lm` function. 
    
      Store the new model in a variable called `mixed_model`.
        
    ii. Calculate (and show) the model coefficients (slopes and intercept) using the `tidy` function.
    
      Take a look at the coefficients for the categorical variables. You should observe that there are actually "slopes" for (almost) all the categories in each of the categorical variables.
        
      For example, the `Liter` variable has a single slope because we treated it as a continuous variable. However the `Cylinder` variable actually has two coefficients: `Cylinder6` and `Cylinder8`. Each of these give the effect of that particular value of the variable (6 or 8) on the predicted price.
      
      > ### What happened to 4 cylinder cars?
      >
      > Internally, R converts the column of `Cylinder` values into two columns which contains either a 1 or a 0. A value of 1 in the `Cylinder8` column means that this car has 8 cylinders, 0 means it does not.
      >
      > What happened to `Cylinder4` (the possible other value of `Cylinder` was 4)? Well, we don't need it because it would be redundant. If a car has neither 6 or 8 cylinders (i.e. it has a 0 in both the `Cylinder6` and the `Cylinder8` columns), then logically it must be the only remaining category. Therefore, we always need one fewer new columns that the number of categories in the original factor.
    
    iii. Calculate (and show) the R<sup>2</sup> value using the `glance()` function.
    
    Commit your work.


11. For this new model:

    i. Extend the `cars_factor_df` dataframe to add columns holding predictions and residuals calculated using the `mixed_model` model from Exercise 10, and save it in a new variable called `mixed_df`.
    
    ii. Using `mixed_df`, create an observed vs. predicted plot.
    
    iii. Using `mixed_df`, create a residuals vs. predicted plot.
    
    iv. Using `mixed_df`, create a Q-Q plot.
    
    Commit your work.
    
12.
    i. How does this mixed model compare to the simpler 2 variable model that we created earlier in the assignment? Consider both how well each model explains variation in `Price` (R<sup>2</sup>) and whether either model violates any of the linear model's three assumptions. Refer back to you graphs and calculations as necessary.

    ii. Which model would you prefer to use if you were picking a car? Justify your conclusion using your answers to part (i) of this exercise.
    
    Commit your work.


## Submitting

To submit your assignment, follow the two steps below.

1.  Save, commit, and push your completed RMarkdown file so that everything is synchronized to GitHub.
    If you do this right, then you will be able to view your completed file on the GitHub website.

2.  Knit your RMarkdown document to PDF format, export (download) the PDF file from RStudio Server, and then upload it to *Assignment 9* posting on Blackboard.


## Cheatsheets

You are encouraged to review and keep the following cheatsheets handy while working on this homework:

*   [Data transformation cheatsheet][data-transformation-cheatsheet]

*   [Data import cheatsheet][data-import-cheatsheet]

*   [ggplot2 cheatsheet][ggplot2-cheatsheet]

*   [RStudio cheatsheet][rstudio-cheatsheet]

*   [RMarkdown cheatsheet][rmarkdown-cheatsheet]

*   [RMarkdown reference][rmarkdown-reference]

[kulper-2008]:                   https://doi.org/10.1080/10691898.2008.11889579
[ggplot2-cheatsheet]:             https://github.com/rstudio/cheatsheets/raw/master/data-visualization-2.1.pdf
[rstudio-cheatsheet]:             https://github.com/rstudio/cheatsheets/raw/master/rstudio-ide.pdf
[rmarkdown-reference]:            https://www.rstudio.com/wp-content/uploads/2015/03/rmarkdown-reference.pdf
[rmarkdown-cheatsheet]:           https://github.com/rstudio/cheatsheets/raw/master/rmarkdown-2.0.pdf
[data-import-cheatsheet]:         https://github.com/rstudio/cheatsheets/raw/master/data-import.pdf
[data-transformation-cheatsheet]: https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf
