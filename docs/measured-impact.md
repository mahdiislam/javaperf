# Measured impact

Combining a GraalVM native image with `Unsafe` gave the lowest typical runtime: a median of 3.05 s, 4.8× faster than the 14.5 s median of JVM entries that used neither.

The groups overlap, though. The best JVM entry without `Unsafe` (`gonix`, 2.997 s) beat 13 of the 22 native-image entries. Technique sets the ceiling, and execution decides the result.

## Runtime by technique

| Technique | Entries | Fastest (s) | 25th pct (s) | Median (s) | 75th pct (s) | Slowest (s) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Native image + `Unsafe` | 18 | 1.535 | 2.05 | 3.05 | 3.81 | 6.635 |
| Native image | 4 | 5.559 | 8.68 | 11.11 | 14.54 | 20.691 |
| JVM + `Unsafe` | 24 | 1.880 | 3.80 | 5.41 | 8.06 | 11.505 |
| JVM, no `Unsafe` | 119 | 2.997 | 8.37 | 14.50 | 27.44 | 289.679 |

Source: the Results table in the top-level [README](../README.md) (165 entries, 8 cores, early 2024). Each entry's group comes from the README's Notes column: "GraalVM native binary" marks a native image and "uses Unsafe" marks `Unsafe`. The slowest JVM entry is the baseline.

## Leaders

| # | Time (s) | Entry | Technique |
| ---: | ---: | --- | --- |
| 1 | 1.535 | `thomaswue` (with `merykitty`, `mukel`) | Native image + `Unsafe` |
| 2 | 1.587 | `artsiomkorzun` | Native image + `Unsafe` |
| 3 | 1.608 | `jerrinot` | Native image + `Unsafe` |
| 4 | 1.880 | `serkan_ozal` | JVM + `Unsafe` |
| 5 | 1.921 | `abeobk` | Native image + `Unsafe` |
| 15 | 2.997 | `gonix` | JVM, no `Unsafe` |
| 17 | 3.210 | `merykitty` | JVM, no `Unsafe` |
| 165 | 289.679 | `baseline` | JVM, no `Unsafe` |

## How to read this

- **Correlation, not cause.** The fastest authors stacked every technique, so a group's median reflects who chose it as much as the technique itself.
- **Small group.** "Native image" without `Unsafe` has only 4 entries.
- **Mixed JITs.** "JVM" mixes HotSpot and the Graal JIT, which aren't split out here.
- **Startup matters at this scale.** A native image mostly removes JVM startup and warmup, which is a large share of a 1.5 to 3 s run.
- **Noise.** Runs vary by about ±3% (per the README), and several top-10 entries are closer together than that.
- **Duplicate names.** `ricardopieper` and `artpar` each appear twice in the README table, as separate submissions.
