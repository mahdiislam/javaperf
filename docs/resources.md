# Resources

The best write-ups come from people who submitted top entries. Start with the step-by-step post, then read the code (see [Performance techniques](performance-techniques.md)).

## Write-ups and talks

| Resource | Type | Why read it |
| --- | --- | --- |
| [The Billion Row Challenge: Step-by-step from 71s to 1.7s](https://questdb.io/blog/billion-row-challenge-step-by-step/), Marko Topolnik | Blog post | Adds one technique at a time with timings. The best map from concept to gain. |
| [#1BRC Timeline](https://tivrfoa.github.io/java/benchmark/performance/2024/02/05/1BRC-Timeline.html), tivrfoa | Blog post | How the leaderboard moved and which idea caused each jump. |
| [Cliff Click on the Coffee Compiler Club](https://www.youtube.com/watch?v=NJNIbgV6j-Y) | Video | A JIT compiler author explains why his code is shaped the way it is. |
| [Reading a file insanely fast in Java](https://rmannibucau.metawerx.net/reading-a-file-insanely-fast-in-java.html), Romain Manni-Bucau | Blog post | Compares file I/O options: streams, channels and mmap. |
| [1BRC shows Java can process a 1B-row file in two seconds](https://www.infoq.com/news/2024/01/1brc-fast-java-processing), InfoQ | Interview | Background from the organizer, Gunnar Morling. |
| [One Billion Row Challenge](https://curiouscoding.nl/posts/1brc/), Ragnar Groot Koerkamp | Blog post | A deep low-level dive in Rust; the ideas carry over to Java. |
| [1BRC in .NET among fastest on Linux](https://hotforknowledge.com/2024/01/13/1brc-in-dotnet-among-fastest-on-linux-my-optimization-journey/), Victor Baybekov | Blog post | The same techniques in another managed runtime. |
| [Entering 1BRC with GitHub Copilot](https://devblogs.microsoft.com/java/entering-the-one-billion-row-challenge-with-github-copilot/), Antonio Goncalves | Blog post | How far an assistant-driven approach gets. |

The top-level README's "1BRC on the Web" section lists about 25 more posts, including SQL, Go, C and database solutions. The upstream [Show & Tell discussions](https://github.com/gunnarmorling/1brc/discussions/categories/show-and-tell) collect entries in other languages.

## Tools

| Tool | Use |
| --- | --- |
| [async-profiler](https://github.com/async-profiler/async-profiler) via [ap-loader](https://github.com/jvm-profiling-tools/ap-loader) and [jbang](https://jbang.dev) | CPU flame graphs (see [Running the examples](running-the-examples.md#profiling)) |
| [hyperfine](https://github.com/sharkdp/hyperfine) | The benchmark runner `evaluate.sh` uses |
| [SDKMAN](https://sdkman.io) | Installing and switching the JDKs the prepare scripts ask for |

## Specs and docs

These links weren't checked when this page was written; confirm they're current before relying on them.

| Link | Relevance |
| --- | --- |
| [JEP 454: Foreign Function & Memory API](https://openjdk.org/jeps/454) | The supported replacement for `Unsafe` memory access (`MemorySegment`) |
| [JEP 471: Deprecate the memory-access methods in sun.misc.Unsafe for removal](https://openjdk.org/jeps/471) | Why `Unsafe`-based entries need a port |
| [JEP 460: Vector API (Seventh Incubator)](https://openjdk.org/jeps/460) | The API behind the `ByteVector` entries |
| [GraalVM Native Image reference](https://www.graalvm.org/latest/reference-manual/native-image/) | The `-O3`, `-march=native` and `--gc=epsilon` options the prepare scripts use |
