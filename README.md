# iptables Firewall Lab

A Docker-based simulation demonstrating network segmentation and traffic filtering using `iptables`. This lab features a router connecting a LAN, DMZ, and WAN, with specific rules controlling access between them.

## Architecture

| Node | Description | IP Address |
|------|-------------|------------|
| **Router** | Linux gateway with `iptables` rules | `10.1.1.1` (LAN), `192.168.10.1` (DMZ), `172.16.1.1` (WAN) |
| **LAN Client** | Internal network client | `10.1.1.10` |
| **DMZ Server** | Nginx web server | `192.168.10.10` |
| **WAN Client** | External user | `172.16.1.100` |
| **WAN Attacker** | Blacklisted IP | `172.16.1.200` |

## Firewall Rules

*   **Default Policy:** Drop all incoming and forwarded traffic.
*   **Blacklist:** All traffic from `172.16.1.200` is dropped.
*   **LAN → DMZ:** SSH (port 22) is blocked; all other traffic is allowed.
*   **WAN → DMZ:** Only HTTP (port 80) and HTTPS (port 443) are allowed. All other traffic (including ICMP/ping) is dropped.

## Prerequisites

*   [Docker](https://www.docker.com/) installed and running.
*   [Docker Compose](https://docs.docker.com/compose/) (v2.0+).

## Getting Started

### 1. Start the Lab
Build and launch all containers in the background:

```bash
docker compose up -d --build
```

## Verify the Setup
Check if the router loaded the rules correctly:
```bash
docker exec <lab-name>-router iptables -L FORWARD -v -n
```

## Test the Firewall
Test 1: Web Access (Should Work) From the WAN Client, access the DMZ server on port 80:

```bash
docker exec <lab-name>-wan-client curl http://192.168.10.10
```
Test 2: Ping (Should Fail) ICMP is blocked by the final drop rule in the WAN-to-DMZ chain:
```bash
docker exec <lab-name>-wan-client ping -c 4 192.168.10.10
```
Test 3: Blacklisted Attacker (Should Fail) All traffic from the attacker IP is dropped:
```bash
docker exec <lab-name>-wan-attacker curl http://192.168.10.10
```
## Destroying the Lab
To stop and remove all containers and networks:
```bash
docker compose down
```
If you also want to remove any built images or volumes:
```bash
docker compose down --rmi local --volumes
```




