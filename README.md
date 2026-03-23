# SOM Experiment Benchmarks

Large-scale benchmarks for [SOM (Simple Object Machine)](https://som-st.github.io/) implementations.
Each benchmark runs 16 standard SOM benchmarks in a different shuffled order within a single invocation, designed to evaluate JIT compilation behavior under diverse workloads.

## Structure

```
Experiment/
  Experiment1.som ... Experiment20.som   # Shuffled-order, single-pass execution
  ExperimentRandom.som                   # Randomized order variant
ExperimentStable/
  ExperimentStable.som                   # Base stable experiment
  ExperimentStable1.som ... ExperimentStable20.som  # Iterated execution with timing
```

### Experiment (shuffle)

Each `ExperimentN` runs the same 16 benchmarks in a unique shuffled order.
A single invocation executes all benchmarks once and prints the total time in milliseconds.

### ExperimentStable

Each `ExperimentStableN` runs the same shuffled benchmark set repeatedly for a given number of iterations,
measuring per-iteration time with `system clock_monotonic`. This is useful for measuring stabilized (warm) performance.

## Included Benchmarks

NBody, PageRank, Bounce, BubbleSort, Fannkuch, Mandelbrot, Queens, QuickSort,
Storage, DeltaBlue, Json, Permute, CD, Sieve, Towers, TreeSort

## Prerequisites

- A SOM implementation with the [SOM standard library](https://github.com/SOM-st/SOM)
  - `Smalltalk/` (standard library)
  - `Examples/Benchmarks/` (benchmark classes including `BenchmarkHarness.som`)
- [ReBench](https://rebench.readthedocs.io/) (for automated benchmarking, optional)

## Running with SOM

The SOM standard library repository ([SOM-st/SOM](https://github.com/SOM-st/SOM)) contains the `Smalltalk/` standard library, `BenchmarkHarness.som`, and all individual benchmark classes (NBody, Json, DeltaBlue, CD, etc.) under `Examples/Benchmarks/`.

Clone the standard library if your SOM implementation does not bundle it:

```sh
git clone https://github.com/SOM-st/SOM.git
```

Then run with any SOM implementation (CSOM, SOM++, TruffleSOM, PySOM, RPySOM, etc.):

```sh
SOM_DIR=path/to/SOM
BENCH_DIR=path/to/som-experiment-benchmarks

# Experiment (single-pass, shuffled order)
./som -cp $SOM_DIR/Smalltalk:$SOM_DIR/Examples/Benchmarks:$SOM_DIR/Examples/Benchmarks/NBody:$SOM_DIR/Examples/Benchmarks/Json:$SOM_DIR/Examples/Benchmarks/DeltaBlue:$SOM_DIR/Examples/Benchmarks/CD \
  $BENCH_DIR/Experiment/Experiment1.som

# ExperimentStable (iterated, 100 iterations)
./som -cp $SOM_DIR/Smalltalk:$SOM_DIR/Examples/Benchmarks:$SOM_DIR/Examples/Benchmarks/NBody:$SOM_DIR/Examples/Benchmarks/Json:$SOM_DIR/Examples/Benchmarks/DeltaBlue:$SOM_DIR/Examples/Benchmarks/CD \
  $BENCH_DIR/ExperimentStable/ExperimentStable1.som 100
```

For example, with CSOM:

```sh
# Build CSOM
git clone https://github.com/SOM-st/CSOM.git
cd CSOM && make

# Run an experiment
./CSOM -cp ../SOM/Smalltalk:../SOM/Examples/Benchmarks:../SOM/Examples/Benchmarks/NBody:../SOM/Examples/Benchmarks/Json:../SOM/Examples/Benchmarks/DeltaBlue:../SOM/Examples/Benchmarks/CD \
  ../som-experiment-benchmarks/Experiment/Experiment1.som
```

## Running with ReBench

### 1. Using the provided `rebench.conf`

The included `rebench.conf` defines the benchmark suites with placeholder variables.
Copy it to your SOM project and fill in the executor and experiment sections:

```yaml
# Add to your project's rebench.conf or adapt rebench.conf from this repo

executors:
    MySOM:
        path: .
        executable: som

experiments:
    all:
        executions:
          - MySOM:
              suites:
                - experiment-shuffle
                - experiment-stable
```

You will need to adjust the `command` field in each suite to match your classpath layout.

### 2. Integrating into an existing ReBench config

Add the experiment suites to your existing config. For example:

```yaml
benchmark_suites:
    experiment-shuffle:
        gauge_adapter: PlainSecondsLog
        command: "-cp Smalltalk:Examples/Benchmarks:Examples/Benchmarks/NBody:Examples/Benchmarks/Json:Examples/Benchmarks/DeltaBlue:Examples/Benchmarks/CD path/to/som-experiment-benchmarks/Experiment/%(benchmark)s.som"
        invocations: 500
        benchmarks:
            - Experiment1
            - Experiment2
            # ... Experiment3 through Experiment20
            - Experiment20
            - ExperimentRandom

    experiment-stable:
        gauge_adapter: PlainSecondsLog
        command: "-cp Smalltalk:Examples/Benchmarks:Examples/Benchmarks/NBody:Examples/Benchmarks/Json:Examples/Benchmarks/DeltaBlue:Examples/Benchmarks/CD path/to/som-experiment-benchmarks/ExperimentStable/%(benchmark)s.som %(iterations)s"
        invocations: 500
        iterations: 100
        benchmarks:
            - ExperimentStable1
            - ExperimentStable2
            # ... ExperimentStable3 through ExperimentStable20
            - ExperimentStable20
```

Then run:

```sh
rebench -df experiment.data rebench.conf
```

## Dependencies on BenchmarkHarness

All experiment classes inherit from `BenchmarkHarness`, which is part of the standard SOM benchmark infrastructure.
The key methods used are:

- `benchmarkClass:` / `numIterations:` / `innerIterations:` — configure which benchmark to run
- `doRuns:` — execute the benchmark with timing
- `printAll:` — control output verbosity

Your SOM implementation's `BenchmarkHarness.som` must provide these methods.

## License

See the individual `.som` files for license information.
