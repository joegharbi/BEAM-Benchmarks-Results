# BEAM Web Server Benchmark Results

This repository holds benchmark and energy-measurement outputs for **BEAM ecosystem web frameworks** (Erlang, Elixir, and Gleam targets).

Data were produced with the [BEAM Web Server Benchmarks Framework](https://github.com/joegharbi/BEAM-web-server-benchmarks). Power and consumption samples were collected with [Scaphandre](https://github.com/hubblo-org/scaphandre).

## Note on the high load HTTP results

The static and dynamic HTTP results in this repository are affected by connection handling in the benchmark client rather than by the servers under test.

The client opens a new TCP connection for every request and does not reuse connections. Servers that implement HTTP connection reuse keep the connection open after responding, which leaves the client to close it. The closing side holds each socket in `TIME_WAIT` for sixty seconds. With 28,232 ephemeral ports available on the measurement host, the client exhausts its port range and requests begin to fail.

The effect is visible in the data here. In every static and dynamic run, successful request counts for the affected targets stop at exactly 28,232, and again at twice that number.

Servers built directly on `gen_tcp`, meaning the Erlang and Elixir lean targets, close each connection themselves. The server holds `TIME_WAIT` and the client is unaffected, which is why those targets report full success. The Gleam lean targets are built on Mist and follow the framework pattern rather than the lean one.

**Verification.** Re-running Erlang Cowboy at 80,000 requests with connection reuse enabled in the client gives 80,000 successes in 48.3 seconds, against 59,949 without. The Elixir `gen_tcp` handler completes 80,000 under both clients, taking 56.8 seconds with reuse. Reproduce with `tools/port_reuse_test.py` in the framework repository.

**Scope.** This affects the interpretation of HTTP results above roughly 20,000 requests in all four run directories listed below. Results below that level and the WebSocket concurrency results are unaffected. Corrected measurements will be published in follow up work.

## Scope of this repository

- **Focus:** BEAM runtime and framework comparisons (not general-purpose web server shootouts).
- **Processed results:** timestamped runs under `results/<run-id>/` (see below).
- **Raw telemetry:** `output/` (hundreds of Scaphandre JSON files; tens of MB on disk).
- **Figures:** `graphs/paper_graphs/` (curated plots aligned with the paper) and `graphs/other_graphs/` (timestamped batch exports).

## Benchmark runs (`results/`)

Each run is a directory named `YYYY-MM-DD_HHMMSS` containing some or all of `dynamic/`, `static/`, and `websocket/`.

| Run ID | Dynamic CSVs | Static CSVs | WebSocket CSVs | Notes |
|--------|--------------|-------------|----------------|--------|
| `2026-05-07_142551` | 11 | 11 | 20 | Full suite |
| `2026-05-07_234827` | 11 | 11 | 20 | Full suite (same layout as above) |
| `2026-05-08_104424` | 11 | 0 | 0 | Dynamic-only partial rerun |
| `2026-05-08_140359` | 0 | 11 | 0 | Static-only partial rerun |

For a **single folder that contains all three categories**, use `2026-05-07_234827` or `2026-05-07_142551`. The May 8 directories are useful when you only need those slices.

Per full run, expect **11** dynamic HTTP CSVs, **11** static HTTP CSVs, and **20** WebSocket CSVs (five stacks × four scenario families).

## Frameworks covered (this dataset)

Versions below match the `dy-`, `st-`, and `ws-` file naming in the May 2026 runs.

### HTTP (dynamic + static)

- **Elixir:** Cowboy `1.19.5`, Phoenix `1.8.5`, Index `1.19.5`, Pure `1.19.5`
- **Erlang:** Cowboy `28.4.3`, Yaws `28.4.3`, Index `28.4.3`, Pure `28.4.3`
- **Gleam:** Index `1.15.2`, Mist `1.15.2`, Pure `1.15.2`

### WebSocket

- **Elixir:** Cowboy `1.19.5`, Bandit `1.8.5`
- **Erlang:** Cowboy `28.4.3`, Yaws `28.4.3`
- **Gleam:** Mist `1.15.2`

## Test categories

### Dynamic HTTP (`results/<run>/dynamic/`)

Throughput and efficiency for dynamic endpoints.

Typical CSV columns include:

- `Total/Successful/Failed Requests`
- `Execution Time (s)` and `Requests/s`
- `Total Energy (J)` and `Avg Power (W)`
- CPU and memory aggregates (`Avg`, `Peak`, `Total`)

### Static HTTP (`results/<run>/static/`)

Static file serving under load; same column schema as dynamic tests.

### WebSocket (`results/<run>/websocket/`)

Four scenario families per stack (suffix on the filename):

- `*_burst.csv`
- `*_concurrency.csv`
- `*_payload.csv`
- `*_stream.csv`

Typical columns include:

- `Total/Successful/Failed Messages`
- `Messages/s` and `Throughput (MB/s)`
- `Avg/Min/Max Latency (ms)`
- Energy, CPU, and memory aggregates
- Scenario fields (`Pattern`, `Num Clients`, `Message Size`, `Rate`, `Bursts`, `Interval`, `Duration`)

## Quick performance snapshot

Peaks below are taken from **`results/2026-05-07_234827/`** (max `Requests/s` or `Messages/s` over all rows in the relevant CSVs). They are illustrative only; always check success/failure counts and load steps before ranking stacks.

- **Dynamic HTTP:** `1627.53 req/s` (`dy-elixir-pure-1-19-5.csv`)
- **Static HTTP:** `1668.39 req/s` (`st-elixir-pure-1-19-5.csv`)
- **WebSocket concurrency:** `2976.24 msg/s` (`ws-erlang-cowboy-28-4-3_concurrency.csv`)
- **WebSocket stream:** `932.04 msg/s` (`ws-elixir-bandit-1-8-5_stream.csv`)

Some configurations show non-zero failures at aggressive load points; treat throughput together with reliability and tail latency.

## Data layout

```text
.
├── output/                    # Raw Scaphandre JSON telemetry (per benchmark execution)
├── results/
│   ├── 2026-05-07_142551/     # Example full run
│   ├── 2026-05-07_234827/     # Example full run
│   ├── 2026-05-08_104424/     # Dynamic-only partial rerun
│   └── 2026-05-08_140359/     # Static-only partial rerun
│       ├── dynamic/           # HTTP dynamic CSVs (when present)
│       ├── static/            # HTTP static CSVs (when present)
│       └── websocket/         # WebSocket CSVs (when present)
├── graphs/
│   ├── paper_graphs/          # Publication-oriented figures (dynamic / static / websocket)
│   └── other_graphs/          # Additional batch outputs (timestamped subfolders)
└── README.md
```

## Notes

- **File names** encode language, stack, and version, plus WebSocket scenario, for example:
  - `dy-erlang-index-28-4-3.csv`
  - `st-gleam-mist-1-15-2.csv`
  - `ws-elixir-bandit-1-8-5_concurrency.csv`
- **`output/*.json`** holds time-series telemetry (host, consumers, sockets, power samples) aligned with benchmark runs.
- **`graphs/paper_graphs/`** groups PNGs (and companion `.title.txt` files) by test type; WebSocket plots are further split into `burst`, `concurrency`, `payload`, and `stream`.
