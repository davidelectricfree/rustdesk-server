# RustDesk Server Patched (secure_tcp)

Fork of [rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server) with **secure_tcp KeyExchange** support backported.

## What This Fixes

The official open-source hbbs server returns `NOT_SUPPORT` when a client with login credentials attempts a TCP connection. This causes the client to hang with **"Failed to secure tcp"** and the connection never establishes.

This patch adds the KeyExchange handshake protocol, allowing TCP connections to be encrypted end-to-end even without the Pro license.

## Changes Summary

Patches applied on top of official master (815c728):

| Commit | Description |
|--------|-------------|
| **Patch 1** | Sink type refactoring: SafeWsSink/SafeTcpStreamSink with optional Encrypt context |
| **Patch 2** | KeyExchange protocol: phase1/phase2 handshake + RegisterPk TCP support |

See commit history for detailed diffs.

## Quick Deploy

```bash
# Set your key from existing SPK install
export RUSTDESK_KEY=$(cat /var/packages/RustDeskServer/etc/key)

# Start
docker compose up -d
```

Ports used: 21115-21119 (host network mode)

## Building from Source

```bash
git clone --recurse-submodules https://github.com/davidelectricfree/rustdesk-server.git
cd rustdesk-server
docker build -f docker/Dockerfile.build -t rustdesk-server-patched .
```

## Protocol Flow

```
Client ──TCP connect──> Server:21116
Server ──[Phase1: signed box_ pubkey]──> Client
Client ──[Phase2: client pubkey + encrypted sym key]──> Server
         Subsequent messages encrypted with NaCl secretbox
```
