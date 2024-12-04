# ***Archival Notice***
This repository has been archived.

As a result all of its historical issues and PRs have been closed.

Please *do not clone* this repo without understanding the risk in doing so:
- It may have unaddressed security vulnerabilities
- It may have unaddressed bugs

<details>
   <summary>Click for historical readme</summary>

# Sandbox: External Tables

## Why

The purpose of this repository is to facilitate the discovering the requirements for bringing the functionality of the [dbt-external-tables dbt package](https://github.com/dbt-labs/dbt-external-tables) into dbt itself.

[dbt-adapters#92: making dbt-external-tables internal to Core](https://github.com/dbt-labs/dbt-adapters/discussions/92) spells out the motivation and initial design questions for this work.

To truly support External Tables as first-class citizens, it is not sufficient to simply copy the existing package macros into dbt. The major barriers are:

- there is no external_table materialization
- you can't `run` sources today in the way that External Tables require

## What

I've been prototyping how the above two points might be resolved, but to do so, I need examples of External Tables in the wild.

Fortunately, dbt-external-tables has a bootstrapped set of integration tests that cover the majority of functionality of the package across a variety of data platforms.

This repository begins as a fork of the dbt project contained within `dbt-external-tables`'s `integration_tests/` directory.

As I work on continue to prototype, I'll modify this project accordingly.

### Related Work

You'll notice hardly any macros in this repo related to External Tables. That's intentional! Instead, they're found in:

- [dbt-adapters#220](https://github.com/dbt-labs/dbt-adapters/pull/220)
- [dbt-snowflake#1058](https://github.com/dbt-labs/dbt-snowflake/pull/1058)

### Project Structure

Virtually all the external tables defined in this project is the same data.

To test a particular definition of an external table in dbt YAML:

1. define the table in a sources YAML that points to the rignt place in object storage
2. define tests on the table that confirm that data returned by the external table matches the data in `seeds/people.csv` (using the `dbt_utils.equality` macro)

Accordingly the repo is structured with the following key components:

1. `seeds/people.csv`: canonical version of table against which all the external tables will be compared
2. `public_data/`: the exact data as the `people` seed that:
   1. has already been uploaded for testing into:
      1. S3: `s3://dbt-external-tables-testing`
      2. GCS: `gs://dbt-external-tables-testing/`
   2. is partitioned (i.e. has many files in a folder structure)
   3. has two versions: `.csv` and `.json` file format
3. a sources YAML file with many external tables defined for each supported platform  (e.g. `models/plugins/snowflake/snowflake_external.yml`)

These integration tests confirm that, when staged as external tables, using different databases / file formats / partitioning schemes, the final combined output is equivalent to `seeds/people.csv`.

## How to Run

### Environment Setup

1. create local environment variables for all of what is mentioned in [`test.env.sample`](test.env.sample)
2. in your virtualenv run `pip install -r requirements.txt`, this will install development versions of both `dbt-adapters` and `dbt-snowflake` that are required for the external table functionality.
3. tell dbt to use the `profiles.yml` in this repo with `export DBT_PROFILES_DIR=/path/to/your/dbt/project`
4. run `dbt deps` to install dbt-utils, which is needed for the `dbt_utils.equality` macro
5. ensure you can connect to your desired data platform with `dbt debug --target snowflake` (or `bigquery`, `redshift`, etc.)

### Running the Tests

The below is for running against Snowflake, but you can substitute `snowflake` for `bigquery`, `redshift`, etc. as needed.

1. `dbt seed -t snowflake` to materialize the `people` seed
2. `dbt run -s path:models/plugins/snowflake -t snowflake`
3. `dbt test -s path:models/plugins/snowflake -t snowflake`


## Current Prototype Status

### v1: a new `dbt source refresh` command

My first prototype was to make a new dbt command `dbt source refresh` that would materialize external tables defined as sources. However, I quickly got out of my depth with dbt Task and Runners. There's too many pieces missing like:

1. sources can't have materializations when sources don't have `.sql` model files and can't be compiled
2. making a `source` runner requires a lot of refactoring to support things like grants, hooks, and contracts etc

Instead, I opted for an intermediate prototype in which I could continue unblocked.

### v2: a external tables are shell models that use `materialized: external_table`

This works and doesn't require changes to Core only dbt-adapters and the adapter repos themselves!

The big conceit here is that even though an external table is defined entirely in YAML just as in the package:

- the definition must be in a `model:` node not a `source:` node
- every model needs a `.sql` file with a trivial `SELECT` statement that is ignored

With the limitations, we get out of the box the ability to implement an external_table materialization with many of the same model features we know and love:

- `--select` logic
- `--full-refresh`
- actually should test more things...

### ongoing challenges

1. how can dbt figure out that an already existing External Table is an external table and not just a regular table?
2. how to handle the profligate use of the `jinja` `config` macro to cover the dozens of configuration options?



