# vscan

A fast, concurrent port scanner written in Go.

## Features

- TCP Connect scan (no root required) and SYN half-open scan
- Host discovery and CIDR range expansion (`192.168.1.0/24`)
- Port presets (`top100`, `top1000`) and custom ranges (`80,443`, `1-1024`)
- Banner grabbing and service detection (HTTP, SSH, MySQL, Redis, etc.)
- Worker pool architecture to keep scans fast without exhausting file descriptors
- Output in CLI table, JSON, or CSV format

## Installation

```bash
go install github.com/ivoidable/vscan@latest
```

Or build from source:

```bash
git clone https://github.com/ivoidable/vscan.git
cd vscan
go build -o vscan .
```

## Quick Start

```bash
# Scan default top 100 ports
vscan scan 192.168.1.1

# Scan specific ports with service detection
vscan scan 192.168.1.1 -p 80,443,8080 --service-detect

# Scan an entire subnet and export results to JSON
vscan scan 192.168.1.0/24 -p 80,443 -o json --out results.json

# Fast full-port sweep with custom worker count and timeout
vscan scan 192.168.1.1 -p 1-65535 -w 500 -t 1s

# SYN scan (requires root)
sudo vscan scan 192.168.1.1 -p 1-1024 --scan syn --iface en0
```

## Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--ports` | `-p` | `top100` | Ports to scan (`80`, `1-1024`, `top100`, `top1000`) |
| `--scan` | | `tcp` | Scan type: `tcp` (connect) or `syn` (half-open) |
| `--output` | `-o` | `cli` | Output format: `cli`, `json`, `csv` |
| `--out` | | | Write results to a file |
| `--workers` | `-w` | `100` | Concurrent worker goroutines |
| `--timeout` | `-t` | `2s` | Per-port connection timeout |
| `--service-detect` | | `false` | Enable service identification and banner grabbing |
| `--no-ping` | | `false` | Skip host discovery ping check |
| `--iface` | | auto | Network interface for SYN scan |

## Scan Modes

- **TCP Connect (`--scan tcp`)**: Standard 3-way handshake. Fast, reliable, and requires no special privileges.
- **SYN Half-Open (`--scan syn`)**: Sends raw SYN packets and tears down responses with RST. Faster and stealthier, but requires root privileges and raw socket access.

## Service Detection

When `--service-detect` is set, `vscan` probes open ports for banners to identify services and versions:
- Sends `HEAD /` to HTTP/HTTPS endpoints to inspect server headers
- Captures SSH identification strings
- Uses protocol probes for services such as MySQL and Redis

## Testing

```bash
go test ./...
```

## License

MIT
