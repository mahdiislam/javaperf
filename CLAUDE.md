# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The One Billion Row Challenge (1BRC): read a `measurements.txt` of `<station>;<temp>` lines and print min/mean/max per station, sorted by name, in the format `{Abha=-23.0/18.0/59.2, Abidjan=...}`. The repo holds ~165 competing implementations side by side. The challenge is closed to new submissions (final leaderboard published Feb 4, 2024).

## Commands

```bash
./mvnw clean verify                      # build target/average-1.0.0-SNAPSHOT.jar (also reformats sources, checks license headers)
./mvnw clean verify -Dquick              # skip the qa profile (formatter + license check)
./mvnw -B clean verify -Pci              # what CI runs: fails if sources aren't formatted

./create_measurements.sh 1000000000      # generate measurements.txt (~12 GB) in the repo root
./calculate_average_<name>.sh            # run one implementation against ./measurements.txt

./test.sh <name>                         # diff one implementation against every sample in src/test/resources/samples
./test.sh <name> src/test/resources/samples/measurements-1.txt   # a single sample
./test_all.sh                            # PASS/FAIL/TIME for every implementation (5s timeout each)
./evaluate.sh <name> [<name> ...]        # official benchmark: hyperfine on measurements_1B.txt (needs sdkman, hyperfine, jq, bc)
```

There are no JUnit tests. "Tests" are the sample `.txt`/`.out` pairs: `test.sh` symlinks each sample to `measurements.txt`, runs the launch script, and diffs the output after `tocsv.sh` normalizes it. `test.sh` deletes `measurements.txt` when done.

## How an implementation is wired

Each entry `<name>` is three files that must agree on the name (case matches the GitHub username):

- `src/main/java/dev/morling/onebrc/CalculateAverage_<name>.java`: the whole implementation in one source file (rule: single file, no external dependencies).
- `calculate_average_<name>.sh`: launch script. Sets `JAVA_OPTS` and runs the class from the shaded jar, or a native binary in `target/` if one exists. It must print nothing to stdout except the result.
- `prepare_<name>.sh` (optional): runs before tests/benchmarks. Typically `sdk use java <version>` (sdkman) and/or building a GraalVM `native-image` or AppCDS archive into `target/`. Scripts only rebuild if the artifact is missing, so `./mvnw clean` forces a rebuild.

`./create_fork.sh <name> [source_fork]` scaffolds a new entry by copying the baseline (or another entry) and renaming.

`CalculateAverage_baseline.java` is the reference implementation; its output defines correctness. `CreateMeasurements*.java` generate input data from `data/weather_stations.csv`.

## Build details

- Compiles with `--release 21`, `--enable-preview`, `jdk.incubator.vector` and some `--add-opens`, so entries can use preview APIs, the Vector API and `Unsafe`.
- `src/main/java-22/` holds entries that need JDK 22 (currently only `linl33`); it compiles only when Maven runs on JDK 22 (the `jdk22` profile). CI switches to `22.ea.32-open` only for those.
- The default `qa` profile runs the Eclipse formatter (`etc/eclipse-formatter-config.xml`) and license-header check (`etc/license.txt`) on every build. Commit the formatter's changes; CI validates formatting.
- Default JDK is `21.0.1-open` (`.sdkmanrc`).
- `src/main/go/` and `src/main/python/` hold non-Java experiments and a non-authoritative Python data generator. They aren't part of the Maven build.

## Rules entries must respect (from README)

- Station names are UTF-8, 1–100 bytes, with no `;` or `\n`. There are at most 10,000 unique stations. Temperatures are in -99.9..99.9 with exactly one decimal.
- Don't rely on dataset specifics: any valid names and distribution must work. The `measurements-10000-unique-keys`, `-boundaries` and `-complex-utf8` samples cover the edge cases.
- Round output using IEEE 754 roundTowardPositive.
- All computation happens at runtime. Native-image build-time processing of the data is not allowed.
- Official timing: 8 cores of a Hetzner AX161 (EPYC 7502P) from a RAM disk, end-to-end via hyperfine (see `evaluate.sh`). JVM startup counts, which is why many top entries use native images.
