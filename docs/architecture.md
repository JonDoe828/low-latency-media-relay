# Architecture Overview

## Goal

This project models the core data path of a low-latency media relay service: ingest media frames from a publisher, maintain stream session state, and fan out the data to multiple subscribers with bounded buffering and predictable latency.

## Scope

The implementation is intentionally scoped to a single relay node. It focuses on the transport and session-management problems that are highly relevant to media network engineering:

- event-driven socket handling
- efficient connection lifecycle management
- stream-oriented session routing
- subscriber fanout
- backpressure handling

## High-Level Components

### Network Layer

The network layer is based on a Reactor-style event loop built around Linux I/O multiplexing. Its responsibilities are:

- accept inbound TCP connections
- manage readable and writable socket events
- maintain per-connection read and write buffers
- dispatch decoded messages upward to application logic

Main modules:

- `EventLoop`
- `Poller`
- `Channel`
- `Socket`
- `Acceptor`
- `Connection`
- `TcpServer`

### Protocol Layer

The protocol layer defines the on-wire message layout and encodes or decodes commands and media payloads. It separates transport framing from business logic so the server can parse requests incrementally from connection buffers.

Main modules:

- `Message`
- `Command`
- `Codec`

### Stream Layer

The stream layer represents the media-facing business logic. It tracks publishers, subscribers, stream sessions, and delivery policy.

Main modules:

- `StreamManager`
- `StreamSession`
- `Publisher`
- `Subscriber`
- `StreamFrame`
- `BackpressurePolicy`

### App Layer

The app layer wires the network and stream modules together into the relay server entry point and runtime configuration.

Main modules:

- `StreamRelayServer`
- `Config`

## Data Flow

1. A publisher connects to the relay and sends a publish command for a `stream_id`.
2. The server creates or looks up the corresponding `StreamSession`.
3. The publisher pushes media frames into the session.
4. Subscribers connect and subscribe to the same `stream_id`.
5. The relay fans out frames to subscriber connections.
6. If a subscriber falls behind, the backpressure policy decides whether to buffer, drop, or disconnect.

## Key Engineering Tradeoffs

### Why Reactor

Reactor keeps the design close to what many high-performance C++ backend and media transport systems use in practice. It also makes the project a good vehicle for discussing non-blocking I/O, event dispatch, and connection ownership in interviews.

### Why a Custom Binary Protocol

A compact binary protocol is closer to real media delivery paths than a text protocol. It also lets the project demonstrate explicit framing, length-delimited parsing, and command separation between control messages and frame payloads.

### Why Backpressure Matters

Media relay systems are constrained by tail latency and slow consumers. A single subscriber should not be allowed to grow unbounded buffers or degrade overall stream health, so flow control behavior is a first-class design concern.

## Future Extensions

Natural next steps that would strengthen the project further:

- per-stream metrics and observability
- relay benchmark tooling and latency measurement
- configurable frame dropping strategy
- heartbeat and timeout handling
- multi-threaded event loop or worker sharding
- RTP-like sequencing or retransmission hooks
