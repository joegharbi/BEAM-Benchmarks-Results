# BEAM Web Server Benchmark Results

This repository contains benchmark and energy-measurement outputs for **BEAM ecosystem web frameworks** only (Erlang, Elixir, and Gleam targets).

The data was produced with the [BEAM Web Server Benchmarks Framework](https://github.com/joegharbi/BEAM-web-server-benchmarks), with power and consumption samples collected through [Scaphandre](https://github.com/hubblo-org/scaphandre).

## Scope Of This Repository

- Focus: BEAM language runtime/framework comparisons (not general web server comparisons)
- Test run directory: `results/2026-02-19_083357/`
- Raw telemetry files: `output/` (`486` JSON files, about `36.26 MB`)
- Processed benchmark CSVs: `46` files total
  - `dynamic/`: `15`
  - `static/`: `15`
  - `websocket/`: `16`

## Frameworks Covered

### HTTP (Dynamic + Static)

- Elixir: Cowboy `1.16`, Phoenix `1.8`, Index `1.16`, Pure `1.16`
- Erlang: Cowboy `27`, Yaws `26/27`, Index `23/26/27`, Pure `23/26/27`
- Gleam: Index `1.0`, Mist `1.0`

### WebSocket

- Elixir: Cowboy `1.16`, Phoenix `1.8`
- Erlang: Cowboy `27`, Yaws `27`

## Test Categories

### Dynamic HTTP (`results/2026-02-19_083357/dynamic/`)

Measures request/response throughput and efficiency for dynamic endpoints.

Key CSV metrics:
- `Total/Successful/Failed Requests`
- `Execution Time (s)` and `Requests/s`
- `Total Energy (J)` and `Avg Power (W)`
- CPU and memory aggregates (`Avg`, `Peak`, `Total`)

### Static HTTP (`results/2026-02-19_083357/static/`)

Measures static file serving throughput and resource usage under load.

Uses the same column schema as dynamic tests.

### WebSocket (`results/2026-02-19_083357/websocket/`)

Includes four scenario families per stack:
- `*_burst.csv`
- `*_concurrency.csv`
- `*_payload.csv`
- `*_stream.csv`

Key CSV metrics:
- `Total/Successful/Failed Messages`
- `Messages/s` and `Throughput (MB/s)`
- `Avg/Min/Max Latency (ms)`
- Energy, CPU, and memory aggregates
- Scenario fields (`Pattern`, `Num Clients`, `Message Size`, `Rate`, `Bursts`, `Interval`, `Duration`)

## Quick Performance Snapshot (This Dataset)

These are best observed per-file peaks in the current CSV outputs:

- Dynamic HTTP peak: `1601.60 req/s` (`dy-elixir-pure-1-16.csv`)
- Static HTTP peak: `1604.52 req/s` (`st-elixir-pure-1-16.csv`)
- WebSocket concurrency peak: `3007.01 msg/s` (`ws-elixir-phoenix-1-8_concurrency.csv`)
- WebSocket stream peak: `925.88 msg/s` (`ws-elixir-cowboy-1-16_stream.csv`)

Some stacks show high failed-request counts at larger load points, so compare both throughput and success rate when interpreting rankings.

## Data Layout

```text
.
├── output/                          # Raw Scaphandre JSON telemetry
├── results/
│   └── 2026-02-19_083357/
│       ├── dynamic/                 # HTTP dynamic benchmarks (15 CSV files)
│       ├── static/                  # HTTP static benchmarks (15 CSV files)
│       └── websocket/               # WebSocket benchmarks (16 CSV files)
└── README.md
```

## Notes

- File names encode stack/runtime/version and test family, for example:
  - `dy-erlang-index-27.csv`
  - `st-gleam-mist-1-0.csv`
  - `ws-elixir-phoenix-1-8_concurrency.csv`
- `output/*.json` contains time-series telemetry (host, consumers, sockets, and power samples) captured during each benchmark execution.
