# Ledger Traffic Engine

This readme explains the working and the usage of Ledger Traffic Engine (LTE
test tool.


- What is the LTE Test Tool
- How to Run the Tests
- Where to View the Results

## What is the LTE Test Tool

The LTE test tool is test harness that engages the Ledger APIs and benchmarks
the performance of the Ledger component. It contains the framework for creating
and managing chains, for transaction submission, block creation, and block
commit. It conducts benchmark tests for insert and read-write transactions
(transactions per second) and contains scripts for launching the benchmarks. An
insert benchmark followed by a readwrite benchmark on the same dataset is
considered to constitute a single test-run.


## How to Run The tests

### Prerequisites

LTE assumes that the Hyperledger Fabric repo is present in the `$GOPATH` and
the necessary prerequisites for Hyperledger Fabric are met. For more details,
please refer to the Fabric getting-started guide [here](http://hyperledger-fabric.readthedocs.io/en/release/getting_started.html).

### Run Tests

To run all the available tests, run:
```
cd fabric-test/tools/LTE/scripts
./runbenchmarks.sh -f parameters_daily_CI.sh all
```
where the file `parameters_daily_CI.sh` has all the necessary test parameters.

To run a single test, named *varyNumChains* (test explained below), run:
```
cd fabric-test/tools/LTE/scripts
./runbenchmarks.sh -f parameters_daily_CI.sh varyNumChains
```

You can run individual tests without running all the available tests by giving
the name of the test as parameter, instead of `all`. You can get the available
test names by:
```
./runbenchmarks.sh help
```

### What the Tests Do

LTE copies itself in the Fabric repo before running the tests. Each test reads
test parameters from the provided parameter file and conducts several test-runs
by varying one or two of the parameters. The parameters are:
* number of chains (ledger),
* number of parallel transactions in each chain,
* number of Key-value pairs,
* number of transactions,
* number of keys to be read in each transaction,
* number of keys to be written in each transaction,
* size of batch for ledger,
* size of Key-value

For example, the *varyNumChains* test reads the parameters and varies the
number of chains for each test-run while keeping the other parameters constant,
and generate result. Varying a specific parameter for each test-runs gives
insight into the parameter's influence on the Ledger performance.

Each test-run consists of two phases: benchmarking of ledger insert phase and
benchmarking of ledger read-write phase.

#### Insert Phase

The insert benchmark starts fresh chains and inserts the Key-values by
simulating writes-only transactions For each of the chains. It launches the
parallel clients (based on the configuration) and the clients simulate and
commit write-only transactions. The keys are divided among clients such that
one key is written only once and all the keys are inserted. For instance, if
this benchmark is invoked with the following parameters:
```
Number of chains=2,
Number of parallel transactions per chain=2,
Number of key-value pairs=100
```
then client_1 on chain_1 will insert Key_1 to key_25 and client_2 on chain_1 will
insert Key_26 to key_50. Similarly client_3 on chain_2 will insert Key_1 to
key_25 and client_4 on chain_2 will insert Key_26 to key_50 where, client_1 and
client_2 run in parallel on chain_1 and client_3 and client_4 run in parallel
on chain_2.

#### Read-write Phase

Subsequently, the Read-write benchmark step opens the existing chains and
modifies the Key-values by simulating read-write transactions For each of the
chains. It launches the parallel clients (based on the configuration) and the
clients simulate and commit read-write transactions. This test assumes the
pre-populated chain by previously running insert benchmark step (described
above). Each transaction simulated by this benchmark randomly selects a
configurable number of keys and modifies their values. For instance, if this
benchmark is invoked with the following test parameters:
```
Number of chains=2,
Number of parallel transactions per chain=2,
Number of key-value pairs=100,
Number of total transactions=200
```
then client_1, and client_2 both execute 50 transactions on
chain_1 in parallel.  Similarly, client_3, and client_4 both execute 50
transactions on chain_2 in parallel In each of the transactions executed by any
client, the transaction expects and modifies any key(s) between Key_1 to key_50
(because, total keys are to be 100 across two chains).

### Running with Custom Parameters

The tests can be run with user-defined parameters by creating a new file that
has all the necessary parameters to run and using that file as the input (see
the section on how to run the tests) . The names of necessary parameters can be
found in the file `parameters_daily_CI.sh`.

### Running with optional CouchDB

By default, the tests use golveldb as the state database. Fabric provides the
option of using CouchDB as a pluggable state database. To run the existing
tests with CouchDB, use the parameter file `parameters_couchdb_daily_CI.sh`:
```
./runbenchmarks.sh -f parameters_couchdb_daily_CI.sh all
```
Note that this parameter file (`parameters_couchdb_daily_CI.sh`) contains the
following line, which is required to run the tests with CouchDB:
```
export useCouchDB="yes"
```
CouchDB can store values in JSON or binary formats. The following option in
`parameters_couchdb_daily_CI.sh` is used to switch between JSON and binary
values:
```
UseJSONFormat="true"
```

## How to View the Test Results

The test results can be viewed as in the stdout where it shows how long each
single operation took to complete in a test. These results are also saved in a
.csv file in the following directory: `/tmp/experiments`

## Troubleshooting

If the test stops with an error message
```
panic: This benchmark should be called with N=1 only. Run this with more volume
of data
```
It means that the test duration was too short to consider it as a valid
benchmark test. This usually happens when the test device has high processing
power and/or the parameter values used for the test are too small. To fix
this, please increase the values in the parameter file you are using. For
example, if your test uses `parameters_daily_CI.sh`, increase the value of
`NumTotalTx` from 100000 to 1000000.
