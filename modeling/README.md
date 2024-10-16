# Modeling in R and Snowflake

## High-Level Overview:

In this example, we use the `tidymodels` and `orbital` packages to train a machine learning model on Lending Club data and then run the inference *directly in Snowflake*. This is a game-changer for fast, efficient inference without the hassle of managing dependencies. The process generates raw SQL queries that Snowflake executes, showcasing Snowflake's compute power.

### What is `orbital`?

[`orbital`](https://orbital.tidymodels.org/index.html) is a powerful package that transforms `tidymodels` `workflows`—including preprocessing and feature engineering—into SQL-compatible objects that can be executed directly against databases. Essentially, it converts a `workflow` into [quosures](https://rlang.r-lib.org/reference/topic-quosure.html), which are then translated into SQL via `dbplyr`. This means that once a `workflow` is converted into an `orbital` object, it can be run using Snowflake’s compute, leveraging its infrastructure for both preprocessing and inference.

### Why Should You Care About Dependencies?

Managing dependencies can be one of the most painful aspects of running remote code.A prime example is the cumbersome process of managing third-party packages when using Snowflake's [Snowpark Python UDFs](https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs#using-third-party-packages-from-anaconda-in-a-udf), which relies on a limited conda channel. If the required package isn't available, you're out of luck.

By translating models and preprocessing steps into SQL, `orbital` bypasses these challenges entirely. This allows seamless execution of models in Snowflake, eliminating the need for dependency management!

### Performance: How Fast Is It?

It’s *really* fast! Running model inference in Snowflake offers two major advantages:

1. You tap into Snowflake’s scalable compute engine.
2. Snowflake’s query optimizer can enhance the performance of the model fitting process by trimming unnecessary operations and boosting efficiency.

These benefits are especially evident when working with larger datasets like Lending Club’s, which has 151 columns and 2.3 million rows. You can demonstrate this live by fitting a model in Snowflake in real-time!


## Demo Assets:

1.  `fit_initial_model.qmd` -- This contains a demonstration of grabbing some data from Snowflake, fitting a simple linear model to predict interest rates, and then conducting inference on snowflake compute using the `orbital` R package. It shows common MLOps actions (like model versioning and tracking) using Vetiver and Connect. It also demonstrates taking an orbital object and converting it to a native Snowflake object.
2.  `refit_model.qmd` -- This is an example of using Connect as an MLOps monitoring/orchestration layer. This takes a model that already exists on Connect (created using `fit_initial_model.qmd`), evaluates its performance over time, attempts to refit the model, and, if the new model outperforms the existing model, it updates the Snowflake and Vetiver objects.


## How do I run this demo?

Steps:

1. Set up the Posit Workbench Native App using the steps in the root README
2. Open Posit Workbench, start an RStudio Session and clone this repo
3. Install `renv` using `install.packages("renv")`
2. Open this project and restore th
3. Create an .Renviron file containing `CONNECT_SERVER` (I recommend palm.ptd.posit.it) and `CONNECT_API_KEY`
4. Edit the `model_name` variable in the first code cell.
5. Run the notebook.
6. Show the new Snowflake view working in Snowsight (aka the Snowflake UI). A great query to use is below:
