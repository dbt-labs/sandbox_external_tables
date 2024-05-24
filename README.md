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

## How (from OG readme.md)

The files in `public_data` are available in two public storage buckets:
- `s3://dbt-external-tables-testing`
- `gs://dbt-external-tables-testing/`

These integration tests confirm that, when staged as external tables, using different databases / file formats / partitioning schemes, the final combined output is equivalent to `seeds/people.csv`.
