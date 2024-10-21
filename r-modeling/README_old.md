# Modeling with R and Snowflake

## High Level Summary:

These notebooks use the `tidymodels` and `orbital` package to fit a model on a sample of Lending Club data, and then run inference *directly in Snowflake*. This is awesome because it allows for really really really fast inference, without having to mess with dependencies. It's literally pushing raw Snowflake SQL! It's also a great addition to our Snowflake story, as it shows pushing compute from platforms like Connect to Snowflake – more consumption for Snowflake, no downside to us!

**What's `orbital`?**

[`orbital`](https://orbital.tidymodels.org/index.html) is an incredible package that allows you to turn `tidymodels` `workflows`, *including* preprocessing and feature engineering steps, into objects that can be run against databases. Practically, it's converting a `workflow` into a series of [quosures](https://rlang.r-lib.org/reference/topic-quosure.html), that can then be translated into SQL by `dbplyr`. This means that a `workflow`, converted to `orbital`, can be run against data stored in a remote database, using that remote database's compute.

**Wait, why do dependencies matter?**

When running code remotely, dependencies are always a sore spot. Obviously there are lots of ways to accomplish this (ie Docker, Connect). However, the major data warehouse providers have done a particularly bad job in enabling users to manage dependencies. For a particularly bad example, see Snowflake's documentation on [Snowpark Python User-Defined Functions(UDFs)](https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs#using-third-party-packages-from-anaconda-in-a-udf). To get dependency packages, you have to install them from a special conda channel as part of your function code. What happens if the package I want to use isn't available? Too bad.

By converting models and preprocessing steps directly to SQL, `orbital` cuts all this out. Now, models can be evaluated natively, no dependency management required!

**How fast is it, really?**

Pretty dang fast! Running model inference on Snowflake has two key advantages:

1\. It gives access to Snowflake's scalable compute.

2\. It allows Snowflake's query optimizer to optimize the model fitting operation -- this is particularly important as it means that Snowflake itself will cut out unnecessary optimizations to maximize efficiency.

This speed can be particularly seen when operating on the lending_club dataset. In Snowflake, the lending_club dataset consists of 151 columns across 2.3m rows. And we can fit a model on it live in a demo!

**What's the difference between a model in Snowflake and a Vetiver API?**

Deploying a model to Snowflake (ie as a Snowflake View) is inherently a form of **Batch Inference**, as the model inference operation is happening on data that's already grouped into a table in a data warehouse. Vetiver APIs, on the other hand, are designed for **Realtime/Online Inference**. For a great summary of the differences between these two types of inference, and when you might choose one over the other, check out [this article](https://mlinproduction.com/batch-inference-vs-online-inference/).

#### Graphical Dataflow:

\![plot](snowflake_dataflow.png)

## Demo Assets:

1.  `fit_initial_model.qmd` -- This contains a demonstration of grabbing some data from Snowflake, fitting a simple linear model to predict interest rates, and then conducting inference on snowflake compute using the `orbital` R package. It shows common MLOps actions (like model versioning and tracking) using Vetiver and Connect. It also demonstrates taking an orbital object and converting it to a native Snowflake object.
2.  `refit_model.qmd` -- This is an example of using Connect as an MLOps monitoring/orchestration layer. This takes a model that already exists on Connect (created using `fit_initial_model.qmd`), evaluates its performance over time, attempts to refit the model, and, if the new model outperforms the existing model, it updates the Snowflake and Vetiver objects.

# Demoing using `fit_initial_model.qmd`

## When should I use this demo?

This is probably the easiest way to demo `orbital`'s capabilities. It shows an ad-hoc model fitting and registering workload. This is where I'd start with most customers.

## What do I need to run this demo?

Nothing special – it'll work with the same setup steps in the readme in the parent folder to this asset.

## How do I run this demo?

Steps:

1.  Set up Snowflake and clone the repo using the steps in the root README
2.  Open this project and restore the `renv`
3.  Create an .Renviron file containing `CONNECT_SERVER` (I recommend palm.ptd.posit.it) and `CONNECT_API_KEY`
4.  Edit the `model_name` variable in the first code cell.
5.  Run the notebook.
6.  Show the new Snowflake view working in Snowsight (aka the Snowflake UI). A great query to use is below:

### Querying an Orbital model saved as a Snowflake View:

You can run this SQL query in Snowsight. It's super fast!

``` sql
SELECT 
  a.id, 
  a.term, 
  a.loan_amnt,
  b.".pred" AS predicted_interest_rate 
FROM 
  LOAN_DATA a 
  LEFT JOIN PREDICTED_INTEREST_RATES b ON a.id = b.id 
WHERE 
  a.addr_state = 'PA'
```

# Demoing using `refit_model.qmd`

[Example of this asset running in the wild](https://pub.palm.ptd.posit.it/connect/#/apps/01384bce-f00a-4eed-9e97-88aaacd97b9a/access/90)

## When Should I Use This Demo?

The ideal audience for this demo asset is a customer/prospect who has expressed desire to learn more about ML **Orchestration** with Connect and Snowflake. This demo gives them a pattern of how that can be achieved, using scheduled (or render-on-demand!) notebooks in Connect.

## How do I run this demo?

Steps:

1.  Run all the steps to run `fit_initial_model.qmd` above.
2.  Change the `model_name` variable in the second code cell to match what you set in `fit_initial_model.qmd`
3.  Run the notebook locally to be sure it works.
4.  Deploy notebook to Connect. The notebook requires two envvars to be set in Connect:
5.  `SNOWFLAKE_ACCOUNT=duloftf-posit-software-pbc-dev` -- this is a simple reference to our Snowflake account name
6.  `SNOWFLAKE_SSH_KEY` -- this is a base64-encoded private key that controls access to our Snowflake service account. The value of this is stored [here](https://start.1password.com/open/i?a=GXMPN22RIVE37KFQRN5GJYYP7Y&v=r7ztssaettsjmj2tv6327a5tge&i=ubusb7gisysz76zcpbotkia2ai&h=positpbc.1password.com)

# Great Examples of Orbital Calls

[First Demo of `orbital` -- Jaguar Land Rover](https://us-1237.app.gong.io/call?id=4591800830105809843&account-id=1178919962027011919)

# Future work:

-   [x] Improve MLOps story

-   [x] Deploy inference notebook to Connect

-   [x] Add Connect orchestration – do inference via a Connect scheduled (or render-on-demand) notebook
