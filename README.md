![CircleCI](https://img.shields.io/circleci/build/github/markormesher/iperf-prometheus-collector)

# `iperf` Prometheus Collector

A simple Prometheus collector to provide measurements about network connection throughput for multiple hosts using the [iperf3](https://iperf.fr/) utility.

:rocket: Jump to [quick-start example](#quick-start-docker-compose-example).

:whale: See releases on [ghcr.io](https://ghcr.io/markormesher/iperf-prometheus-collector).

Note that `iperf` tests take 10+ seconds per target and are executed sequentially. Results from each test will be returned on the next call to `/metrics`.

## Measurements

| Measurement                   | Description                                                        | Labels   |
| ----------------------------- | ------------------------------------------------------------------ | -------- |
| `iperf_tests_started`         | Number of tests that this collector has started.                   | `target` |
| `iperf_tests_finished`        | Number of tests that have finished successfully.                   | `target` |
| `iperf_tests_failed`          | Number of tests that have failed.                                  | `target` |
| `iperf_sent_bytes`            | Number of bytes sent during the test.                              | `target` |
| `iperf_sent_packets`          | Number of packets sent during the test (UDP only).                 | `target` |
| `iperf_sent_lost_packets`     | Number of packet lost during the test (UDP only).                  | `target` |
| `iperf_sent_seconds`          | Duration of the test on the sending side, in seconds.              | `target` |
| `iperf_received_bytes`        | Number of bytes received during the test.                          | `target` |
| `iperf_received_packets`      | Number of packets received during the test (UDP only).             | `target` |
| `iperf_received_lost_packets` | Number of packets lost by the receiver during the test (UDP only). | `target` |
| `iperf_received_seconds`      | Duration of the test on the receiving side, in seconds.            | `target` |

These metrics can be combined to show the throughput in bps with the following example Prometheus query:

```
avg by (target) (iperf_received_bytes / iperf_received_seconds * 8)
```

## Configuration

Configuration is via the following environment variables:

| Variable           | Required? | Description                                                              | Default                 |
| ------------------ | --------- | ------------------------------------------------------------------------ | ----------------------- |
| `TARGET_LIST`      | yes       | Comma separated list of host names or IP addresses to run tests against. | n/a                     |
| `TEST_INTERVAL_MS` | no        | How often to run iperf tests.                                            | 600000ms (= 10 minutes) |
| `PROTOCOL`         | no        | Test protocol, `tcp` or `udp`.                                           | `tcp` |

### `iperf` Server

This collector requires that an `iperf` server is running on each of the targets to be tested and is available on the default port.

You can start the tool in server mode with `$ iperf3 -s`, but for long-term use you'll want the server to be available all the time. The easiest way to do this is enable an `iperf` background daemon so that it is started at system boot - information on how to do this can be found in the `iperf` man pages, or [here](https://askubuntu.com/questions/1251443/start-iperdf3-deamon-at-startup) for Debian-based/`systemd` systems.

## Quick-Start Docker-Compose Example

```yaml
version: "3.8"

services:
  iperf-prometheus-collector:
    image: ghcr.io/markormesher/iperf-prometheus-collector:VERSION
    restart: unless-stopped
    environment:
      - TARGET_LIST=my-host-01,my-host-02,12.34.56.78
    ports:
      - 9030:9030
```
