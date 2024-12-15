---
layout: post
title: Using dbt-expectations as part of a dbt build.
---

<i> In order to evaluate and give an overview of dbt-expectations, this dbt project is based on a dbt data build course (more info below). The post gives a summary of the different types of data tests applied to the data transformation, with examples and explanations as to what's being tested and how.
</i>

### Why data testing?

From my experience with data transformation projects in the past (e.g. moving data from on prem to the Azure cloud) I'm aware of the challenges of ensuring the quality of data taken from multiple sources into target tables, the transformations at each stage and maintaining this quality continuously in a CI/CD delivery. This complexity makes manual testing onerous (especially given the transformations are likely to be part of an automated pipeline), and quality issues can erode the stakeholder's confidence in the end data.

In the context of these data testing challenges, [Great Expectations.io](https://greatexpectations.io/) and its open source version [dbt-expectations](https://github.com/calogica/dbt-expectations) are frameworks that enable automated tests to be embedded in data ingestion/transformation pipelines.

![Great Expectations logo, December 2024](/images/gx_logo_horiz_color.png)

In order to try it out and evaluate this tool, I undertook the following Udemy course, the screenshots and material are based on this course which uses Snowflake, a command line/Terminal interface and Git.

[The Complete dbt (Data Build Tool) Bootcamp:](https://www.udemy.com/course/complete-dbt-data-build-tool-bootcamp-zero-to-hero-learn-dbt) ![dbt bootcamp](/images/dbtHeroUdemy.png)

This course covers the theory and practical application of a data project using snowflake as the data warehouse, and the open source version of dbt. What was particularly relevant for a tester are the sections covering [dbt expectations](https://hub.getdbt.com/calogica/dbt_expectations/latest/). This post will explain what dbt expectations can do, alongside some practical examples of how it can be applied to a data transformation project.

## What is dbt-expectations?

dbt-expectations is an open source python package for dbt based on Great Expectations, and enables integrated tests in a data warehouse.

This allows us to extend the coverage of the dbt core built in tests) using a range of tests within the package. The examples below include the built in tests, dbt-expecations tests, these tests are written in the schema.yml file. This is a breakdown of the examples in [the schema file](https://github.com/dp2020-dev/completeDbtBootcamp/blob/main/models/schema.yml).

In addition to these tests captured in schema file, we also have customer sql tests (example below).

### Built-in dbt Tests:

<ul>
<li>not_null: Ensures that the column doesn't contain null values.</li>
<li>unique: Verifies that all values in the column are distinct.</li>
</ul>
<ul>
<li>relationships: Checks if a foreign key relationship exists between two columns in different models.</li>
</ul>
<ul>
<li>accepted_values: Ensures that the column only contains specific values from a predefined list.</li>
<li>positive_value:</b> Verifies that the column values are positive numbers.</li>
</ul>

### Built-in dbt-expectations Tests:

<ul>
<li>dbt_expectations. expect_table_row_count_to_equal_other_table: Compares the row count of two tables.</li>

<li>dbt_expectations.expect_column_values_to_be_of_type: Checks the data type of a column.</li>
<li>dbt_expectations.expect_column_quantile_values_to_be_between: Verifies that quantile values fall within a specific range.</li>
<li>dbt_expectations.expect_column_max_to_be_between: Ensures that the maximum value of a column is within a certain range.</li><br>
</ul>
#### Example dbt-expectations test:<br>

To apply dbt expectation tests, the code is added to the schema.yml file
, in the example below its used to check column type, expected values (including the quantile value to check values in the table are in an expected range), and a max value. We can also set if a failing test is a warning or an error.

![Great Expectations logo, December 2024](/images/dbtExpectSampleTests.png)

### Built-in custom sql Tests:

The third type of dbt test used in this project is a <b>custom sql test</b>.

This simple sql custom test checks the 'dim_listings_cleansed' table for any listings with < 1 night.

![Custom sql example- min nights](/images/dim_listings_min_nights.png)

Custom tests sit outside the dbt core and dbt-expectactions tests and can
extend test coverage to cover edge cases. They are also flexible in enabling ad hoc testing to investigate
scenarios, or to be part of the CI/CD pipeline- see an example of how we can trace the `dim_listings_min_nights` custom rest on the data lineage graph in the [lineage graph section.](#dag_lineage)

## Debugging<br>

To briefly summarise the practical side of dbt test, they can be run in the command line, and there is a
standard `dbt test --debug` command, but interestingly the course recommended more in depth debugging
via running the sql test against the source table (e.g. in our example in Snowflake) and simplifying the test code to find exactly where it failed- a good approach for a complex failure.

The documentation on how to run tests and debug is pretty clear and user friendly, see [About dbt debug command](https://docs.getdbt.com/reference/commands/debug)

## Lineage Graph (Data Flow DAG)<br>

In the section above we've looked at practical tests in dbt-expectations which can be embedded in the data transformation pipeline. These tests can be included on a really useful dbt feature, the 'lineage graph' alongside the source tables, dimension, fact tables etc. to show where and when the tests run, what table it relates to etc.

<a id="dag_lineage"></a>

![dbt lineage graph](/images/dbt-dag-3.png)

Provided test in question is included in the schema.yml and has a description value, it will be included in the correct part of the data transformation flow.

For example, the lineage graph below shows the flow of data in our data warehouse, for instance we can see at a glance that `dim_listings_cleansed` is a cleansed dimension table based on the `src_listings table`.

![dbt lineage graph right click](/images/lineage_right_click.png)

By right clicking and checking documentation for `dim_listings_cleansed`, we can check all the tests we have in place for this stage of the transformation, for instance we can tell the the `room_type` test checks the type of room as per the description.

![dbt docs](/images/docs_room_type_test.png)

For reference the test itself is a built in test in the [schema.yml](https://github.com/dp2020-dev/completeDbtBootcamp/blob/ebd7310c905f63a124e43aee2725aeab9a00f8d9/models/schema.yml#L21), and while the schema clearly lists all tests its great to be able to visualise where exactly this test sits in the data pipeline, what table(s) it references and we're able to click through to read its description and code cia the graph. In a data transformation with many sources/transformations I'm sure this tool would be invaluable.

** Summary **
