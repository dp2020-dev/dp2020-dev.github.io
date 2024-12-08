---
layout: post
title: Using dbt expectations as part of a dbt build.
---

<i> The objective of the blog post is to give a practical overview of the data transformation testing tool Great Expectations/dbt expectations. </i>

### Why data testing?

Having been involved in data transformations in the past (e.g. moving data from on prem to the Azure cloud) I'm aware of the potential complexity of ensuring the quality of data from source to target, verifying the transformations at each stage and maintaining data integrity.

Given

### Great Expectations

[Great Expectations.io](https://greatexpectations.io/) and its open source version [dbt expectations](https://github.com/calogica/dbt-expectations) are frameworks that enable automated tests to be embedded in ingestion/transformation pipelines.

<GE Image>
![Great Expectations logo', December 2024](/images/gx_logo_horiz_color.png)

This is a widely used tool in data engineering, and in order to try it out and evaluate this tool, I undertook the following Udemy course, the screenshots and material are based on this:

[The Complete dbt (Data Build Tool) Bootcamp:](https://www.udemy.com/course/complete-dbt-data-build-tool-bootcamp-zero-to-hero-learn-dbt)
![dbt bootcamp](/images/dbtHeroUdemy.png)

This course covers the theory and practical application of a data project using snowflake as the data warehouse, and the open source version of dbt. What was particularly relevant for a tester are the sections covering dbt expectations<add link>. This post will explain at a high level what dbt expectations can do, how it can enable QA in a data ingestion/data transformation project rather than a hand on how to' guide.

### What is dbt expectations?

dbt expectations is an open source package for dbt based on Great Expectations, to enable testing in a data warehouse.

<b> How is it used to test, and why? </b>

Using the dbt expectations package allows data to be verified in terms of quality and accuracy at specific stages of the transformation process. It includes built in tests including not_null, unique etc. and custom tests written in sql which can extend test coverage (see /tests/no_nulls_in_dim_listings for example.)

When the package is imported etc. the tests are written in the schema.yml file. This is a breakdown of the examples in [/models/schema.yml](https://github.com/dp2020-dev/completeDbtBootcamp/blob/main/models/schema.yml):

#### Basic Expectations:

<li>not_null: Ensures that the column doesn't contain null values.</li>
<li>unique: Verifies that all values in the column are distinct.</li>

#### Relationship Expectations:

</li>relationships: Checks if a foreign key relationship exists between two columns in different models.</li>

#### Value-Based Expectations:

<li>accepted_values: Ensures that the column only contains specific values from a predefined list.</li>
<li>positive_value:</b> Verifies that the column values are positive numbers.</li>

#### Statistical Expectations:

<li>dbt_expectations. expect_table_row_count_to_equal_other_table: Compares the row count of two tables.</li>

</li>dbt_expectations.expect_column_values_to_be_of_type: Checks the data type of a column.</li>
<li>dbt_expectations.expect_column_quantile_values_to_be_between: Verifies that quantile values fall within a specific range.</li>
<li>dbt_expectations.expect_column_max_to_be_between: Ensures that the maximum value of a column is within a certain range.</li>

#### Example test:

Room_type, see screenshot.

To run the tests in the schema:
dbt test
To run a specific test:
dbt test --select model:dim_listings_cleansed

To debug, the standard tool is dbt test --debug, but the advice on the bootcamp course is to run the failing test to find the sql which failed.

In a specific example, the failing sql code is run directly against the table (in Snowflake in this example) to find where exactly the failure is.

### Lineage Graph (Data Flow DAG)

Source data in green -> dependencies

Select what types of elements to include in the graph, refresh to only show selection

To create: dbt docs generate
To open: dbt docs serve

Opens docs based on .md (confirm this)

For example:

The lineage graph shows the flow of data in our tata warehouse, for instance we can see at a glance that dim_listings_cleansed is a cleansed dimension table based on the src_listings table.
Figure 1 Lineage Graph png

By right clicking and checking documentation for dim_listings_cleansed, we can check all the tests we have in place for this stage of the transformation, for instance we can tell the the room_type test checks the type of room as per the description. While it takes some additional time to understand the how to set descriptions and how to link the schema.yml to the test files (I found I had to adhere closely to a set folder structure to get this to work), the benefit of having this lineage gr[ah and information are evidence- we can see what's tested where during the data transformation, and I feel it will save signicant time for someone picking up these tests to extend coverage/adapt.
