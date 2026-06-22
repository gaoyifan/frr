<!-- PATCHED FORK NOTICE -->
# FRRouting (patched fork) — `mpls lsp <label> dev IFNAME`

This is a patched fork of FRRouting **10.6.1**. It adds support for
interface-only static MPLS LSPs, i.e. an LSP that specifies only an outgoing
interface and no gateway — the equivalent of:

```bash
sudo ip -M route add 18 dev dummy0
```

## New CLI

```
mpls lsp (16-1048575) dev IFNAME
no mpls lsp (16-1048575) dev IFNAME
```

The outgoing label is forced to `implicit-null`, so the kernel pops the label
and forwards out of the interface. `write memory` saves it back as
`mpls lsp <label> dev <ifname>`.

Example:

```bash
sudo vtysh -c 'conf t' -c 'mpls lsp 18 dev dummy0'
sudo vtysh -c 'show mpls table 18'
ip -M route show | grep '^18 '      # -> 18 dev dummy0 proto static
```

## Install from the APT repository (Debian 13 "trixie", amd64)

Prebuilt, signed packages are published to GitHub Pages by the
[`Publish APT repo`](.github/workflows/apt-repo.yml) workflow
(built in a `debian:trixie` container via
[`jtdor/build-deb-action`](https://github.com/jtdor/build-deb-action) and
packaged with [`morph027/apt-repo-action`](https://github.com/morph027/apt-repo-action)).

```bash
# Remove the official FRR repo first to avoid version conflicts (if present):
sudo rm -f /etc/apt/sources.list.d/frr.list

# Trust the repo signing key:
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -sfLo /etc/apt/keyrings/frr-patched.asc https://gaoyifan.github.io/frr/gpg.key

# Add the repository:
echo "deb [signed-by=/etc/apt/keyrings/frr-patched.asc] https://gaoyifan.github.io/frr/ trixie main" \
  | sudo tee /etc/apt/sources.list.d/frr-patched.list

# Install:
sudo apt-get update
sudo apt-get install frr frr-pythontools
```

The patched packages use version `10.6.1-0+mplsdev1`.

### Kernel prerequisites for MPLS

```bash
sudo modprobe mpls_router
sudo sysctl -w net.mpls.platform_labels=100000
# enable MPLS input on the interfaces you use, e.g.:
sudo sysctl -w net.mpls.conf.dummy0.input=1
```

---

<p align="center">
<img src="http://docs.frrouting.org/en/latest/_static/frr-icon.svg" alt="Icon" width="20%"/>
</p>

FRRouting
=========

FRR is free software that implements and manages various IPv4 and IPv6 routing
protocols. It runs on nearly all distributions of Linux and BSD and
supports all modern CPU architectures.

FRR currently supports the following protocols:

* BGP
* OSPFv2
* OSPFv3
* RIPv1
* RIPv2
* RIPng
* IS-IS
* PIM-SM/MSDP
* LDP
* BFD
* Babel
* PBR
* OpenFabric
* VRRP
* EIGRP (alpha)
* NHRP (alpha)

Installation & Use
------------------

For source tarballs, see the
[releases page](https://github.com/FRRouting/frr/releases).

For Debian and its derivatives, use the APT repository at
[https://deb.frrouting.org/](https://deb.frrouting.org/).

Instructions on building and installing from source for supported platforms may
be found in the
[developer docs](http://docs.frrouting.org/projects/dev-guide/en/latest/building.html).

Once installed, please refer to the [user guide](http://docs.frrouting.org/)
for instructions on use.

Community
---------

The FRRouting email list server is located
[here](https://lists.frrouting.org/listinfo) and offers the following public
lists:

| Topic             | List                         |
|-------------------|------------------------------|
| Development       | dev@lists.frrouting.org      |
| Users & Operators | frog@lists.frrouting.org     |
| Announcements     | announce@lists.frrouting.org |

For chat, we currently use [Slack](https://frrouting.slack.com). You can join
by clicking the "Slack" link under the
[Participate](https://frrouting.org/community) section of our website.


Contributing
------------

FRR maintains [developer's documentation](http://docs.frrouting.org/projects/dev-guide/en/latest/index.html)
which contains the [project workflow](http://docs.frrouting.org/projects/dev-guide/en/latest/workflow.html)
and expectations for contributors. Some technical documentation on project
internals is also available.

We welcome and appreciate all contributions, no matter how small!


Security
--------

To report security issues, please use our security mailing list:

```
security [at] lists.frrouting.org
```
