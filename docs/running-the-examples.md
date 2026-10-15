# Running the examples

This repo (the One Billion Row Challenge, 1BRC) holds 165 Java solutions to one task. Each reads a 1-billion-line `measurements.txt` (about 12 GB) of `station;temperature` rows and prints the min, mean and max for each station, sorted by name. The fastest entry takes 1.535 s. The baseline takes 289.7 s on the same 8 cores, about 190× slower.

Each entry is self-contained, so you can build, test and time any of them on its own.

## Prerequisites

You need Linux or macOS, JDK 21 managed by SDKMAN, and about 13 GB of free disk for the full dataset. Everything else depends on which entries you want to run. On a Mac, also read [Running on Apple Silicon](apple-silicon.md).

| Requirement | Needed for | Notes |
| --- | --- | --- |
| [SDKMAN](https://sdkman.io) | Any entry with a `prepare_<name>.sh`, plus `test_ci.sh` and `evaluate.sh` | Prepare scripts call `sdk use java <version>`. The scripts expect `$HOME/.sdkman/bin/sdkman-init.sh`. |
| JDK `21.0.1-open` | Building, and every entry without a prepare script | The default in `.sdkmanrc`. The build compiles with `--release 21 --enable-preview` and the incubator Vector API. |
| GraalVM `21.0.2-graal` / `21.0.1-graal` | Native-image entries (all of the top 3) | Prepare scripts call `native-image`. Without the binary in `target/`, most launch scripts fall back to JVM mode. |
| JDK `22.ea.32-open` | `linl33` only (`src/main/java-22`) | Compiled only when Maven runs on JDK 22 (the `jdk22` profile). |
| Maven | Building | Bundled as `./mvnw`; no separate install. |
| Disk: about 12 GB per 1B-row file | Generating `measurements.txt` | `evaluate.sh` expects `measurements_1B.txt`; `test_ci.sh` expects `measurements_10M.txt`. |
| RAM: 16 GB or more recommended | Reproducing published timings | Official runs read the file from a RAM disk, so it stays in memory. |
| `hyperfine`, `jq`, `bc` | `evaluate.sh` | Checked at startup; the script exits if any is missing. |
| `numactl` | `evaluate.sh` on Linux | Pins the run to 8 cores (`--physcpubind=0-7`), like the official setup. Not used on macOS. |
| `timeout` (`gtimeout` on macOS) | `test_all.sh`, `test_ci.sh`, `evaluate.sh` | Kills runaway entries; without it a hung entry blocks the loop. |
| [jbang](https://jbang.dev) (optional) | Flame graphs with async-profiler | See [Profiling](#profiling). |

Prepare scripts ask for these exact JDK versions: `21.0.1-graal` (33 scripts), `21.0.2-graal` (18), `21.0.2-tem` (4), `21.0.1-tem` (3), `21.0.2-open` (2), `21.0.1-graalce` (1), `21.0.1-open` (1) and `22.ea.32-open` (1). SDKMAN drops old builds over time. If a version is gone, install the closest 21.x build and edit that script's `sdk use` line.

The official machine was a Hetzner AX161 (AMD EPYC 7502P, Zen 2, 32 cores, 128 GB RAM) with SMT and turbo boost off (see [ENVIRONMENT.md](../ENVIRONMENT.md)). Expect different absolute numbers on other hardware.

## How an entry is wired

Each entry `<name>` is three files that must agree on the name (casing matches the author's GitHub username):

| File | Role |
| --- | --- |
| `src/main/java/dev/morling/onebrc/CalculateAverage_<name>.java` | The whole implementation, in one source file with no external dependencies |
| `calculate_average_<name>.sh` | Launch script. Sets `JAVA_OPTS` and runs the class from the jar, or a native binary from `target/` if one exists. It must print only the result to stdout. |
| `prepare_<name>.sh` (optional) | Runs before tests and benchmarks: selects the JDK and may build a GraalVM native image or an AppCDS archive into `target/` |

`CalculateAverage_baseline.java` is the reference implementation, and its output defines correctness.

## Steps

1. Build the jar (`target/average-1.0.0-SNAPSHOT.jar`). The default build also reformats sources and checks license headers; add `-Dquick` to skip both.

    ```bash
    ./mvnw clean verify
    ```

2. Generate the input. One billion rows takes a few minutes and writes about 12 GB to `measurements.txt` in the repo root.

    ```bash
    ./create_measurements.sh 1000000000
    ```

3. If the entry has a `prepare_<name>.sh`, run it first. It skips the build when the file already exists; `./mvnw clean` forces a rebuild.

    ```bash
    source ./prepare_thomaswue.sh
    ```

4. Run the entry. It reads `./measurements.txt` and prints `{Abha=-23.0/18.0/59.2, ...}`.

    ```bash
    time ./calculate_average_thomaswue.sh
    ```

5. Check correctness against the samples in `src/test/resources/samples`. `test.sh` runs the prepare script, links each sample to `measurements.txt`, and diffs the output with the matching `.out` file.

    ```bash
    ./test.sh thomaswue
    ./test.sh thomaswue src/test/resources/samples/measurements-boundaries.txt
    ./test_all.sh 2>/dev/null     # PASS / FAIL / TIME for every entry
    ```

    `test.sh` deletes `measurements.txt` when it finishes. Keep the big file under another name (for example `measurements_1B.txt`) and link it back afterwards.

6. Benchmark like the official runs. `evaluate.sh` runs hyperfine on `measurements_1B.txt`, discards the fastest and slowest runs and averages the rest.

    ```bash
    mv measurements.txt measurements_1B.txt
    ./evaluate.sh baseline thomaswue
    ```

## Profiling

Use async-profiler through jbang. It writes a flame graph to `profile.html`.

```bash
jbang --javaagent=ap-loader@jvm-profiling-tools/ap-loader=start,event=cpu,file=profile.html \
  -m dev.morling.onebrc.CalculateAverage_<name> target/average-1.0.0-SNAPSHOT.jar
```

## Starting your own entry

Run `./create_fork.sh <name>`, or `./create_fork.sh <name> <source_entry>` to start from an existing entry. It copies the Java file and the scripts and renames the class.
