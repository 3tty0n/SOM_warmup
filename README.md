# SOM Experiment Benchmarks

Large-scale benchmarks for [SOM (Simple Object Machine)](https://som-st.github.io/) implementations.
Each benchmark runs 16 standard SOM benchmarks in a different shuffled order within a single invocation, designed to evaluate JIT compilation behavior under diverse workloads.

## Structure

```
Experiment.som                  # Base experiment (small subset: DeltaBlue, Json, CD)
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

- A SOM implementation (e.g., [SOM](https://github.com/SOM-st), TruffleSOM, RPySOM, CSOM, etc.)
- The SOM standard library (`Smalltalk/`)
- The standard SOM benchmark classes listed above (typically found in `Examples/Benchmarks/` of your SOM implementation)
- [ReBench](https://rebench.readthedocs.io/) (for automated benchmarking)

## Running Manually

Run a single experiment directly with your SOM implementation:

```sh
# Experiment (single-pass, shuffled order)
./som -cp Smalltalk:Examples/Benchmarks:Examples/Benchmarks/NBody:Examples/Benchmarks/Json:Examples/Benchmarks/DeltaBlue:Examples/Benchmarks/CD \
  path/to/som-experiment-benchmarks/Experiment/Experiment1.som

# ExperimentStable (iterated, 100 iterations)
./som -cp Smalltalk:Examples/Benchmarks:Examples/Benchmarks/NBody:Examples/Benchmarks/Json:Examples/Benchmarks/DeltaBlue:Examples/Benchmarks/CD \
  path/to/som-experiment-benchmarks/ExperimentStable/ExperimentStable1.som 100
```

Adjust the classpath (`-cp`) to match your SOM implementation's directory layout.

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
