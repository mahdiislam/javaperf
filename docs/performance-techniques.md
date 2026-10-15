# Performance techniques

The gains come from four layers:

- spreading the work across cores
- cutting the cost of each byte
- keeping the hot loop free of allocation and branches
- removing JVM startup and shutdown

The baseline (`CalculateAverage_baseline.java`, 92 lines) uses none of them. It runs `Files.lines`, `String.split`, `Double.parseDouble` and a `TreeMap` collector on one thread.

Adoption counts below are out of the 165 Java entries, counted by searching the source files and scripts.

| Technique | What it does | Why it pays | Where to read it |
| --- | --- | --- | --- |
| Parallel chunking | Split the file into byte ranges, one per core, with each boundary snapped to the next `\n`. | 8 cores do the work of one. Usually the first and largest single gain. | Almost every fast entry |
| Work stealing in small segments | Workers claim segments of about 2 MB from a shared `AtomicLong`/`AtomicInteger` cursor instead of taking one equal slice each. | No core sits idle while a slower one finishes. Credited to `artsiomkorzun`. | `thomaswue`, `artsiomkorzun`, `jerrinot` |
| Memory-mapped input | `FileChannel.map`, or a `MemorySegment` from the FFM API, maps the file into memory with no read buffers or copies. | Removes copying and decoding; the OS page cache serves bytes directly. | 124 entries use `MapMode`; 46 use `MemorySegment` |
| Raw memory access (`Unsafe`) | Read 8 bytes at a time from the mapped address with `Unsafe.getLong`. | Skips bounds checks and segment liveness checks in the hot loop. | 41 entries, including all of the top 3 |
| SWAR (SIMD within a register) | Find `;` and `\n` in 8 bytes at once with the `0x0101010101010101L` / `0x8080808080808080L` bit trick and `numberOfTrailingZeros`. | One branch per 8 bytes instead of one per byte. | 25 entries; `thomaswue`, `royvanrijn`, `jerrinot` |
| Vector API | `jdk.incubator.vector` `ByteVector` compares 16 to 32 bytes per instruction. | Wider than SWAR. Strongest in HotSpot entries. | 22 entries; `merykitty`, `serkan_ozal` |
| Branchless number parsing | Turn `-12.3` into the int `-123` with shifts and one multiply, with no per-digit branches. Temperatures always have one decimal, so integer tenths are exact. | Avoids branch mispredictions on random data and avoids `double` math. By Quan Anh Mai (`merykitty`). | `merykitty`, `thomaswue` (`convertIntoNumber`) |
| Custom open-addressing hash table | A flat array keyed by a hash of the name's first 8 to 16 bytes; names compared as `long` words; linear probing on collision. | No `String` creation, no boxing, and a cache-friendly layout. | All top entries |
| Avoiding the `<8` / `8-16` byte branch | Handle names up to 16 bytes with the same straight-line code, even if it does extra work. | Mixed name lengths caused mispredictions. Credited to `jerrinot`. | `jerrinot`, `thomaswue` |
| Several cursors per thread | Each thread scans 2 or 3 parts of its segment at once. | Instruction-level parallelism: the CPU overlaps independent dependency chains. | `thomaswue` (3 cursors), `jerrinot` |
| GraalVM native image | Compile ahead of time with `-O3 -march=native`, so there is no JIT warmup. | Startup and warmup are a large share of a 1.5 to 3 s run. | 22 native-image entries, including all of the top 3 |
| Subprocess trick | `main` spawns a worker process and exits as soon as it prints the result. The worker unmaps the 12 GB file afterwards. | Unmapping is slow, and the timing stops when the parent exits. Idea by Alfonso² Peterssen. | 8 entries use `ProcessBuilder` |
| No GC, fewer safepoints | `--gc=epsilon` or `-XX:+UseEpsilonGC`, `-H:-GenLoopSafepoints`; everything preallocated. | The hot loop allocates nothing, so GC is pure overhead. | `prepare_thomaswue.sh` |
| JVM startup flags | `-XX:-TieredCompilation`, AppCDS (`-XX:SharedArchiveFile`), `-XX:+UseTransparentHugePages`, `-XX:+AlwaysPreTouch`. | Faster startup and fewer TLB misses on the mapped file. | `linl33` (AppCDS); various launch scripts |

## Where to start reading

The header comment of [`CalculateAverage_thomaswue.java`](../src/main/java/dev/morling/onebrc/CalculateAverage_thomaswue.java), the winning entry, credits the authors of several of these ideas. It's the best file to read first. Then compare:

- `CalculateAverage_artsiomkorzun.java` (2nd): a similar design with heavier use of `Unsafe`.
- `CalculateAverage_jerrinot.java` (3rd): branch avoidance for name lengths.
- `CalculateAverage_merykitty.java`: the fastest approach without `Unsafe`, using the Vector API and `MemorySegment`.
- `CalculateAverage_baseline.java`: the starting point.

See [Measured impact](measured-impact.md) for how these techniques line up with results.
