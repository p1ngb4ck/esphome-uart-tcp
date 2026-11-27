# esphome-uart-tcp

## Purpose
UARTs over the network: `uart_tcp` puts a UART on a TCP port (server or client), `tcp_uart` gives the device a UART
whose other end is a TCP connection, `noise` encrypts the stream; virtual UARTs (the log as a UART), several
connections per port, forwards configured at runtime from a file, RFC 2217 with DTR / RTS, a log ring for viewers.

## How it works
`uart_tcp` and `tcp_uart` run on `socket` over the device's network (Ethernet or Wi-Fi from esphome-network); the
`noise` stream encrypts a link with a 32-byte key, set in YAML or at runtime. A virtual UART is a ring buffer in PSRAM
that any UART consumer reads; `logger: uart:` makes the log one (boot log kept for the first reader). Settings are
changeable at runtime and saved; `role: forwards` reads its UARTs from a settings file on a storage. The stack builds
on six open upstream pull requests by Bascht74.

## Quick start
Chips: ESP32, ESP32-S2, ESP32-S3, ESP32-P4

Needs: [esphome-network](https://github.com/p1ngb4ck/esphome-network),
[esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform)

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-uart-tcp@main
    components: [uart, uart_tcp, tcp_uart, noise, socket, logger]
  - {source: github://p1ngb4ck/esphome-network@main, components: [network, ethernet]}
  - {source: github://p1ngb4ck/esphome-esp32-platform@main, components: [esp32]}
ethernet: {type: W5500, clk_pin: GPIO18, mosi_pin: GPIO21, miso_pin: GPIO14, cs_pin: GPIO13, interrupt_pin: GPIO4}
uart: [{id: meter_uart, tx_pin: GPIO5, rx_pin: GPIO15, baud_rate: 115200}]
logger: {uart: {id: log_uart}}
uart_tcp: [{role: server, uart_id: meter_uart, port: 6638}, {role: server, uart_id: log_uart, port: 4000}]
```

Use your board's pins (or your Wi-Fi). Port 6638: the meter's UART; port 4000: the device's log (boot log kept).

## Main options
| Option | What it does | Default |
|---|---|---|
| [`uart_tcp: role`](CONFIGURATION.md#uart_tcp) | `server`, `client` or `forwards` (from a file) | required |
| [`uart_tcp: protocol`](CONFIGURATION.md#uart_tcp) | `raw` or `rfc2217` (line settings, DTR / RTS) | `raw` |
| [`encryption: key`](CONFIGURATION.md#uart_tcp) | noise stream key; without it set at runtime | - |
| [`uart_tcp: settings_file`](CONFIGURATION.md#uart_tcp) | the forwards' file | `/sdcard/serial_ports.txt` |
| [`tcp_uart: max_connections`](CONFIGURATION.md#tcp_uart) | server: connections at once, 1-8 | `1` |
| [`logger: uart`](CONFIGURATION.md#logger) | the log as a virtual UART, boot buffer in PSRAM | - |
| [`logger: log_ring_size`](CONFIGURATION.md#logger) | recent log lines in a PSRAM ring for viewers | `0` |
| [`uart: debug_add_uart_settings`](CONFIGURATION.md#uart) | line settings in every debug line | `false` |

## Catalog
- Components (folder on main, configuration):
  - `uart_tcp`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/uart_tcp),
    [configuration](CONFIGURATION.md#uart_tcp), [actions](CONFIGURATION.md#actions)
  - `tcp_uart`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/tcp_uart),
    [configuration](CONFIGURATION.md#tcp_uart)
  - `noise`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/noise),
    configuration under [`encryption`](CONFIGURATION.md#uart_tcp)
  - `socket`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/socket)
  - `uart`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/uart),
    [configuration](CONFIGURATION.md#uart)
  - `logger`: [folder](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main/esphome/components/logger),
    [configuration](CONFIGURATION.md#logger)
- Branches ([`main`](https://github.com/p1ngb4ck/esphome-uart-tcp/tree/main) = the whole topic), in stack order:
  [`pr/uart-debugger-prefix-settings`][b1], [`pr/uart-runtime-line-settings`][b2], [`pr/noise-shared-key`][b3],
  [`pr/uart-tcp-runtime-settings`][b4], [`pr/uart-virtual-ring`][b5], [`pr/logger-virtual-uart`][b6],
  [`pr/uart-tcp-virtual-peer`][b7], [`pr/uart-tcp-forwards`][b8], [`pr/socket-peer-address`][b9],
  [`pr/uart-tcp-logger-getters`][b10], [`pr/uart-tcp-change-callbacks`][b11], [`pr/socket-loop-interval`][b12],
  [`pr/uart-tcp-logger-psram`][b13], [`pr/logger-log-ring`][b14], [`pr/uart-tcp-rfc2217`][b15]
- Needed repos: [esphome-network](https://github.com/p1ngb4ck/esphome-network) (network, ethernet / wifi),
  [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform) (esp32)
- Optional: [esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface) (`role: forwards`)
- All topics together: `github://p1ngb4ck/esphome@dev`

[b1]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-debugger-prefix-settings
[b2]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-runtime-line-settings
[b3]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/noise-shared-key
[b4]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-runtime-settings
[b5]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-virtual-ring
[b6]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/logger-virtual-uart
[b7]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-virtual-peer
[b8]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-forwards
[b9]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/socket-peer-address
[b10]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-logger-getters
[b11]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-change-callbacks
[b12]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/socket-loop-interval
[b13]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-logger-psram
[b14]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/logger-log-ring
[b15]: https://github.com/p1ngb4ck/esphome-uart-tcp/tree/pr/uart-tcp-rfc2217

---

# Detailed version

UARTs over the network for ESPHome: `uart_tcp` puts a device's UART on a TCP port (as server or client), `tcp_uart`
gives the device a UART whose other end is a TCP connection, `noise` encrypts the stream between two devices, and on
top: virtual UARTs (the device's own log as a UART), several connections per port, UART forwards configured at
runtime from a file, RFC 2217 (remote serial ports with line settings and DTR / RTS), and a log ring for viewers.

Chips: ESP32 family (ESP32, ESP32-S2, ESP32-S3, ESP32-P4); runtime line settings (and RFC 2217's line settings) on
ESP32 and ESP8266. Needs a network ([esphome-network](https://github.com/p1ngb4ck/esphome-network)) and `esp32` from
[esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform).

## Examples

A sensor's UART reachable on TCP port 6638:

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-uart-tcp@main
    components: [uart, uart_tcp, tcp_uart, noise, socket, logger]

uart:
  - id: meter_uart
    tx_pin: GPIO17
    rx_pin: GPIO18
    baud_rate: 115200

uart_tcp:
  - role: server
    uart_id: meter_uart
    port: 6638
```

The device's log on TCP port 4000, with the boot log kept for the first client:

```yaml
logger:
  uart:
    id: log_uart

uart_tcp:
  - role: server
    uart_id: log_uart
    port: 4000
```

## More

**[CONFIGURATION.md](CONFIGURATION.md)**: every option - roles, encryption and runtime keys, RFC 2217, forwards and
their settings file, `tcp_uart`, the log as a UART, the log ring, runtime line settings.

## Branches

One branch per change of ours; `main` contains every branch. Base: ESPHome `dev` at `a5a75738ad`.

`uart_tcp`, `tcp_uart`, `noise` and `socket` are upstream components. Our changes need six pull requests by
[Bascht74](https://github.com/Bascht74) that are still open upstream. They are included as esphome-testing carries
them; they are his work, not ours, and have no branch here:

| Upstream pull request (open) | What it adds |
|---|---|
| [#20198](https://github.com/esphome/esphome/pull/20198) | A virtual UART base |
| [#20482](https://github.com/esphome/esphome/pull/20482) | An encrypted byte stream between two devices |
| [#20483](https://github.com/esphome/esphome/pull/20483) | Encrypt tcp_uart with the noise stream |
| [#20484](https://github.com/esphome/esphome/pull/20484) | Encrypt uart_tcp with the noise stream |
| [#20508](https://github.com/esphome/esphome/pull/20508) | Set the encryption key at runtime |
| [#21040](https://github.com/esphome/esphome/pull/21040) | Accept several connections on one port |

Merged upstream by now and so part of the base: [#20097](https://github.com/esphome/esphome/pull/20097) (close a
quiet TCP connection), [#20476](https://github.com/esphome/esphome/pull/20476) (the exclusive UART claim),
[#21039](https://github.com/esphome/esphome/pull/21039) (several links per listener), also by Bascht74.

Our changes, stacked in this order (each contains the one before it; from `pr/noise-shared-key` on they sit on the
six pull requests above):

| Branch | Change |
|---|---|
| `pr/uart-debugger-prefix-settings` | uart: the debugger's prefix through the debug callback on every platform (1) |
| `pr/uart-runtime-line-settings` | uart: line settings settable at runtime and saved (ESP32, ESP8266) |
| `pr/noise-shared-key` | noise: a stream can run under another stream's key |
| `pr/uart-tcp-runtime-settings` | uart_tcp: runtime settings, saved |
| `pr/uart-virtual-ring` | uart: the virtual UART ring in PSRAM, re-allocatable, reader-connected hook |
| `pr/logger-virtual-uart` | logger: the log as a virtual UART (`logger: uart:`) |
| `pr/uart-tcp-virtual-peer` | uart_tcp: a virtual UART is told when the peer comes and goes |
| `pr/uart-tcp-forwards` | uart_tcp `role: forwards`: UARTs forwarded at runtime from a settings file |
| `pr/socket-peer-address` | socket: `TcpClientLink::peer_address()` |
| `pr/uart-tcp-logger-getters` | uart_tcp, logger: connection and buffer getters |
| `pr/uart-tcp-change-callbacks` | uart_tcp: connection and forwards change callbacks |
| `pr/socket-loop-interval` | socket, noise: `get_loop_interval()` in socket |
| `pr/uart-tcp-logger-psram` | uart_tcp, logger: objects in PSRAM, no callback vectors, the UART list in PSRAM |
| `pr/logger-log-ring` | logger `log_ring_size`: the recent lines in a PSRAM ring for viewers |
| `pr/uart-tcp-rfc2217` | uart_tcp `protocol: rfc2217`: RFC 2217 as server and client (2) |

1. Also `debug_add_uart_settings`.
2. Line settings from the peer, DTR / RTS to a usb_uart channel.

The forwards' settings file lives on a storage
([esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface)). DTR / RTS forwarding goes to a
`usb_uart` channel's modem lines; upstream ESPHome's `usb_uart` is enough for it.

All topics together: `github://p1ngb4ck/esphome@dev`. `external_components` replaces components only; the ESPHome
core of the build host stays as installed.

## License

- ESPHome's own files, also where this repository changes them: the [ESPHome License](LICENSE) (MIT for the Python
  code and everything else, GPLv3 for the C++ runtime files).
- New files written for this repository (new components, their code, tests and documentation): MIT,
  [LICENSE-MIT](LICENSE-MIT).

## Thanks

This topic is built on [Bascht74](https://github.com/Bascht74)'s work: the virtual UART base, the exclusive UART
claim, the encrypted noise stream for `tcp_uart` / `uart_tcp`, the runtime key, the idle close, and several links
and connections per port. Thank you for a great collaboration and for all those pull requests. Everyone this project
builds on: [ATTRIBUTIONS.md](https://github.com/p1ngb4ck/esphome/blob/dev/ATTRIBUTIONS.md).

## Branch structure

Each `pr/` branch holds one part of this topic on top of upstream and says in its `BRANCH.md` what it is based on.
`main` holds all of them.

| Branch | Based on |
|---|---|
| `pr/logger-log-ring` | `pr/uart-tcp-logger-psram` |
| `pr/logger-virtual-uart` | `pr/uart-virtual-ring` |
| `pr/noise-shared-key` | `pr/uart-debugger-prefix-settings`, `pr/uart-runtime-line-settings` |
| `pr/socket-loop-interval` | `pr/uart-tcp-change-callbacks` |
| `pr/socket-peer-address` | `pr/uart-tcp-forwards` |
| `pr/uart-debugger-prefix-settings` | upstream only |
| `pr/uart-runtime-line-settings` | upstream only |
| `pr/uart-tcp-change-callbacks` | `pr/uart-tcp-logger-getters` |
| `pr/uart-tcp-forwards` | `pr/uart-tcp-virtual-peer` |
| `pr/uart-tcp-logger-getters` | `pr/socket-peer-address` |
| `pr/uart-tcp-logger-psram` | `pr/socket-loop-interval` |
| `pr/uart-tcp-rfc2217` | `pr/logger-log-ring` |
| `pr/uart-tcp-runtime-settings` | `pr/noise-shared-key` |
| `pr/uart-tcp-virtual-peer` | `pr/logger-virtual-uart` |
| `pr/uart-virtual-ring` | `pr/uart-tcp-runtime-settings` |
