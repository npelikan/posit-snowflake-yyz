# Modeling in R and Snowflake

## High-Level Overview:

In this example, we use the `tidymodels` and `orbital` packages to train a machine learning model on [Lending Club loan data](https://www.kaggle.com/datasets/wordsforthewise/lending-club) and then run the inference *directly in Snowflake*. This is a game-changer for fast, efficient inference without the hassle of managing dependencies. The process generates raw SQL queries that Snowflake executes, showcasing Snowflake's compute power.


### What is `orbital`?

[`orbital`](https://orbital.tidymodels.org/index.html) is a powerful package that transforms `tidymodels` `workflows`—including preprocessing and feature engineering—into SQL-compatible objects that can be executed directly against databases. Essentially, it converts a `workflow` into [quosures](https://rlang.r-lib.org/reference/topic-quosure.html), which are then translated into SQL via `dbplyr`. This means that once a `workflow` is converted into an `orbital` object, it can be run using Snowflake’s compute, leveraging its infrastructure for both preprocessing and inference.


### Why Should You Care About Dependencies?

Managing dependencies can be one of the most painful aspects of running remote code. A prime example is the cumbersome process of managing third-party packages when using [Snowpark Python UDFs](https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs#using-third-party-packages-from-anaconda-in-a-udf), which relies on a limited conda channel. If the required package isn't available, you're out of luck. And there's no way to manage R dependencies in Snowflake!

By translating models and preprocessing steps into SQL, `orbital` bypasses these challenges entirely. This allows seamless execution of models in Snowflake, eliminating the need for dependency management!


### Performance: How Fast Is It?

It’s *really* fast! Running model inference in Snowflake offers two major advantages:

1. You tap into Snowflake’s scalable compute engine.
2. Snowflake’s query optimizer can enhance the performance of the model inference process by trimming unnecessary operations and boosting efficiency.

These benefits are especially evident when working with larger datasets like Lending Club’s, which has 151 columns and 2.3 million rows. You can demonstrate this live by fitting a model in Snowflake in real-time!


## Demo Assets:

`fit_model.qmd` -- This contains a demonstration of grabbing some data from Snowflake, fitting a simple linear model to predict interest rates, and then conducting inference on snowflake compute using the `orbital` R package. It shows common MLOps actions (like model versioning and tracking) using Vetiver and Connect. It also demonstrates taking an orbital object and converting it to a native Snowflake view.

## How do I run this demo?

**Fitting your First `orbital` model:**

1. Set up the Posit Workbench Native App using the steps in the root README
2. Start an RStudio Session
3. Install `renv` by running `install.packages("renv")` in the R console
4. Open the project by clicking on modeling.Rproj
5. Restore the packages needed by running the following commands in the R console:
    1. `renv::init()`
    2. `renv::restore()`
6. Open `fit_initial_model.qmd`, and run!
7. Head to Snowsight, and run the following query to see your model inference in action!
    ```sql
    SELECT 
        a.id, 
        a.term, 
        a.loan_amnt,
        ROUND(b.".pred", 5) AS predicted_interest_rate
    FROM 
        LOAN_DATA a 
        LEFT JOIN "r-interest-rate_latest" b ON a.id = b.id 
    WHERE 
        a.addr_state = 'PA' AND b.".pred" IS NOT NULL;
    ```

**Versioning and Deploying the model as an API to Posit Connect**
1. Login to your Posit Connect Server (it's in my slides!)
2. Click your username in the top right and generate an API key
3. Create a file, named .Renviron with the following content:
    ```
    CONNECT_SERVER=<the url of your Posit Connect Server>
    CONNECT_API_KEY=<your API key>
    ```
4. Uncomment the commented code cells in `fit_model.qmd`, and re-run!
