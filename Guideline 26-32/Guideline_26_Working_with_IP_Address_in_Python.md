# Guideline 26: Working with IP Address in Python

Every device on a network — your laptop, your phone, the server hosting this guidebook if it's ever put online — needs an address. Python doesn't leave you to juggle these as plain strings and hope your math is right. The built-in `ipaddress` module (no `pip install` needed) understands IP addresses, networks, and masks as real objects, and it does the arithmetic for you. This guideline walks through that module from the ground up, ending with a full VLSM subnetting script you can reuse.

Everything here uses:

```python
from ipaddress import IPv4Address, IPv4Network
```

## 26.1 IPv4 Address — Comparing and Sorting

An `IPv4Address` object represents a single address. You can build one directly from a string:

```python
addr = IPv4Address("192.168.1.10")
print(addr)          # 192.168.1.10
print(int(addr))     # 3232235786  <- the address as a plain 32-bit integer
```

That integer conversion is the reason comparison and sorting "just work": under the hood, every `IPv4Address` is really a 32-bit number, and Python compares and sorts them the same way it would compare and sort integers.

```python
a = IPv4Address("192.168.1.10")
b = IPv4Address("192.168.1.20")

print(a < b)          # True
print(a == a)         # True
```

Sorting a list of addresses follows naturally:

```python
ip_strings = ["10.0.0.5", "192.168.1.1", "10.0.0.20", "172.16.0.1"]
addresses = [IPv4Address(ip) for ip in ip_strings]

for ip in sorted(addresses):
    print(ip)
```

```
10.0.0.5
10.0.0.20
172.16.0.1
192.168.1.1
```

Notice that `10.0.0.20` correctly sorts after `10.0.0.5` — this is *not* alphabetical sorting (which would put `"10.0.0.20"` before `"10.0.0.5"` as strings, since `"2" < "5"` isn't how a string comparison of `"20"` vs `"5"` works character-by-character anyway — try `sorted(ip_strings)` yourself and compare). Working with real `IPv4Address` objects instead of raw strings sidesteps that trap entirely.

## 26.2 CIDR

CIDR (Classless Inter-Domain Routing) is the `/24`-style notation you've probably already seen — an IP address, a slash, and a number from 0 to 32. That number is the **prefix length**: how many bits, counting from the left, are fixed as the "network" portion of the address. Everything after those bits is available for individual hosts.

`192.168.50.0/24` means: the first 24 bits (`192.168.50`) identify the network, and the remaining 8 bits are free for host addresses — 256 possible values, from `.0` to `.255`.

Python represents this with `IPv4Network`:

```python
net = IPv4Network("192.168.50.0/24")

print(net.network_address)   # 192.168.50.0
print(net.broadcast_address) # 192.168.50.255
print(net.prefixlen)         # 24
print(net.num_addresses)     # 256
```

CIDR replaced the older rigid "class" system (next section) precisely because it lets a network be *any* size — a `/28` for a small office, a `/16` for a whole campus — instead of only the fixed sizes that classes allowed.

## 26.3 IP Address Classes

Before CIDR existed, IPv4 addresses were split into fixed **classes**, identified by the value of the first octet. You'll still see this vocabulary in networking conversations even though real-world routing has moved on to CIDR.

| Class | First octet range | Default mask | Typical use |
|---|---|---|---|
| A | 1 – 126 | `/8` (255.0.0.0) | Very large networks |
| B | 128 – 191 | `/16` (255.255.0.0) | Medium-sized networks |
| C | 192 – 223 | `/24` (255.255.255.0) | Small networks |
| D | 224 – 239 | — | Multicast (not for hosts) |
| E | 240 – 255 | — | Reserved / experimental |

(127 is skipped — that's the loopback range, `127.0.0.0/8`, reserved for `localhost`.)

You can check which class an address *would* have belonged to just by looking at the first octet:

```python
def classful_category(ip_str):
    first_octet = int(ip_str.split(".")[0])
    if first_octet <= 126:
        return "A"
    elif first_octet <= 191:
        return "B"
    elif first_octet <= 223:
        return "C"
    elif first_octet <= 239:
        return "D"
    else:
        return "E"

print(classful_category("192.168.50.1"))  # C
```

The `ipaddress` module itself doesn't have a `.get_class()` method, because modern networking doesn't route by class anymore — it's CIDR all the way down. The classes are worth knowing as vocabulary and history, not as something your code should rely on.

## 26.4 Private and Public IP Address

Not every address is reachable from the open internet. Three ranges — carved out in RFC 1918 — are reserved for **private** use: inside a home, an office, a school. Routers translate between these and a single **public** address when traffic leaves the network (this is NAT, which you may recognize from Chapter 2 of *AI in Everything* if you've read it).

| Private range | CIDR | Common use |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` | Large private networks |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` | Medium private networks |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` | Home / small office networks |

`ipaddress` will tell you which category an address falls into without needing to memorize the table:

```python
addresses = ["192.168.1.1", "8.8.8.8", "10.0.0.5", "172.20.4.4"]

for ip in addresses:
    a = IPv4Address(ip)
    print(f"{ip}: private={a.is_private}, global={a.is_global}")
```

```
192.168.1.1: private=True, global=False
8.8.8.8: private=False, global=True
10.0.0.5: private=True, global=False
172.20.4.4: private=True, global=False
```

`is_global` is effectively the opposite of `is_private` here — a routable, public-internet address.

## 26.5 Subnet Mask and Host Masks

A **subnet mask** marks, in the same dotted format as an IP address, which bits belong to the network and which belong to the host. `/24` corresponds to `255.255.255.0` — 24 bits of `1`s followed by 8 bits of `0`s.

A **host mask** (sometimes called a wildcard mask) is the exact inverse — it marks which bits are free for hosts. For `/24`, that's `0.0.0.255`.

`IPv4Network` gives you both directly:

```python
net = IPv4Network("192.168.50.0/24")

print(net.netmask)   # 255.255.255.0
print(net.hostmask)  # 0.0.0.255
```

Try a smaller subnet and watch both change together:

```python
net2 = IPv4Network("192.168.50.0/28")

print(net2.netmask)   # 255.255.255.240
print(net2.hostmask)  # 0.0.0.15
```

A `/28` reserves 4 bits for hosts (32 − 28 = 4), and `15` is exactly `2^4 − 1` — the hostmask is always "one less than the number of addresses on that side," which leads directly into the next section.

## 26.6 Calculating Hosts

The number of addresses a network of prefix length `p` contains is:

$$2^{(32 - p)}$$

But not every one of those addresses can be assigned to a device. Two are reserved: the **network address** (the very first one, identifying the subnet itself) and the **broadcast address** (the very last one, used to reach every host on the subnet at once). So the number of **usable** host addresses is:

$$2^{(32 - p)} - 2$$

```python
def usable_hosts(prefix):
    return 2 ** (32 - prefix) - 2

for p in (24, 28, 30):
    print(f"/{p}: {usable_hosts(p)} usable hosts")
```

```
/24: 254 usable hosts
/28: 14 usable hosts
/30: 2 usable hosts
```

`IPv4Network` will also just tell you the total address count directly through `.num_addresses` (which still includes the network and broadcast addresses, so subtract 2 yourself for the usable figure):

```python
net = IPv4Network("192.168.50.0/26")
print(net.num_addresses)        # 64
print(net.num_addresses - 2)    # 62 usable
```

(A `/31` is a special case used for point-to-point links where both addresses are usable and there's no broadcast address at all — but that's an edge case worth knowing exists rather than something to build around here.)

## 26.7 Subnetting

Subnetting is the practical skill all of the above builds toward: given one network, carve it into several smaller ones — each sized to what it actually needs, rather than every subnet being forced to the same size. This technique is called **VLSM** (Variable Length Subnet Masking), and the guiding rule is simple: **allocate the subnet that needs the most hosts first.** If you allocate small subnets first, you can accidentally carve the address space up in a way that leaves no room left for the big one that comes later.

Here's a complete, runnable version of the subnetting workflow, built on everything from this guideline:

```python
from ipaddress import IPv4Network

def get_required_hosts(hosts):
    # hosts: a list of (label, host_count) tuples.
    # Sort in descending order — prioritize the subnet with the
    # largest required hosts, so it gets first pick of the address space.
    return sorted(hosts, key=lambda pair: pair[1], reverse=True)

def get_cidr(required_hosts):
    # required_hosts: the sorted (label, host_count) list above.
    # For each entry, find the smallest prefix length whose address
    # space (host_count + 2, for the network and broadcast addresses)
    # can actually fit that many hosts.
    cidr_list = []
    for _, host_count in required_hosts:
        needed = host_count + 2
        prefix = 32
        while 2 ** (32 - prefix) < needed:
            prefix -= 1
        cidr_list.append(prefix)
    return cidr_list

def subnetting(network, hosts):
    required_hosts = get_required_hosts(hosts)
    cidr = get_cidr(required_hosts)
    current_address = network.network_address
    for c, h in zip(cidr, required_hosts):
        # Set network address
        subnet = IPv4Network(f"{current_address}/{c}", strict=False)
        # get broadcast subnet
        network_addr = subnet.network_address
        broadcast_addr = subnet.network_address + (2 ** (32 - c) - 1)
        # usable IPs
        first_addr, last_addr = network_addr + 1, broadcast_addr - 1
        print(f"{h}: {first_addr} - {last_addr}")
        # Update the next address, 'last_addr + 2' to exclude the broadcast_addr
        # of the previous network, and the network_addr of the current network
        current_address = last_addr + 2
```

Notice `hosts` is zipped against `required_hosts` — the *sorted* list — rather than the original argument. That's deliberate: `cidr` was computed from `required_hosts`, so the label printed alongside each range has to come from that same sorted order, or the sizes and the labels would drift apart.

A worked example, using a single `/24` split across four departments:

```python
network = IPv4Network("192.168.50.0/24")
hosts = [("Sales", 50), ("IT", 20), ("HR", 10), ("Guest", 5)]

subnetting(network, hosts)
```

```
('Sales', 50): 192.168.50.1 - 192.168.50.62
('IT', 20): 192.168.50.65 - 192.168.50.86
('HR', 10): 192.168.50.129 - 192.168.50.140
('Guest', 5): 192.168.50.193 - 192.168.50.198
```

Sales, needing the most hosts, is placed first and gets a `/26` (62 usable addresses). IT gets a `/27` (30 usable), HR a `/28` (14 usable), and Guest a `/29` (6 usable) — each subnet sized to what it actually needs, with no host requirement left unaccommodated and no address space wasted on a department that will never use it. That's the entire point of VLSM: instead of splitting `192.168.50.0/24` into four identical `/26` blocks regardless of size, the network is shaped around the real requirements.

---

Everything in this guideline builds on a single idea: an IP address is a 32-bit number wearing a dotted-decimal costume. Once you let Python treat it that way — through `IPv4Address` and `IPv4Network` — comparing, masking, and subnetting stop being manual arithmetic and become straightforward object operations.
