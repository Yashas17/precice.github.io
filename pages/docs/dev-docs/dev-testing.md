---
title: Running and writing tests
keywords: pages, development, tests
permalink: dev-docs-dev-testing.html
---

## Running

Use `ctest` (or `make test`) to run all test groups and `mpirun -np 4 ./testprecice` to run individual tests.

Some important options for `ctest` are:

- `-R petsc` to run all tests groups matching `petsc`
- `-E petsc` to runs all tests groups not matching `petsc`
- `-VV` show the test output
- `--output-on-failure` show the test output only if a test fails

To run individual tests, please run `mpirun -np 4 ./testprecice` directly. Examples:

Some important options for `./testprecice` are:

- `--report_level` or `-r` with options `confirm|short|detailed|no`
- `--run_test` or `-t` with a unit test filter.
- `--[no_]color_output` or `-x[bool]` to enable or disable colored output.
- `--log_level=<all|success|test_suite|unit_scope|message>` to control the verbosity. This defaults to the value of the ENV `BOOST_TEST_LOG_LEVEL`.

Examples:

- `mpirun -np 4 ./testprecice -x` runs boost test with colored output.
- `mpirun -np 4 ./testprecice -x` runs boost test with colored output.
- `mpirun -np 4 ./testprecice -x -r detailed -t "+/+PetRadial+"` (replace all + by *, due to Doxygen) runs all `PetRadial\*` tests from all test suites using colored output and detailed reporting.

## Writing

To learn, how to write new unit tests, have a look at `src/testing/tests/ExampleTests.cpp`. Most of the rules below also apply to integration tests, but there are some important exceptions that you should keep in mind (see below).

Quick reference:

### Grouping tests

```cpp
BOOST_AUTO_TEST_SUITE(NameOfMyGroup)
BOOST_AUTO_TEST_SUITE_END()
```

### Starting tests

```cpp
BOOST_AUTO_TEST_CASE(NameOfMyTest)
{
  PRECICE_TEST(1_rank);
```

### preCICE test specification

Unit tests:

- Unit test case running on 1 rank: `PRECICE_TEST(1_rank);`
- Unit test case running on 2 ranks, with no intra-communication setup: `PRECICE_TEST(2_ranks);`
- Unit test case running on 2 ranks with intra-communication setup: `PRECICE_TEST(""_on(2_ranks).setupMasterSlaves());`
- Unit test case running on 2 ranks with intra-communication setup and events initialized: `PRECICE_TEST(""_on(2_ranks).setupMasterSlaves(), require::Events);`

Integrations tests:

- Integration test with Solver A on 1 rank and B on 2 ranks: `PRECICE_TEST("A"_on(1_rank), "B"_on(2_ranks));`
- Integration test with Solver A on 2 rank and B on 2 ranks: `PRECICE_TEST("A"_on(2_ranks), "B"_on(2_ranks));`
- Integration test with Solver A, B and C on 1 rank each: `PRECICE_TEST("A"_on(1_rank), "B"_on(1_rank), "C"_on(1_rank));`

### Test context

The [test context](https://precice.org/doxygen/develop/classprecice_1_1testing_1_1TestContext.html) provides context of the currently running test.
Inforamation is accessible directly and checkable as a predicate.
You can safely pass this per reference (`const precice::testing::TestContext&`) to other functions.

Attribute | Accessor | Predicate
Communicator size | `context.size` | `context.hasSize(2)`
Communicator rank | `context.rank` | `context.isMaster()`, `context.isRank(2)`
Participant name | `context.name` | `context.isNamed("A")`

In addition to this, you can also use the context to [connect the masters](https://precice.org/doxygen/develop/classprecice_1_1testing_1_1TestContext.html#a85f8b4146ceb4de0afdedee97c865c0f) of 2 partiticpants.

### Writing integration tests

If you are writing a new integration test there are the following important differences to unit tests:

- the preCICE integration tests are located under `./tests`
- each test goes into an individual `.cpp` file.
- suites are organized in folder hierarchies within `./tests`
- common functionality in a suite may be provided in a `helpers.cpp` file

Please use the script [createTest.py](https://github.com/precice/precice/blob/develop/tools/building/createTest.py) for the generation of a skeleton for a new test. It will take care of setting everything up in the required format. The documentation of the script is accessed by calling the `python3 createTest.py --help`.
As an example, refer to existing integration tests in `./tests`.
