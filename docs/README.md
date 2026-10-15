# 1BRC guides

Notes on running, understanding and extending the entries in this repository.

| Document | What it covers |
| --- | --- |
| [Running the examples](running-the-examples.md) | Prerequisites, build, data generation, running, testing, benchmarking and profiling an entry |
| [Running on Apple Silicon](apple-silicon.md) | Tools and fixes needed on an Apple M2 (24 GB) or similar Mac |
| [Performance techniques](performance-techniques.md) | The techniques behind the speedups, with adoption counts and code pointers |
| [Measured impact](measured-impact.md) | Leaderboard results grouped by technique (native image, `Unsafe`) |
| [Resources](resources.md) | Write-ups, talks, tools and specs worth reading |
| [Recommendations](recommendations.md) | Suggested next steps for the code and for the repo's tooling |

All figures come from the Results table in the top-level [README](../README.md): 165 entries, run on 8 cores of a Hetzner AX161 (AMD EPYC 7502P) in early 2024.
