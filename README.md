# Docker Socat Port Forward

This Docker image facilitates the setup of TCP port forwarding using socat, suitable for a variety of networking tasks. It's designed to be simple enough for beginners, while also being robust for more advanced users.

## Features

- Supports the configuration of single or multiple port forwards.
- Multi-arch images make it deployable on various platforms.
- Beginner-friendly documentation with clear examples.
- Demonstrations of different configurations, including port ranges and Docker Compose usage.

## Getting Started

### Prerequisites
- Docker installed on your system.

### Port Configuration

- Ports are mapped using environment variables prefixed with `PORT`. For each port mapping, a unique environment variable is required.
- The format is `LOCAL_PORT:REMOTE_HOST:REMOTE_PORT`. If `LOCAL_PORT` is omitted, it defaults to the same value as `REMOTE_PORT`.
- To handle multiple mappings, assign each to a separate environment variable starting with `PORT`.

### Example

To forward the following TCP ports:
- Remote port `9000` from host `192.168.0.10` to the container's port `9999`
- Remote port `8080` from host `192.168.0.100` to the container's port `8080`, using the same port number for both local and remote

Configure the environment variables as follows:

- `PORT1=9999:192.168.0.10:9000`
- `PORT_B=192.168.0.100:8080`

Run the container with the command:

```bash
docker run -d --name=portforward --net=host -e PORT1="9999:192.168.0.10:9000" -e PORT_B="192.168.0.100:8080" ghcr.io/bearlike/portforward
```

### Port Range Forwarding

Forward a series of consecutive ports by specifying a range in `START-END` format in place of a single port.

Example for forwarding ports `1000` to `1010`:

- Same local and remote port range: `PORTS1=192.168.0.10:1000-1010`
- Different local (`2000-2010`) and remote (`1000-1010`) port ranges: `PORTS2=2000-2010:192.168.0.10:1000-1010`

### SOCKS Proxy Support

Set the `SOCKS_PROXY` environment variable to the `ip:port` of a SOCKSv4 proxy for all port mappings in the container.

```bash
docker run -d --name=portforward --net=host -e SOCKS_PROXY="1.2.3.4:1080" ghcr.io/bearlike/portforward
```

### Docker Compose Examples

#### Exposing Ports  

```yaml
version: '3'
services:
  portforward:
    image: ghcr.io/bearlike/portforward
    network_mode: host
    container_name: portforward
    environment:
      - PORTS1=192.168.1.46:5100-5200
      - PORT2=9002:192.168.1.46:9001
      - PORT3=192.168.1.46:9000
```

### Forwarding ports from a WireGuard Network
The `connect_to_wg` service connects to a WireGuard network using the gluetun image and exposes port 9002, while `portforward` forwards remote port 9001 to container port 9002 and shares network with `connect_to_wg`.

```yaml
version: "3"
services:
  # Use gluetun to connect to a WG network
  connect_to_wg:
    container_name: connect_to_wg
    image: qmcgaw/gluetun
    # Forwarding container (portforward) 9002 to host 9002
    ports:
      - 9002:9002
    cap_add:
      - NET_ADMIN
    # These are placeholder values, modify them to your requirement
    environment:
      - VPN_SERVICE_PROVIDER=custom
      - VPN_TYPE=wireguard
      - VPN_ENDPOINT_IP=${VPN_ENDPOINT_IP}
      - VPN_ENDPOINT_PORT=51820
      - WIREGUARD_PUBLIC_KEY=${WG_PUBLIC_KEY}
      - WIREGUARD_PRIVATE_KEY=${WG_PRIVATE_KEY}
      - WIREGUARD_PRESHARED_KEY=${WG_PRESHARED_KEY}
      - WIREGUARD_ADDRESSES=10.8.0.2/24

  # Portforward will be able to access IPs within the WG network
  portforward:
    image: ghcr.io/bearlike/portforward
    container_name: portforward
    # Forwarding remote 9001 to container 9002
    environment:
      - PORT1=9002:192.168.1.44:9001
    # portward will use the same IP as connect_to_wg 
    network_mode: "service:connect_to_wg"
    depends_on:
      connect_to_wg:
        condition: service_healthy


```

## Changelog

- **0.1.1**
  - Support for port range forwarding using a single socat command per port.
  - Addition of SOCKS proxy support.
  - Integrated tests with GitHub Actions.

- **0.0.2**
  - Bug fixes in socat commands.

- **0.0.1**
  - Initial release with basic port forwarding functionality.

## Acknowledgements

This project was originally forked from [David-Lor/Docker-PortForward](https://github.com/David-Lor/Docker-PortForward). Special thanks to [David-Lor](https://github.com/David-Lor) for their foundational contributions up to version `0.1.1`.