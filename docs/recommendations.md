# Recommendations

The top 10 entries are within 0.9 s of each other (1.535 s to 2.367 s). The largest remaining wins are in tooling, portability and measurement, not in shaving another 50 ms.

## For the code

1. **Port `Unsafe` entries to the FFM API.** Memory access through `sun.misc.Unsafe` is deprecated for removal ([JEP 471](https://openjdk.org/jeps/471)). 41 entries depend on it, including the top 3. A `MemorySegment` port of `thomaswue` with the same native-image flags would show how much of the speed comes from `Unsafe` on current JDKs.
2. **Re-run on a current JDK and GraalVM.** All results are from JDK 21.0.1/21.0.2 (early 2024). The Vector API, FFM and Graal have changed since. The HotSpot-only entries (`merykitty`, `serkan_ozal`) may gain the most.
3. **Combine the Vector API with the native-image tricks.** No entry combines `ByteVector` scanning with the subprocess trick. Only `SamuelYvon` pairs the Vector API with a native image. This combination is largely untested.
4. **Test against the full input spec.** Many fast hash tables key on the first 8 to 16 bytes and fall back to a slow path. Test candidates against `measurements-10000-unique-keys.txt` and long UTF-8 names, not only the 413 stations that `CreateMeasurements` generates.
5. **Show the gain step by step.** Pick one entry and keep each optimization as its own commit with its timing, as Topolnik's post does. That would be the most reusable teaching material this repo could add.
6. **Guard Linux-only JVM flags.** 18 launch scripts pass `-XX:+UseTransparentHugePages` unconditionally, which probably breaks them on macOS (see [Running on Apple Silicon](apple-silicon.md)). Wrap the flag in a `uname` check, as `calculate_average_melgenek.sh` does.

## For the repo and harness

1. **Add a real test entry point.** There are no JUnit tests. A parameterized test that runs each `CalculateAverage_*` class against the sample `.txt`/`.out` pairs would let `./mvnw verify` catch regressions without the shell scripts.
2. **Stop `test.sh` from deleting `measurements.txt`.** It replaces the file with a symlink and removes it at the end, so a generated 12 GB file disappears after a test run. Use a temp directory, or restore the original file or link.
3. **Record machine-readable results.** Results live only in the README table. A CSV of entry, time, JDK, technique and date would make analysis and re-runs easy. See [Measured impact](measured-impact.md) for one analysis.
4. **Make the environment reproducible.** `evaluate.sh` assumes SDKMAN, hyperfine, jq, bc and numactl. A container image or devcontainer would make runs repeatable on other machines.
5. **Benchmark on more hardware.** One AMD Zen 2 server decided the ranking. Running the top 20 on a recent Intel machine and an ARM machine (Graviton or Apple Silicon) would show which tricks are portable and which are tuned to Zen 2.
6. **Report variance.** The README notes ±3% run-to-run variation, and several top-10 entries are closer than that. Publish hyperfine's standard deviation next to each mean.
