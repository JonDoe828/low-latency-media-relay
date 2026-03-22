# Low-Latency Media Relay

A C++ real-time media relay project designed for media network and live streaming backend roles. The system targets low-latency publisher-to-subscriber fanout on top of an event-driven networking stack built with the Reactor pattern and Linux I/O multiplexing.

## Positioning

This project is intentionally framed as a media relay instead of a full CDN or globally distributed media platform. That makes the scope accurate for a resume project while still demonstrating the core engineering concerns behind real-time media transport:

- event-driven network programming in C++
- publisher-subscriber stream fanout
- custom binary protocol design
- backpressure-aware delivery
- low-latency stream session management

## Target Scenario

The relay accepts media frames from publishers, maintains per-stream session state, and forwards data to multiple subscribers with predictable latency and bounded buffering behavior.

## Why This Name

`Low-Latency Media Relay` is a better fit than `stream distribution` for this repository because the current scope is a single relay/fanout service rather than a full multi-region distribution network.

## Resume-Friendly Summary

Built a low-latency media relay system in C++ based on an event-driven Reactor architecture, supporting publisher-subscriber stream fanout, custom transport protocol handling, and backpressure-aware delivery.

## Design Docs

- [Architecture](docs/architecture.md)
- [Protocol](docs/protocol.md)
