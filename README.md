# gonetwork

[![Go Reference](https://pkg.go.dev/badge/github.com/sonnt85/gonetwork.svg)](https://pkg.go.dev/github.com/sonnt85/gonetwork)

Low-level networking library for Go — ARP, Ethernet II frames, and raw packet sockets.

## Installation

```bash
go get github.com/sonnt85/gonetwork
```

The library is organized into three sub-packages:

| Package | Import path |
|---------|-------------|
| ARP | `github.com/sonnt85/gonetwork/arp` |
| Ethernet | `github.com/sonnt85/gonetwork/ethernet` |
| Raw socket | `github.com/sonnt85/gonetwork/raw` |

## Features

### `arp`
- Send ARP requests and receive replies (RFC 826)
- Resolve an IP address to its hardware (MAC) address
- Send ARP replies (proxy ARP support)
- High-level helpers: `GetMac`, `PingMac` — resolve MAC from any IP/interface

### `ethernet`
- Marshal and unmarshal IEEE 802.3 Ethernet II frames
- Optional IEEE 802.1Q VLAN and Q-in-Q (double-tagged) support
- Frame check sequence (FCS/CRC32) marshal/unmarshal
- Common EtherType constants (IPv4, IPv6, ARP, VLAN)

### `raw`
- Open raw packet sockets via `net.PacketConn` (cross-platform: Linux, BSD)
- BPF filter attachment (`SetBPF`)
- Promiscuous mode toggle
- Packet statistics (Linux)
- Configurable via `Config` (SOCK_DGRAM mode, BPF direction, initial filter)

## Usage

```go
package main

import (
    "fmt"
    "net"

    "github.com/sonnt85/gonetwork/arp"
)

func main() {
    // Resolve a MAC address from an IP (tries all interfaces)
    mac, err := arp.GetMac("192.168.1.1")
    if err != nil {
        panic(err)
    }
    fmt.Println("MAC:", mac)

    // ARP on a specific interface
    ifi, _ := net.InterfaceByName("eth0")
    mac2, _ := arp.PingMac(net.ParseIP("192.168.1.1"), ifi)
    fmt.Println("MAC:", mac2)

    // Low-level ARP client
    c, _ := arp.Dial(ifi)
    defer c.Close()
    hw, _ := c.Resolve(net.ParseIP("192.168.1.254"))
    fmt.Println("Resolved:", hw)
}
```

## API

### `arp`

- `Dial(ifi *net.Interface) (*Client, error)` — opens an ARP client on an interface
- `New(ifi *net.Interface, p net.PacketConn) (*Client, error)` — creates a client with a custom PacketConn
- `(*Client).Request(ip net.IP) error` — sends an ARP request
- `(*Client).Resolve(ip net.IP) (net.HardwareAddr, error)` — sends a request and waits for a reply
- `(*Client).Read() (*Packet, *ethernet.Frame, error)` — reads one incoming ARP packet
- `(*Client).WriteTo(p *Packet, addr net.HardwareAddr) error` — sends an ARP packet to a hardware address
- `(*Client).Reply(req *Packet, hwAddr net.HardwareAddr, ip net.IP) error` — sends an ARP reply
- `(*Client).Close() error` — closes the socket
- `(*Client).SetDeadline/SetReadDeadline/SetWriteDeadline(t time.Time) error` — deadline control
- `GetMac(ip interface{}, timeouts ...time.Duration) (string, error)` — resolve MAC from string or `net.IP`
- `PingMac(ip net.IP, ifi *net.Interface, timeouts ...time.Duration) (string, error)` — resolve MAC on a specific interface
- `PingMacAnyDevice(ip net.IP, ifi *net.Interface, timeouts ...time.Duration) (string, error)` — resolve MAC, trying all suitable devices if the specified interface fails
- `(*Client).HardwareAddr() net.HardwareAddr` — return the hardware address of the client's interface
- `NewPacket(op Operation, srcHW net.HardwareAddr, srcIP net.IP, dstHW net.HardwareAddr, dstIP net.IP) (*Packet, error)` — constructs an ARP packet

### `ethernet`

- `(*Frame).MarshalBinary() ([]byte, error)` — marshal frame to bytes
- `(*Frame).MarshalFCS() ([]byte, error)` — marshal frame with CRC32 FCS
- `(*Frame).UnmarshalBinary(b []byte) error` — unmarshal bytes into frame
- `(*Frame).UnmarshalFCS(b []byte) error` — unmarshal and verify FCS

### `raw`

- `ListenPacket(ifi *net.Interface, proto uint16, cfg *Config) (*Conn, error)` — open a raw socket
- `(*Conn).ReadFrom/WriteTo/Close` — `net.PacketConn` interface
- `(*Conn).SetBPF(filter []bpf.RawInstruction) error` — attach a BPF filter
- `(*Conn).SetPromiscuous(bool) error` — toggle promiscuous mode
- `(*Conn).Stats() (*Stats, error)` — retrieve packet/drop counts (Linux only)

## Author

**sonnt85** — [thanhson.rf@gmail.com](mailto:thanhson.rf@gmail.com)

## License

MIT License - see [LICENSE](LICENSE) for details.
