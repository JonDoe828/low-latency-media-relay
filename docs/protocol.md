# Protocol Overview

## Design Goals

The protocol is designed for a low-latency media relay rather than a general-purpose RPC service. The main goals are:

- compact framing
- incremental decoding from TCP byte streams
- clear separation between control messages and media frames
- enough metadata to route data by `stream_id`

## Message Categories

The protocol can be split into two categories:

- control messages: publish, subscribe, unsubscribe, heartbeat, error
- data messages: media frame transport for a specific stream

## Suggested Frame Layout

A practical binary frame layout for this project is:

1. fixed-size header
2. optional command metadata
3. payload bytes

Example header fields:

- `version`
- `message_type`
- `flags`
- `stream_id_length`
- `payload_length`
- `sequence_number`
- `timestamp`

After the header:

- `stream_id` bytes
- command-specific metadata if needed
- media payload bytes

## Core Commands

### `PUBLISH`

Sent by a publisher to announce the start of a media stream.

Fields:

- `stream_id`
- optional codec or media type metadata

### `SUBSCRIBE`

Sent by a subscriber to join an existing stream.

Fields:

- `stream_id`

### `UNSUBSCRIBE`

Sent by a subscriber before disconnecting or switching streams.

Fields:

- `stream_id`

### `FRAME`

Carries media payload for a published stream.

Fields:

- `stream_id`
- `sequence_number`
- `timestamp`
- frame payload

### `HEARTBEAT`

Used to detect stale connections and keep session state clean.

### `ERROR`

Used by the relay to reject malformed requests or invalid stream operations.

## Session Semantics

- A `stream_id` identifies one logical live stream.
- One publisher is associated with a stream session at a time unless the application explicitly supports takeover.
- Multiple subscribers may attach to the same stream session.
- Frames are forwarded in arrival order within a session.

## Backpressure Expectations

The protocol itself is simple, but the server should define how delivery behaves when subscribers cannot keep up. Reasonable policies include:

- bounded per-subscriber queue
- dropping oldest frames first
- dropping newest frames first for burst protection
- disconnecting persistently slow subscribers

## Resume and Interview Framing

This protocol is intentionally simple enough to implement in a portfolio project, but rich enough to discuss:

- TCP stream framing
- binary serialization
- session routing by stream identifier
- frame sequencing and timing metadata
- slow-consumer handling in media delivery pipelines
