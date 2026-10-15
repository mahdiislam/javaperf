# Running on Apple Silicon

These notes target an Apple M2 with 24 GB of memory, and most of them apply to any Apple Silicon Mac. You can run nearly everything, except the Linux-only parts of the official benchmark setup.

These notes come from reading the scripts, not from a run on a Mac. Check items 1 and 2 under [Issues specific to Apple Silicon](#issues-specific-to-apple-silicon) first.

## Tools to install

| Tool | Install | Needed for |
| --- | --- | --- |
| Xcode Command Line Tools | `xcode-select --install` | git, and the C toolchain GraalVM `native-image` needs |
| Homebrew | see [brew.sh](https://brew.sh) | the tools below |
| SDKMAN | `curl -s "https://get.sdkman.io" \| bash` | switching JDKs; every `prepare_*.sh` runs `sdk use java …` |
| JDK 21 (arm64) | `sdk install java 21.0.1-open` | building, and every entry without a prepare script |
| GraalVM 21 (arm64) | `sdk install java 21.0.2-graal` | the native-image entries, including the top 3 |
| coreutils | `brew install coreutils` | `gtimeout`, which `test_all.sh`, `test_ci.sh` and `evaluate.sh` use on macOS |
| hyperfine, jq | `brew install hyperfine jq` | `evaluate.sh` (`bc` ships with macOS) |
| jbang (optional) | `brew install jbangdev/tap/jbang` | flame graphs with async-profiler |

Maven is bundled as `./mvnw`. You don't need `numactl`, because `evaluate.sh` has a macOS branch that runs hyperfine without it.

## Issues specific to Apple Silicon

1. **18 launch scripts will probably fail to start.** They pass `-XX:+UseTransparentHugePages`, which HotSpot only defines on Linux. On macOS this should fail with `Unrecognized VM option` and exit. Remove the flag, or guard it the way `calculate_average_melgenek.sh` does:

    ```bash
    if [[ "$(uname -s)" == "Linux" ]]; then
        JAVA_OPTS="$JAVA_OPTS -XX:+UseTransparentHugePages"
    fi
    ```

    List the affected scripts with `grep -l UseTransparentHugePages calculate_average_*.sh`.

2. **Old JDK versions may be missing for arm64.** Check with `sdk list java`. If a version a prepare script asks for is gone, install the closest 21.x build and edit the script's `sdk use` line, or symlink the old name in `~/.sdkman/candidates/java/`.

3. **24 GB of memory is enough.** The 1B-row file is about 12 GB. After the first run, macOS can keep most or all of it in the page cache, which is close to the official RAM-disk setup. Close other apps when timing. You also need about 13 GB of free disk.

4. **Cores are uneven.** A 24 GB M2 is the base chip, with 4 performance cores and 4 efficiency cores. Entries that split the file into equal slices per thread wait for the slow efficiency cores. Entries that hand out small segments as threads free up (`thomaswue`, `artsiomkorzun`, `jerrinot`) should cope better. Expect rankings to differ from the README.

5. **Vector API entries may lose ground.** M2 NEON registers are 128 bits wide, against 256-bit AVX2 on the test machine. An entry that hard-codes `SPECIES_256` is likely to fall back to slow code. Entries using `SPECIES_PREFERRED` should adapt.

6. **`Unsafe`, SWAR and branchless parsing work unchanged.** Apple Silicon is little-endian, like x86, so the 8-byte word tricks give the same results.

7. **Native images build for arm64.** `-march=native` should work with GraalVM 21 on AArch64. If a build rejects it, drop the flag from that prepare script. The first build of each image takes a minute or two.

8. **`test.sh` uses `diff --color=always`.** Recent macOS versions support it. If yours errors, run `brew install diffutils` and put it first on your `PATH`.

## Quick start

```bash
sdk install java 21.0.1-open && sdk use java 21.0.1-open
./mvnw clean verify -Dquick
./create_measurements.sh 1000000000
./test.sh baseline
time ./calculate_average_baseline.sh
```
