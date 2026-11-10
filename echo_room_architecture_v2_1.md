<div align="center">
  <h1>Echo Room – System Architecture and Requirements Document</h1>
</div>

> **Document version:** 2.1
> **Document status:** Draft for review
> **Document purpose:** This document records the agreed design for Echo Room. Use this document to plan development. Use this document to check that all features and limits are correct before you write code. This version replaces the peer-to-peer runtime with a decentralized Cloudflare D1 stack. This document uses ASD-STE100 style. Sentences are short. Voice is active. Terms are defined.

---

## 1. Purpose and Scope

### 1.1 Purpose

Echo Room is a desktop application for professional audio collaboration. It targets podcasters, voice actors, directors, and producers. It supports live audio. It supports optional screen sharing. It does not support camera video. It uses standard Web Real-Time Communication (WebRTC) for media. It uses a decentralized Cloudflare D1 stack for signaling and room discovery. It does not use TURN relays. TURN means Traversal Using Relays around NAT. NAT means Network Address Translation.

### 1.2 Scope

This document covers:
- The user features.
- The decentralized host stack.
- The WebRTC connection design.
- The Selective Forwarding Unit (SFU) tree design.
- The bandwidth limits.
- The failure recovery design.
- The notification system.
- The visual feedback design.
- The security and privacy model.
- The licensing model.
- The implementation plan.

### 1.3 Out of Scope

This document does not cover:
- Camera video streaming.
- Central media servers.
- TURN media relays.
- Public web browser support.
- Mobile application support.
- The retired Pear and Bear stack.

### 1.4 What Changed in Version 2.1

> [!NOTE]
> Version 1.0 used Pear, Bear, Hyperswarm, and HyperDHT. Version 2.1 removes these components. Version 2.1 uses a decentralized Cloudflare D1 stack. Each host deploys their own Worker and database. The host shares their own Uniform Resource Locator (URL). The developer is not in the data path.

Version 2.1 also corrects all ASD-STE100 compliance issues found in version 2.0. It expands every abbreviation at first use. It replaces passive voice with active voice. It removes "will" and "should" from requirements. It shortens long sentences. It fixes synonym drift.

This change affects the discovery layer, the signaling layer, and the file transfer layer. It does not affect the media layer. The WebRTC media path remains peer-to-peer.

The next section gives a high-level view of the new system.

---

## 2. System Overview

### 2.1 What Echo Room Does

Echo Room connects users in a virtual room. Users speak to each other. Users share screens. Users play soundboard sounds. Users view a shared document. Users see profile photos. The system sends audio and screen data directly between peers. The system does not relay media through a central server.

### 2.2 Main Technologies

Echo Room uses these technologies:
- **Electron**: The desktop application wrapper.
- **WebRTC**: The media transport for audio and screen share.
- **STUN**: The service that discovers public IP addresses. STUN means Session Traversal Utilities for NAT.
- **Cloudflare Workers**: The per-host signaling Application Programming Interface (API).
- **Cloudflare D1**: The per-host SQLite database. SQLite means a lightweight embedded database engine.
- **Wrangler CLI**: The deployment tool for Workers and D1. CLI means Command-Line Interface.
- **IndexedDB**: The local metadata store.
- **File I/O**: The local file store for audio bytes, documents, and photos. I/O means Input/Output.
- **pdf.js**: The Portable Document Format (PDF) renderer.

> [!WARNING]
> Echo Room does not use Pear, Bear, Hyperswarm, HyperDHT, Hypercore, or the Bare worker.

### 2.3 High-Level Data Flow

1. A user starts Echo Room.
2. The host deploys their own Worker and D1 database. The host shares their URL.
3. A guest opens the host URL in Echo Room.
4. The guest requests to join the room through the host Worker.
5. The host approves or denies the request.
6. The Worker stores the approval in D1.
7. Peers exchange Session Description Protocol (SDP) offers and answers through the Worker.
8. STUN discovers public IP addresses.
9. WebRTC establishes direct media connections.
10. The SFU tree assigns forwarding roles.
11. Audio flows directly between peers and SFU nodes.
12. Screen shares flow only to users who subscribe to them.
13. Soundboard files stream over the WebRTC data channel.
14. Document sync messages flow over the WebRTC data channel.

### 2.4 The Decentralized Principle

> [!IMPORTANT]
> Each host owns their infrastructure. The host deploys their own Worker. The host deploys their own D1 database. The host shares their own URL. The developer does not run a central server. The developer does not pay for host infrastructure. The developer does not see host data.

The next section states the design principles that guide every decision in this document.

---

## 3. Key Design Principles

### 3.1 No Central Media Server
Echo Room does not use a central media server. Media travels directly between peers. This design removes server bandwidth costs. This design removes relay latency.

### 3.2 No TURN Relay
Echo Room does not use TURN relays. STUN discovers public addresses. WebRTC hole punching creates direct connections. The system accepts a small failure rate on restrictive networks. This design keeps the system at zero cost.

### 3.3 Decentralized Host Ownership
Each host deploys their own Worker and D1 database. The host shares their own URL. The developer is not in the data path. This design removes developer infrastructure cost. This design gives the host data sovereignty.

### 3.4 Automatic Role Assignment
The system assigns SFU roles automatically. Users do not volunteer. The system uses measured upload speed and current load. This design prevents user error. This design prevents overload.

### 3.5 Strict Bandwidth Caps
Each SFU peer has a hard cap of 15 Mbps upload and 15 Mbps download. Mbps means Megabits per second. This cap protects consumer internet connections. This cap forces a scalable tree structure.

### 3.6 Fast Connection Setup
The system limits Interactive Connectivity Establishment (ICE) candidate gathering. ICE means Interactive Connectivity Establishment. A candidate means an address that a peer can try. The system targets a connection time of 5 seconds or less. This design prevents long waits. This design improves the user experience.

### 3.7 User-Owned Data
Echo Room stores all user data in a user-selected folder. The system does not use a developer system folder. The user controls sound files, profile photos, and cache files.

### 3.8 Consistent Visual Feedback
Every asynchronous operation shows a visible loading indicator. Every system message appears through a toast notification. The application does not use modal prompts for system messages. This design keeps the user informed. This design keeps the interface responsive.

### 3.9 No Freemium Limits
Echo Room does not limit call length. Echo Room does not limit participant count. Echo Room does not limit core features. Paid features are add-ons or business tools.

These principles shape every component that follows. The next section defines the terms used throughout this document.

---

## 4. Definitions and Terms

| Term | Definition |
| :--- | :--- |
| **ASD-STE100** | ASD Simplified Technical English, Issue 100. A controlled language standard for technical documents. |
| **Candidate** | An ICE address that a peer can try to connect to. |
| **Cloudflare D1** | A serverless SQLite database from Cloudflare. |
| **Cloudflare Worker** | A serverless function from Cloudflare. |
| **Co-host** | A user that the host promotes. A co-host can share a camera feed. |
| **Data channel** | A WebRTC channel for reliable, ordered messages. |
| **D1** | Short form of Cloudflare D1. |
| **Echo Room** | The name of this application. |
| **Edge SFU** | An SFU peer that serves a group of end-users. |
| **End-user** | A user that sends audio and receives audio. An end-user does not forward media. |
| **Host URL** | The Worker URL that the host deploys and shares. |
| **ICE** | Interactive Connectivity Establishment. The WebRTC connection protocol. |
| **IndexedDB** | A browser-based local database. Electron supports it. |
| **Jitter buffer** | A local buffer that smooths packet arrival timing. |
| **Mbps** | Megabits per second. A unit of network speed. |
| **Message** | The payload that a subsystem sends to the toast service. |
| **Opus** | An audio codec. Echo Room uses Opus for voice. |
| **Peer** | A user device in the room. |
| **Profile photo** | A small image that represents a user. |
| **Room** | A virtual meeting space. A room has a topic and a host. |
| **Screen share** | A live video stream of a user's screen. |
| **SDP** | Session Description Protocol. A text format for WebRTC handshakes. |
| **SFU** | Selective Forwarding Unit. A peer that forwards media streams. |
| **Soundboard** | A collection of short audio files that users can play. |
| **Standby SFU** | A peer that is ready to become an SFU if an active SFU fails. |
| **STUN** | Session Traversal Utilities for NAT. The public-address discovery service. |
| **Toast** | A small, non-blocking UI element that shows a message. |
| **TURN** | Traversal Using Relays around NAT. A media relay service. |
| **WebRTC** | Web Real-Time Communication. A set of protocols for direct media. |
| **Worker** | Short form of Cloudflare Worker. |
| **Wrangler** | The command-line tool for Cloudflare Workers and D1. |

### 4.1 Abbreviation Register

This section lists every abbreviation that the document uses. This section shows where the full term first appears in the body text.

| Abbreviation | Full Term | First Use Location |
| :--- | :--- | :--- |
| AEC | Acoustic Echo Cancellation | Section 10.4 |
| AGC | Automatic Gain Control | Section 10.4 |
| API | Application Programming Interface | Section 2.2 |
| ASD-STE100 | ASD Simplified Technical English, Issue 100 | Section 1.1 |
| B2B | Business to Business | Section 23.4 |
| CLI | Command-Line Interface | Section 2.2 |
| CPU | Central Processing Unit | Section 22.4 |
| D1 | Cloudflare D1 | Section 1.1 |
| DTLS | Datagram Transport Layer Security | Section 21.3 |
| FEC | Forward Error Correction | Section 10.2 |
| HTTPS | Hypertext Transfer Protocol Secure | Section 21.4 |
| ICE | Interactive Connectivity Establishment | Section 3.6 |
| I/O | Input/Output | Section 2.2 |
| JSON | JavaScript Object Notation | Section 23.3 |
| Mbps | Megabits per second | Section 3.5 |
| NAT | Network Address Translation | Section 1.1 |
| NS | Noise Suppression | Section 10.4 |
| OAuth | Open Authorization | Section 6.1 |
| PDF | Portable Document Format | Section 2.2 |
| PLC | Packet Loss Concealment | Section 25.3 |
| QR | Quick Response | Section 6.1 |
| RTCP | RTP Control Protocol | Section 17.1 |
| SDP | Session Description Protocol | Section 2.3 |
| SFU | Selective Forwarding Unit | Section 1.2 |
| SHA | Secure Hash Algorithm | Section 12.2 |
| SIMD | Single Instruction Multiple Data | Section 25.10 |
| SQL | Structured Query Language | Section 25.22 |
| SQLite | A lightweight embedded database engine | Section 2.2 |
| SRTP | Secure Real-time Transport Protocol | Section 21.3 |
| SSO | Single Sign-On | Section 23.4 |
| STUN | Session Traversal Utilities for NAT | Section 1.1 |
| TLS | Transport Layer Security | Section 21.4 |
| TTL | Time To Live | Section 7.3 |
| TURN | Traversal Using Relays around NAT | Section 1.1 |
| UDP | User Datagram Protocol | Section 9.1 |
| UI | User Interface | Section 5.4 |
| URL | Uniform Resource Locator | Section 1.4 |
| VAD | Voice Activity Detection | Section 10.4 |
| WebRTC | Web Real-Time Communication | Section 1.1 |

### 4.2 Abbreviation Rules

Use an abbreviation only after you write its full term. Write the full term first. Then write the abbreviation in parentheses. Example: "The Selective Forwarding Unit (SFU) forwards media."

Repeat the full term at first use in each section. Do not rely on Section 4 alone.

Do not use an abbreviation in a section title unless Section 4.1 lists it and the section body writes the full term in its first paragraph.

Keep the register in Section 4.1 current. Add a row when you add an abbreviation.

These definitions apply to all sections. The next section describes the users and their permissions.

---

## 5. User Roles and Permissions

### 5.1 Host
The host creates the room. The host deploys the Worker and D1 database. The host has these permissions:
- Start and end the room.
- Share a screen. The host is always an SFU peer.
- Share a camera feed.
- Add sounds to the room soundboard.
- Set soundboard permissions for guests.
- Control the document viewer scroll position.
- Control the document viewer highlights.
- Promote a guest to co-host.
- Remove a user from the room.
- Deploy and update the host Worker.

### 5.2 Co-host
The host promotes a co-host. A co-host has these permissions:
- Share a screen.
- Share a camera feed.
- Control the document viewer if the host grants control.
- Play soundboard sounds for all users if the host grants permission.

### 5.3 Guest
A guest joins the room. A guest has these permissions:
- Send audio.
- Receive audio.
- Watch screen shares. A guest must subscribe to a screen share.
- View the document.
- View profile photos.
- Play soundboard sounds according to host permissions.
- Save soundboard sounds to a personal soundboard.
- Set a profile photo.

### 5.4 Permission Enforcement
The application enforces all permissions. The application does not rely on user honesty. The host changes permissions through the host control panel. The system sends permission changes over the data channel. All peers update their local permission state when they receive a change message. The UI (User Interface) shows the current permission state in the host panel.

Permissions depend on the room being active. The next section explains the lifecycle of a room, from creation to end.

---

## 6. Room Lifecycle

### 6.1 Host Setup Before Room Creation

The host must deploy their Worker and D1 database one time. The application provides a setup wizard.

1. The host clicks "Set Up My Echo Room Server".
2. The application opens a browser window to the Cloudflare login page.
3. The host logs in or creates a free Cloudflare account.
4. The application receives an Open Authorization (OAuth) token through a local callback.
5. The application runs Wrangler commands. The commands are `d1 create`, `d1 execute --file=schema.sql`, and `deploy`.
6. The application reads the resulting Worker URL.
7. The application stores the URL in the local configuration.
8. The application shows the URL and a Quick Response (QR) code. The message is: *"This is your room server. Share this link with your guests."*

The application shows a progress bar during each step. The application shows a spinner on the button. The application shows a success toast when the deployment completes.

### 6.2 Room Creation

1. The host opens Echo Room.
2. The host selects "Create Room".
3. The application generates a room ID. The room ID is a random string.
4. The application sends a `POST /rooms` request to the host Worker.
5. The Worker stores the room in D1.
6. The application shows the room view.
7. The host shares the URL. The URL includes the room ID.

### 6.3 Room Join

1. A guest opens Echo Room.
2. The guest pastes the host URL or scans the QR code.
3. The application sends a `POST /rooms/:id/join` request. The request includes the guest's public key and display name.
4. The Worker stores the join request in D1 with status `pending`.
5. The host sees a pending join message. The message appears as a toast with an "Approve" button.
6. The host clicks "Approve".
7. The Worker updates the join status to `approved`.
8. The guest polls the Worker for status. The poll interval is 500 ms. ms means milliseconds.
9. The guest sees `approved`. The guest enters the room.

The application shows a "Waiting for host approval" spinner during step 8. The spinner shows elapsed time.

### 6.4 Room End

1. The host selects "End Room".
2. The application broadcasts a room-end message.
3. The application sends a `DELETE /rooms/:id` request to the Worker.
4. The Worker marks the room as ended in D1.
5. All peers close their connections.
6. The application shows a toast: *"Room ended by host."*

### 6.5 Host Disconnection

If the host disconnects unexpectedly, the room closes after 30 seconds. The application does not support host migration in version 2.1. This limit prevents split-brain rooms. The host Worker remains online. Guests see a message: *"Host disconnected. Room closes in 30 seconds."*

The room lifecycle depends on the host Worker. The next section describes the decentralized host stack in detail.

---

## 7. The Decentralized Host Stack

### 7.1 Purpose
The host stack replaces the Pear and Bear discovery layer. The stack has two parts: a Cloudflare Worker and a Cloudflare D1 database. Each host deploys both to their own Cloudflare account. The developer runs no part of this stack.

### 7.2 The Worker

The Worker is a small Hypertext Transfer Protocol (HTTP) API. HTTP means Hypertext Transfer Protocol. The Worker exposes these endpoints:

| Method | Path | Purpose |
| :--- | :--- | :--- |
| `POST` | `/rooms` | Create a room. |
| `GET` | `/rooms/:id` | Get room state. |
| `DELETE` | `/rooms/:id` | End a room. |
| `POST` | `/rooms/:id/join` | Request to join. |
| `POST` | `/rooms/:id/approve` | Approve a join request. |
| `POST` | `/rooms/:id/deny` | Deny a join request. |
| `PUT` | `/signals` | Store an SDP payload or ICE candidate. |
| `GET` | `/signals/:room` | Poll for SDP payloads. |
| `PUT` | `/manifests/:room` | Store the soundboard manifest. |
| `GET` | `/manifests/:room` | Get the soundboard manifest. |
| `PUT` | `/presence/:room` | Update peer heartbeat. |
| `GET` | `/presence/:room` | Get the list of active peers. |

The Worker validates all input. The Worker rejects malformed requests with a `400` status. The Worker rejects unauthorized requests with a `403` status.

### 7.3 The D1 Database

The D1 database stores five tables. The schema is fixed at deployment time.

```sql
Table: rooms
  id TEXT PRIMARY KEY
  host_pubkey TEXT
  created_at INTEGER
  expires_at INTEGER
  status TEXT            -- 'active', 'ended'

Table: room_peers
  room_id TEXT
  peer_pubkey TEXT
  display_name TEXT
  role TEXT              -- 'host', 'co-host', 'guest'
  status TEXT            -- 'pending', 'approved', 'denied'
  last_seen INTEGER
  PRIMARY KEY (room_id, peer_pubkey)

Table: signals
  id TEXT PRIMARY KEY
  room_id TEXT
  from_peer TEXT
  to_peer TEXT
  payload TEXT
  kind TEXT              -- 'offer', 'answer', 'ice'
  created_at INTEGER
  expires_at INTEGER

Table: manifests
  room_id TEXT PRIMARY KEY
  payload TEXT
  updated_at INTEGER

Table: presence
  room_id TEXT
  peer_pubkey TEXT
  last_seen INTEGER
  PRIMARY KEY (room_id, peer_pubkey)
```

The `signals` table has a 5-minute Time To Live (TTL). A scheduled Worker task deletes expired rows. The `presence` table has a 30-second TTL. A peer that does not send a heartbeat for 30 seconds is treated as offline.

### 7.4 Deployment Through Wrangler

The application bundles Wrangler. The application runs Wrangler commands in a child process. The application captures `stdout` and `stderr`. The application shows the output in a collapsible log panel. The panel opens if an error occurs.

The application runs these commands in order:

1. `wrangler whoami` — Check the account.
2. `wrangler d1 create echo-room-<short-id>` — Create the database.
3. `wrangler d1 execute echo-room-<short-id> --file=schema.sql --remote` — Apply the schema.
4. `wrangler deploy` — Publish the Worker.
5. `wrangler deployments list` — Read the Worker URL.

Each command shows a spinner next to its name. Each command shows a green checkmark when it completes. Each command shows a red cross if it fails. The application shows a toast for the overall result.

### 7.5 Cost

The Cloudflare free tier covers typical usage. A podcast host who runs 2 to 4 hours of calls per week stays well within the limits. The limits are:
- 100,000 Worker requests per day.
- 5,000,000 D1 rows read per day.
- 100,000 D1 rows written per day.
- 5 GB D1 storage.

The host pays nothing unless they exceed these limits. The developer pays nothing at any time.

The host stack handles discovery and signaling. The next section explains how peers use it.

---

## 8. Peer Discovery and Signaling

### 8.1 Room Discovery
The host shares their URL directly. The URL has this form: `https://echo-room-<short-id>.<subdomain>.workers.dev/room/<room-id>`. The guest pastes the URL into Echo Room. The application parses the URL and extracts the Worker base and the room ID.

There is no central registry. There is no global room list. Each host URL is a closed ecosystem. This design removes all developer involvement in discovery.

### 8.2 Presence Heartbeat
Each peer sends a `PUT /presence/:room` request every 10 seconds. The request updates the `last_seen` value. The Worker considers a peer offline if `last_seen` is older than 30 seconds. The Worker returns the list of active peers on `GET /presence/:room`. All peers poll this endpoint every 5 seconds.

### 8.3 Signaling Flow
Signaling uses the Worker as a temporary mailbox. The flow is:

1. Peer A creates an `RTCPeerConnection`.
2. Peer A creates an SDP offer.
3. Peer A sends `PUT /signals` with `kind = 'offer'`.
4. Peer B polls `GET /signals/:room` every 500 ms.
5. Peer B receives the offer.
6. Peer B creates an SDP answer.
7. Peer B sends `PUT /signals` with `kind = 'answer'`.
8. Peer A polls and receives the answer.
9. The WebRTC connection proceeds.

The application deletes each signal after the recipient reads it. The Worker deletes any unread signal after 5 minutes.

### 8.4 ICE Candidate Exchange
The application exchanges ICE candidates through the same `signals` table with `kind = 'ice'`. The application sends each candidate as a separate payload.

The application limits candidate gathering. Section 9 explains the limit.

### 8.5 Polling Limits
The application polls with exponential backoff. The first poll is at 500 ms. The second poll is at 750 ms. Later polls are at 1000 ms. The application stops polling after 30 seconds. If no answer arrives, the application shows a toast: *"Connection timed out. Try again."*

### 8.6 Message Size
Each signal payload is under 10 KB. KB means kilobytes. An SDP offer is typically 2 KB to 5 KB. An ICE candidate is under 200 bytes. The Worker rejects payloads over 100 KB.

The signaling layer prepares peers for media. The next section explains how the media connection forms.

---

## 9. WebRTC Connection Setup and ICE

### 9.1 STUN Configuration

The application uses these STUN servers. It tries them in order:

1. `stun:stun.l.google.com:19302`
2. `stun:stun1.l.google.com:19302`
3. `stun:stun.cloudflare.com:3478`

The application does not configure a TURN server. The `RTCConfiguration` has `iceTransportPolicy: 'all'`. The `RTCConfiguration` has no `turn` entries. UDP means User Datagram Protocol.

### 9.2 ICE Candidate Limit

The application limits ICE candidate gathering to a maximum of 10 candidates per connection. The application stops gathering when the limit is reached. The application stops gathering after 3 seconds, whichever comes first.

The application uses these settings:

```javascript
{
  iceCandidatePoolSize: 0,
  bundlePolicy: 'max-bundle',
  rtcpMuxPolicy: 'require',
  iceServers: [ /* ...STUN list... */ ]
}
```

The application ignores host candidates that come from virtual network adapters. The application keeps server-reflexive candidates and relay candidates. The application sorts candidates by priority. The application sends the top 10 candidates to the peer.

### 9.3 Connection Time Target
The application targets a connection time of 5 seconds or less. The application measures the time from the first ICE candidate to the `connected` state. If the time exceeds 5 seconds, the application logs a warning. If the time exceeds 15 seconds, the application fails the connection. The application shows a toast: *"Connection failed. Your network may block peer-to-peer traffic."*

### 9.4 Connection State Messages

The application shows connection state through toasts. It uses these messages:

| State | Toast text | Toast type |
| :--- | :--- | :--- |
| `new` | "Connecting..." | info |
| `checking` | "Finding a route..." | info |
| `connected` | "Connected" | success |
| `disconnected` | "Connection unstable" | warning |
| `failed` | "Connection failed" | error |
| `closed` | (no toast) | — |

### 9.5 ICE Restart
If a connection drops, the application performs an ICE restart. The application generates new ICE credentials. The application sends a new offer through the Worker. The application repeats the signaling flow. The application restarts candidate gathering with the same 10-candidate limit.

### 9.6 Network Change Handling
If the peer's network changes, the application detects the change through the `iceconnectionstatechange` event. The application performs an ICE restart. The application keeps the audio buffer. The application does not drop the call unless the restart fails three times.

### 9.7 Failure Handling
If the connection fails after three ICE restarts, the application shows a toast: *"Cannot connect. Your network blocks peer-to-peer connections. Echo Room requires a direct path."* The application logs the failure. The application does not retry indefinitely.

Once the connection forms, audio flows directly. The next section explains audio capture and playback.

---

## 10. Audio Capture and Playback

### 10.1 Audio Capture
Echo Room uses the Web Audio API. Echo Room uses `getUserMedia` with these constraints:
- `echoCancellation: true`
- `noiseSuppression: true`
- `autoGainControl: true`
- `sampleRate: 48000`
- `channelCount: 1`

The application captures mono audio. Mono audio halves the bitrate. Mono audio is sufficient for voice.

### 10.2 Audio Codec
Echo Room uses Opus. The target bitrate is 128 Kbps. Kbps means kilobits per second. The application sets the bitrate in the SDP. The application uses a 20 ms frame size. The application enables in-band Forward Error Correction (FEC). FEC helps with packet loss.

### 10.3 Audio Playback
Echo Room uses the Web Audio API for playback. Each remote peer has a dedicated audio node. The application mixes all remote audio streams. The application applies a 2-second jitter buffer. The jitter buffer absorbs network timing variation. The jitter buffer hides brief disconnections.

### 10.4 Audio Processing
The application applies these processes:
- **Acoustic Echo Cancellation (AEC)**: Removes speaker feedback.
- **Noise Suppression (NS)**: Reduces background noise.
- **Automatic Gain Control (AGC)**: Keeps volume steady.
- **Voice Activity Detection (VAD)**: Detects speaking. VAD drives the speaking indicator.

### 10.5 Audio Mixing for SFU Peers
An SFU peer receives multiple audio streams. The SFU mixes these streams into one stream per downstream peer. Mixing reduces upstream bandwidth. The SFU uses a simple summing mixer. The mixer clips at full scale to prevent distortion.

### 10.6 Headphone Warning
> [!WARNING]
> An SFU peer must use headphones. Speaker playback creates echo in the mix. The application detects speaker output through the audio device label. The application shows a toast when an SFU peer uses speakers: *"Use headphones to prevent echo. You are forwarding audio for other users."*

Audio is the primary stream. Screen sharing is the secondary stream. The next section explains screen sharing.

---

## 11. Screen Sharing

### 11.1 Screen Share Capture
Echo Room uses `getDisplayMedia` in the renderer process. The application captures system audio when available. Electron allows system audio capture. Browser environments often block system audio capture. This capability is a key Electron advantage.

### 11.2 Screen Share Limits
The application enforces a maximum of 3 simultaneous screen shares per room. The application enforces this limit at the room level. A user cannot start a fourth screen share. If a user requests a fourth screen share, the application shows a toast: *"Three screen shares are active. Wait for one to stop."*

### 11.3 Screen Share Quality
The application targets 2 Mbps to 3 Mbps per screen share. The application uses VP8 or VP9. VP8 and VP9 are video codecs. The application uses 15 frames per second. The application prioritizes text clarity. The application reduces frame rate before it reduces resolution. This trade-off favors document readability.

### 11.4 Selective Subscription
Screen shares are optional streams. A user must click "Watch" on a screen share to receive it. The application does not send a screen share to a user who does not subscribe. This design saves bandwidth. The SFU forwards a screen share only to subscribers. The SFU uses a separate forwarding branch for each screen share.

The application shows a "Watching" badge on the screen share that the user has subscribed to. The application shows a spinner while the first frame loads.

### 11.5 Screen Share Audio
The application captures screen share audio as a separate audio track. The application mixes screen share audio with the sharer's microphone audio. The application marks screen share audio with a different audio level. The UI shows a "Screen Audio" indicator.

### 11.6 Screen Share Start Feedback
When a user starts a screen share, the application shows a toast: *"Sharing your screen. Others can now watch."* The application shows a red border around the shared window. The application shows a timer with the share duration.

### 11.7 Screen Share Stop
When a user stops a screen share, the application shows a toast: *"Screen share stopped."* The application removes the shared stream from all subscribers. The application releases the capture handle.

Screen sharing adds a second stream type. The next section adds a lightweight visual identity for each user.

---

## 12. Profile Photos

### 12.1 Profile Photo Format
The application accepts JPEG, PNG, and WebP images. JPEG, PNG, and WebP are image formats. The application resizes images to 256 by 256 pixels. The application compresses images to under 100 KB. The application stores the profile photo in the user folder.

### 12.2 Profile Photo Exchange
When a user joins a room, the application announces the profile photo hash. The hash is a Secure Hash Algorithm 256-bit (SHA-256) digest. If a peer does not have the photo, the peer requests the file over the WebRTC data channel. The application caches the photo in `~/EchoRoom/Cache/Photos/` and records the metadata in IndexedDB.

### 12.3 Profile Photo Display
The application shows the profile photo in a circular avatar. The application shows a speaking indicator. The speaking indicator is a colored ring. The ring color changes when the user speaks. The application shows the user's display name below the photo.

### 12.4 Loading State
While a profile photo downloads, the application shows a placeholder. The placeholder is a gray circle with the first letter of the display name. The application shows a small spinner in the corner of the circle. When the photo loads, the application fades the image in over 200 ms.

### 12.5 Photo Change
If a user changes their profile photo during a room, the application announces the new hash. Other peers request the new file over the data channel. The application replaces the old photo. The application deletes the old cache entry.

Profile photos identify users. The next section adds sound, which brings session personality and utility.

---

## 13. Soundboard System

### 13.1 Soundboard Storage

The soundboard files live in the user folder: `~/EchoRoom/Sounds/`. The application stores a manifest file: `~/EchoRoom/Sounds/manifest.json`. JSON means JavaScript Object Notation. The manifest contains:
- Sound ID.
- File name.
- Display name.
- Duration in seconds.
- SHA-256 hash.
- File size in bytes.

The application stores the manifest copy in IndexedDB for fast lookup.

### 13.2 Adding a Sound

1. The user clicks "Add Sound".
2. The application opens a file picker.
3. The user selects an audio file. Supported formats: MP3, WAV, OGG, FLAC. These are audio file formats.
4. The application copies the file to `~/EchoRoom/Sounds/`.
5. The application generates a unique sound ID.
6. The application adds an entry to the manifest.
7. The application updates the UI.
8. The application shows a toast: *"Sound added: <display name>."*

The application shows a progress bar during the file copy. The application shows a spinner on the "Add Sound" button.

### 13.3 Soundboard Sharing
The host shares the soundboard manifest with all guests. The manifest travels over the WebRTC data channel. The manifest lists sound IDs and hashes.

A guest checks their local cache for each sound. If a sound is missing, the guest requests it over the data channel. The host sends the file in chunks. The guest writes each chunk to `~/EchoRoom/Cache/Sounds/`. The guest records the completed file in IndexedDB.

The SFU tree accelerates this transfer. The host uploads each file once to their SFU children. The SFU children forward the file to their own children. Total host upload is one copy per file. Total transfer time for a 5 MB file across 100 peers is 1 to 3 seconds. MB means megabytes.

### 13.4 Sound Playback
When a user plays a sound, the application sends a data channel message: `{"type":"play_sound","sound_id":"abc123","target":"all"}`. The target field can be `all`, `self`, or `host_only`. The application plays the sound locally using the Web Audio API. The application does not stream the sound audio through WebRTC. The sound file is already cached locally. This design avoids media bandwidth for soundboard sounds.

The sound playback latency is under 150 ms. This latency is acceptable for sound effects.

### 13.5 Soundboard Permissions

The host sets one of three permission modes:
- **Host-Only**: Only the host can play sounds. Guests see the soundboard but cannot play sounds.
- **Guest Self-Only**: Guests can play sounds for themselves. Other users do not hear the sound.
- **Guest Broadcast**: Guests can play sounds for all users. The host can revoke this permission at any time.

The host changes the permission mode through the host control panel. The application broadcasts the permission change over the data channel. All peers update their UI immediately. The application shows a toast to all users: *"Soundboard permissions changed to <mode>."*

### 13.6 Saving a Sound to a Personal Soundboard
A guest can save a room sound to their personal soundboard. The guest clicks "Save to My Soundboard". The application copies the cached file to `~/EchoRoom/Sounds/`. The application adds a manifest entry. The sound is available in future rooms that the guest hosts. The application shows a toast: *"Saved to My Soundboard."*

### 13.7 Soundboard Loading Feedback
The application shows a loading state for each sound in the soundboard panel. The state has three values:
- **Missing**: Gray. A download icon is shown.
- **Downloading**: Spinner overlay. A progress bar is shown.
- **Ready**: Normal color. The play button is enabled.

The application shows a small spinner on the play button while the sound plays.

Sound brings utility. Documents bring collaboration. The next section explains the shared document viewer.

---

## 14. Document Viewer and Synchronization

### 14.1 Document Sources
The document viewer supports two sources:
- **PDF file**: The host uploads a PDF file. The application renders the PDF with `pdf.js`.
- **Google Docs link**: The host pastes a public Google Docs URL. The application renders the document in a sandboxed iframe. An iframe is an embedded browser frame. The application falls back to PDF if the iframe fails.

### 14.2 PDF Transfer
The host selects a PDF file. The application transfers the PDF to all guests over the WebRTC data channel. The application uses the SFU tree. The host uploads one copy. The SFU children forward the file. The PDF file size is typically 100 KB to 500 KB. The transfer completes in under one second on a normal connection.

The application shows a progress bar during the transfer. The progress bar shows the percentage complete. The bar fills from left to right. The application shows the message *"Loading document..."* under the bar.

### 14.3 Document Rendering
The application uses `pdf.js` to render PDF pages. The application renders pages on demand. The application supports text selection. The application supports highlight overlays. The application supports scroll position tracking.

The application shows a skeleton loader while the first page renders. The skeleton loader is a gray rectangle that matches the page shape. When the page is ready, the application fades it in over 200 ms.

### 14.4 Scroll Synchronization
The host controls the scroll position. When the host scrolls, the application sends a data channel message: `{"type":"doc_scroll","page":3,"offset":0.45}`. The `offset` field is a fraction from 0.0 to 1.0. Guests receive the message and scroll their view to the same position. The application applies a smooth scroll animation. The animation duration is 200 ms.

### 14.5 Highlight Synchronization
The host can highlight text. The host selects a highlight color. The application sends a data channel message: `{"type":"doc_highlight","start":1200,"end":1450,"color":"#FF0000","page":3}`. The `start` and `end` fields are character offsets in the extracted text. Guests receive the message and apply the highlight. The application supports these colors: yellow, green, blue, red, and purple.

### 14.6 Personal Highlights
A guest can add personal highlights. Personal highlights are local only. The application does not broadcast personal highlights. Personal highlights help voice actors mark their own lines.

### 14.7 Host Cursor
The application shows a host cursor in the document viewer. The cursor is a colored vertical line. The cursor shows where the host is currently reading. The application sends cursor position updates every 500 ms.

### 14.8 Document Synchronization Bandwidth
Document sync messages are tiny. A scroll message is about 100 bytes. A highlight message is about 200 bytes. A cursor message is about 80 bytes. The total bandwidth is under 10 Kbps. Document sync does not affect audio quality.

### 14.9 Google Docs Fallback
Google Docs iframe embedding often fails due to Cross-Origin Resource Sharing (CORS). CORS means Cross-Origin Resource Sharing. The application detects iframe failure. The application shows a toast: *"Google Docs cannot be embedded. Please upload a PDF instead."* The application does not proxy Google Docs content.

The document viewer serves the host's script. The SFU tree carries every stream. The next section explains the SFU tree in detail.

---

## 15. SFU Tree Architecture and Auto-Election

### 15.1 SFU Definition
A Selective Forwarding Unit (SFU) is a peer that forwards media streams. An SFU receives audio from its children. An SFU mixes the audio. An SFU sends the mixed audio to its children. An SFU forwards screen shares to subscribers. An SFU forwards data channel messages to its children.

### 15.2 SFU Cap
Each SFU has a hard cap of 15 Mbps upload and 15 Mbps download. The application measures the SFU's current load. The application does not assign a new child if the load would exceed the cap. The application uses the cap to prevent overload.

### 15.3 Bandwidth Test
Every peer runs a bandwidth test on join. The test uses a temporary UDP echo server. The test sends 1 MB of data. The test measures the upload speed. The test takes 2 to 3 seconds. The test result determines the peer's role.

The application shows a progress ring during the bandwidth test. The ring is next to the peer's name. The ring fills as the test proceeds. The application shows the message *"Testing connection..."* The application shows a toast when the test completes: *"Your connection: <N> Mbps upload."*

### 15.4 Role Assignment Rules
The application assigns roles with these rules:
1. The host is always an SFU peer.
2. A peer with upload speed greater than 15 Mbps is an SFU candidate.
3. The top 3 fastest SFU candidates (excluding the host) become Standby SFUs.
4. Other peers become end-users.
5. The application re-evaluates roles every 60 seconds.

### 15.5 Parent Selection
An end-user selects a parent SFU with these steps:
1. The end-user broadcasts a "need parent" message on the data channel.
2. Each SFU replies with its current load percentage.
3. The end-user selects the SFU with the lowest load.
4. The end-user establishes a WebRTC connection to that SFU.

The application shows a spinner while the parent is selected. The application shows a toast when the parent is chosen: *"Connected to <SFU display name>."*

### 15.6 Load Balancing
An SFU monitors its load. If the load exceeds 80% of the cap, the SFU sends a "rebalance" message to its children. The message tells some children to find a new parent. The children repeat the parent selection process. The application does not interrupt audio during rebalancing. The application uses a make-before-break handover.

### 15.7 Standby SFU Promotion
If an active SFU disconnects, the host promotes a Standby SFU. The host sends a "promote" message to the Standby SFU. The Standby SFU starts accepting children. The disconnected SFU's children reconnect to the new SFU.

### 15.8 Tree Depth
The application limits the tree depth to 3 hops. A depth of 3 hops gives a latency of 15 to 45 ms. The application does not allow deeper trees. Deeper trees increase latency. Deeper trees increase failure impact.

### 15.9 SFU Count Formula
The application calculates the needed SFU count with this formula:

`SFU_count = ceil(total_audio_bandwidth / 12 Mbps)`

The application uses 12 Mbps as the usable audio bandwidth per SFU. The application reserves 3 Mbps for screen share forwarding.

The SFU tree decides who forwards media. The next section quantifies the bandwidth and capacity limits.

---

## 16. Bandwidth Math and Capacity Planning

### 16.1 Constants

| Constant | Value |
| :--- | :--- |
| Audio bitrate per stream | 128 Kbps (0.128 Mbps) |
| Screen share bitrate | 2 Mbps |
| SFU upload cap | 15 Mbps |
| SFU download cap | 15 Mbps |
| Reserved bandwidth for screen share | 3 Mbps |
| Usable audio bandwidth per SFU | 12 Mbps |

### 16.2 Single SFU Capacity

| Active Screen Shares | Usable Audio Bandwidth | Max Direct Audio Peers |
| :--- | :--- | :--- |
| 0 | 15 Mbps | 117 |
| 1 | 12 Mbps | 93 |
| 2 | 9 Mbps | 70 |
| 3 | 6 Mbps | 46 |

### 16.3 Room Capacity Table

| Participants | Active Screens | SFUs Needed | User Download | User Upload | Max Latency |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 10 | 0 | 1 | 1.28 Mbps | 0.128 Mbps | <5 ms |
| 10 | 1 | 1 | 3.28 Mbps | 0.128 Mbps | <5 ms |
| 100 | 0 | 1 | 12.8 Mbps | 0.128 Mbps | <10 ms |
| 100 | 3 | 2 | 12 Mbps | 0.128 Mbps | <15 ms |
| 500 | 0 | 5 | 12.8 Mbps | 0.128 Mbps | ~20 ms |
| 500 | 3 | 11 | 13 Mbps | 0.128 Mbps | ~25 ms |
| 1000 | 0 | 10 | 12.8 Mbps | 0.128 Mbps | ~30 ms |
| 1000 | 3 | 22 | 13 Mbps | 0.128 Mbps | ~35 ms |

### 16.4 Latency Budget

| Source | Latency |
| :--- | :--- |
| Audio capture | 10 ms |
| Opus encoding | 20 ms |
| Network hop (per hop) | 5–15 ms |
| Jitter buffer | 2000 ms (buffer, not latency) |
| Opus decoding | 20 ms |
| Audio playback | 10 ms |
| **Total one-hop latency** | **65–75 ms** |
| **Total three-hop latency** | **85–105 ms** |

The jitter buffer adds delay but hides network variation. The application targets a total latency under 150 ms. Human perception notices latency above 150 ms. Echo Room stays below this threshold for rooms up to 1000 users.

### 16.5 Connection Setup Time

| Step | Target Time |
| :--- | :--- |
| STUN discovery | < 1 s |
| ICE candidate gathering (capped at 10) | < 3 s |
| Signaling exchange through Worker | < 2 s |
| WebRTC connection established | < 5 s total |

The application logs any connection that exceeds 5 seconds. The application fails any connection that exceeds 15 seconds.

### 16.6 Bandwidth for Non-Media Features

| Feature | Bandwidth |
| :--- | :--- |
| Document scroll sync | < 1 Kbps |
| Document highlight sync | < 1 Kbps |
| Soundboard trigger | ~100 bytes per trigger |
| Soundboard file transfer | 1–5 MB once per file |
| Profile photo transfer | 50–200 KB once per user |
| SFU role messages | < 1 Kbps |
| Presence heartbeat | < 1 Kbps |

Non-media features do not affect media quality. The application prioritizes media traffic.

### 16.7 Host Worker Cost

| Metric | Typical Use | Free Tier Limit |
| :--- | :--- | :--- |
| Worker requests | ~2,000 per day | 100,000 per day |
| D1 rows read | ~10,000 per day | 5,000,000 per day |
| D1 rows written | ~500 per day | 100,000 per day |
| D1 storage | < 50 MB | 5 GB |

A typical host stays well within the free tier.

Bandwidth is planned. Failures still happen. The next section explains failure detection and recovery.

---

## 17. Failure Detection and Recovery

### 17.1 Failure Types

| Failure Type | Detection Method | Detection Time |
| :--- | :--- | :--- |
| SFU peer disconnects | Data channel `close` event and ICE state change | < 1 s |
| Network packet loss | RTP Control Protocol (RTCP) receiver reports | 1–2 s |
| ICE connection failure | `iceConnectionState` change to `failed` | 2–5 s |
| Host disconnects | Worker presence heartbeat timeout | < 30 s |
| Worker unavailable | HTTP timeout | 5 s |
| STUN unreachable | ICE gathering timeout | 3 s |

### 17.2 SFU Failover Procedure

1. An SFU disconnects.
2. The host detects the disconnect through the data channel `close` event.
3. The host selects a Standby SFU.
4. The host sends a "promote" message to the Standby SFU.
5. The Standby SFU starts accepting children.
6. The disconnected SFU's children detect the disconnect.
7. The children send "need parent" messages.
8. The new SFU replies with its load.
9. The children connect to the new SFU.
10. The process completes in 1 to 2 seconds.

The application shows a toast to affected users: *"Reconnecting to a new forwarder..."*

### 17.3 Jitter Buffer During Failover
The jitter buffer holds 2 seconds of audio. A 2-second failover fits inside the buffer. Users hear a brief volume dip. Users do not hear a full dropout. The application logs the failover event.

### 17.4 Host Failover
Version 2.1 does not support host failover. If the host disconnects, the room closes after 30 seconds. The application shows a toast: *"Host disconnected. Room closes in 30 seconds."* The application saves the room state locally. The host can resume the room later.

### 17.5 Worker Failure
If the host Worker is unavailable, the application cannot complete signaling. The application retries 3 times. The application shows a toast: *"Server unavailable. Check your internet connection."* The application does not fall back to TURN.

If the Worker is unreachable but peers are already connected, the application keeps the media streams active. The application shows a toast: *"Server unreachable. Existing connections remain active."*

### 17.6 STUN Failure
If all STUN servers fail, the application cannot discover public addresses. The application attempts each STUN server in order. The application waits 3 seconds per server. If all servers fail, the application shows a toast: *"Cannot reach STUN servers. Check your firewall settings."*

### 17.7 Network Change
If a peer's network changes (for example, Wi-Fi to Ethernet), the peer performs an ICE restart. The peer generates new ICE credentials. The peer re-establishes connections. The application keeps the audio buffer. The application does not drop the call.

### 17.8 ICE Candidate Cap During Reconnection
The application applies the same 10-candidate cap during reconnection. This design prevents long reconnection times. The target reconnection time is 5 seconds or less.

Failures are visible to users. The next section explains how the application communicates status.

---

## 18. The Notification and Toast System

### 18.1 Purpose
The application uses a single notification system. The system shows all user messages. The system does not use modal prompts. The system does not use browser `alert()` or `confirm()`. The system uses inline toasts. A toast is a small, non-blocking UI element that shows a message.

### 18.2 Toast Anatomy
A toast has these parts:
- **Icon**: A small image that shows the toast type.
- **Title**: A short line of text. Maximum 40 characters.
- **Body**: An optional line of text. Maximum 120 characters.
- **Actions**: Optional buttons. Maximum 2 buttons per toast.
- **Close button**: A small "X" on the right side.
- **Progress bar**: An optional bar that shows auto-dismiss timing.

### 18.3 Toast Types

| Type | Color | Icon | Default Duration |
| :--- | :--- | :--- | :--- |
| **info** | Blue | Information circle | 4 s |
| **success** | Green | Check mark | 3 s |
| **warning** | Yellow | Warning triangle | 6 s |
| **error** | Red | Error circle | 8 s |
| **progress** | Gray | Spinner | Manual dismiss |

Progress toasts do not auto-dismiss. The application dismisses them when the operation completes.

### 18.4 Toast Placement
Toasts appear in the bottom-right corner of the window. Toasts stack upward. The maximum visible toasts is 5. Older toasts move up when new ones appear. The application queues toasts that exceed 5.

### 18.5 Toast Service Architecture
The application has a central ToastService. The service uses a publish-subscribe pattern.

The service has these methods:
- `publish(message)`: Any subsystem calls this to raise a message.
- `subscribe(callback)`: The renderer calls this to receive messages.
- `dismiss(id)`: Any subsystem calls this to remove a specific toast.
- `dismissAll()`: Any subsystem calls this to clear all toasts.

Each message has this shape:

```typescript
{
  id: string,
  type: 'info' | 'success' | 'warning' | 'error' | 'progress',
  title: string,
  body?: string,
  duration?: number,
  actions?: [
    { label: string, callback: function }
  ],
  sticky?: boolean
}
```

The ToastService assigns the `id`. The publisher provides the other fields.

### 18.6 Subsystem Subscription
Any subsystem can publish messages. No subsystem needs to know about the toast UI. For example:
- The connection manager publishes `{type: 'info', title: 'Connecting...'}`.
- The soundboard publishes `{type: 'success', title: 'Sound added'}`.
- The document viewer publishes `{type: 'warning', title: 'Google Docs cannot be embedded'}`.
- The SFU manager publishes `{type: 'error', title: 'Connection failed'}`.

The toast UI subscribes once. The UI receives all messages. The UI renders the toast. The publishing subsystem does not change.

This design lets new subsystems add notifications without touching the toast code.

### 18.7 Accessibility
Each toast has `role="status"` or `role="alert"`. Screen readers announce the toast automatically. Info, success, and progress toasts use `role="status"`. Warning and error toasts use `role="alert"`.

The close button has an accessible label: *"Dismiss notification."*

### 18.8 Pause on Hover
The auto-dismiss timer pauses when the user hovers over a toast. The timer resumes when the user moves the pointer away. This design gives the user time to read.

### 18.9 Deduplication
If the same toast message publishes more than once within 2 seconds, the application updates the existing toast. The application does not stack duplicates. The application increments a small "×2" counter on the toast.

### 18.10 Logging
Every toast event writes a line to the log file. The log line contains the timestamp, type, title, and body. The log does not contain action callbacks.

Toasts communicate state changes. Loading indicators communicate ongoing work. The next section explains loading indicators.

---

## 19. Visual Loading and Progress Indicators

### 19.1 Purpose
The application must never leave the user guessing. Every asynchronous operation shows visible feedback. The user must see that work is happening. The user must see when work completes.

### 19.2 Loading Indicator Types
The application uses these indicator types:

| Indicator | When Used |
| :--- | :--- |
| **Spinner** | Short operations under 3 seconds. Button actions. |
| **Progress bar** | Known-length operations. File transfers, deployments. |
| **Progress ring** | Bandwidth tests, connections. |
| **Skeleton loader** | Content that has a known shape but unknown timing. Lists, pages. |
| **Inline text** | "Loading…" text next to a label. |
| **Button state** | Disabled button with a spinner. |

### 19.3 Button Loading State
When a user clicks a button that starts an async action, the button:
1. Becomes disabled.
2. Shows a spinner on the left side of the label.
3. Keeps its label text.
4. Returns to the normal state when the action completes.

The application does not change button size when showing a spinner. The application reserves space for the spinner.

### 19.4 Page-Level Loading
When a full page loads, the application shows a centered spinner. The spinner is a rotating ring. Below the spinner, the application shows a short message: *"Loading room…"*, *"Loading document…"*, or similar.

### 19.5 List Loading
When a list loads (peers, sounds, rooms), the application shows skeleton rows. Each row is a gray rectangle that matches the height and width of the final content. When the data arrives, the skeleton rows are replaced with the real rows in a fade-in over 200 ms.

### 19.6 File Transfer Progress
File transfers show a determinate progress bar. The bar has a percentage label on the right. The bar has the file name on the left. The bar fills from left to right. If the transfer fails, the bar turns red and shows "Failed" with a retry button.

### 19.7 Connection Progress
Connection setup shows a progress ring around the peer's avatar. The ring fills as the connection state advances:
- **0%**: Not started.
- **25%**: STUN discovery.
- **50%**: ICE gathering.
- **75%**: Signaling exchange.
- **100%**: Connected.

The application shows a small text label next to the ring: *"Connecting…"*, *"Finding a route…"*, *"Almost there…"*, *"Connected."*

### 19.8 Deployment Progress
The host setup wizard shows a step list. Each step has an icon and a label. The current step shows a spinner. Completed steps show a green checkmark. Failed steps show a red cross. The application shows a log panel below the list. The panel opens automatically on failure.

### 19.9 Empty States
When a list is empty, the application shows an empty state. The empty state has an icon, a short message, and an action button. For example: *"No sounds yet. [Add your first sound]."*

### 19.10 Error States
When an operation fails, the application shows an error state in place of the content. The error state has an icon, a short message, and a retry button. The application also shows an error toast.

### 19.11 Timeout Handling
Every async operation has a timeout. When the timeout expires, the application shows a toast: *"Operation took too long. Try again."* The application shows an error state in the affected area. The application logs the timeout.

| Operation | Timeout |
| :--- | :--- |
| Connection setup | 15 s |
| Signaling poll | 30 s |
| File transfer | 60 s |
| Worker request | 10 s |
| STUN discovery | 3 s per server |

### 19.12 Reduced Motion
If the user enables "Reduce Motion" in the operating system, the application disables spinning and fading animations. Spinners become static progress bars. Transitions become instant.

Loading indicators and toasts both support the user. Both feed the log. The next section explains local storage.

---

## 20. Local Storage, IndexedDB, and File Management

### 20.1 User Folder
The application asks the user to select a folder on first launch. The default folder is `~/Documents/EchoRoom/`. The application creates these subfolders:
- `Sounds/`: Soundboard audio files.
- `Profile/`: Profile photo.
- `Cache/`: Cached PDFs, sounds, and photos.
- `Logs/`: Application logs.
- `Recordings/`: Local recordings.

### 20.2 IndexedDB Usage
The application uses IndexedDB for metadata. IndexedDB stores:
- Sound manifests.
- Profile photo metadata (hash, size, path).
- Room state cache (last room, last participants).
- SDP cache (recent offers and answers, cleared after success).
- Document metadata (page count, last scroll position).

The application does not store large binary files in IndexedDB. Binary files live in the file system.

### 20.3 File System Usage
The application uses the Node.js `fs` module through the preload bridge. The file system stores:
- Audio files (soundboard, cache).
- PDF files.
- Profile photos.
- Logs.
- Recordings.
- Configuration.

The application always uses the user-selected folder. The application never writes to a developer-owned folder.

### 20.4 Cache Management
The application limits the cache to 500 MB. The application deletes the oldest cached files when the cache exceeds the limit. The application does not delete soundboard files. The application does not delete profile photos. The user can clear the cache from the settings panel. The application shows a progress bar during cache clearing.

### 20.5 Logging
The application writes logs to `Logs/echo-room.log`. The application rotates logs daily. The application keeps 7 days of logs. Logs contain:
- Connection events.
- SFU role changes.
- Failover events.
- Toast events.
- Timeout events.
- Error messages.

Logs do not contain audio data. Logs do not contain personal messages.

### 20.6 Configuration
The application stores `Config.json` in the user folder. The config file contains:
- Display name.
- Profile photo hash.
- Preferred audio input device.
- Preferred audio output device.
- Host Worker URL.
- Last used room topic.
- Notification preferences.

Local data supports the user. Security and privacy protect the user. The next section explains both.

---

## 21. Security and Privacy

### 21.1 Identity
Each installation generates an Ed25519 key pair. Ed25519 is a digital signature algorithm. The public key is the user's identity. The application does not use email addresses. The application does not use passwords. The application does not use OAuth for core use.

### 21.2 Room Access
Room access uses host approval. The guest requests to join. The host approves or denies. The application does not allow anonymous join. The application does not allow public rooms in version 2.1.

### 21.3 Media Encryption
WebRTC uses Datagram Transport Layer Security (DTLS) and Secure Real-time Transport Protocol (SRTP). DTLS provides key exchange. SRTP provides media encryption. All media is encrypted in transit. The application does not support unencrypted media. The encryption keys are ephemeral. The encryption keys are not stored.

### 21.4 Signaling Encryption
The Worker uses Hypertext Transfer Protocol Secure (HTTPS) for all requests. HTTPS means Hypertext Transfer Protocol Secure. Transport Layer Security (TLS) protects the connection. Cloudflare terminates TLS at the edge. The Worker stores SDP payloads in D1 until the recipient reads them or until the 5-minute TTL expires.

### 21.5 Data at Rest
The application does not encrypt local files by default. The user can enable full-disk encryption on the operating system. The application does not store audio recordings unless the host starts a recording. The application does not store meeting transcripts.

### 21.6 Privacy Boundary
The host Worker sees this metadata:
- Room IDs.
- Peer public keys.
- Display names.
- Join approval times.
- SDP payloads (transient).
- Presence heartbeats.

The host Worker does not see media. The host Worker does not see audio content. The host Worker does not see screen shares. The host Worker does not see document content. The host Worker does not see soundboard files.

> [!IMPORTANT]
> The developer sees nothing. The developer does not host any part of the runtime stack.

### 21.7 Threat Model

| Threat | Mitigation |
| :--- | :--- |
| Eavesdropping on media | DTLS-SRTP encryption |
| Unauthorized room access | Host approval flow |
| Malicious soundboard file | File type validation, size limit, hash check |
| Malicious PDF file | Sandboxed rendering, pdf.js, JavaScript disabled |
| Denial of service on host Worker | Cloudflare rate limits (built-in) |
| SFU overload attack | Hard 15 Mbps cap |
| Spoofed presence heartbeat | Signed heartbeat with peer key |

Security protects data. Performance defines the user experience. The next section sets performance targets.

---

## 22. Performance Targets and Latency Budget

### 22.1 Performance Targets

| Metric | Target |
| :--- | :--- |
| Connection setup time | < 5 s |
| Audio latency (one hop) | < 75 ms |
| Audio latency (three hops) | < 150 ms |
| Audio packet loss tolerance | 5% with FEC |
| Screen share latency | < 200 ms |
| Document sync latency | < 500 ms |
| Profile photo load time | < 2 s |
| Soundboard file stream start | < 1 s |
| SFU failover time | < 2 s |
| Room join time (after approval) | < 5 s |
| Toast display latency | < 100 ms |
| Loading indicator appearance | < 100 ms |

### 22.2 Audio Quality Targets

| Metric | Target |
| :--- | :--- |
| Audio bitrate | 128 Kbps |
| Audio sample rate | 48 kHz. kHz means kilohertz. |
| Audio channels | Mono |
| Audio codec | Opus |
| Echo cancellation | Enabled |
| Noise suppression | Enabled |
| Automatic gain control | Enabled |

### 22.3 Screen Share Quality Targets

| Metric | Target |
| :--- | :--- |
| Resolution | 1920 x 1080 |
| Frame rate | 15 fps. fps means frames per second. |
| Bitrate | 2 Mbps |
| Codec | VP9 (fallback VP8) |
| Text clarity | High priority |

### 22.4 System Resource Targets

| Resource | Target |
| :--- | :--- |
| Central Processing Unit (CPU) usage (end-user) | < 10% |
| CPU usage (SFU peer) | < 25% |
| Memory usage | < 500 MB |
| Disk usage (cache) | < 500 MB |
| Network usage (end-user) | < 0.2 Mbps upload, < 15 Mbps download |

### 22.5 Connection Setup Budget

| Step | Budget |
| :--- | :--- |
| Bandwidth test | < 3 s |
| STUN discovery | < 1 s |
| ICE gathering (capped at 10 candidates) | < 3 s |
| SDP exchange through Worker | < 2 s |
| **Total** | **< 5 s** |

Performance targets define success. Licensing defines sustainability. The next section explains licensing.

---

## 23. Licensing and Monetization

### 23.1 Core Principle
Echo Room does not use a freemium model. Echo Room does not limit call length. Echo Room does not limit participant count. Echo Room does not limit core features. The core application is free forever.

### 23.2 Paid Features
The application offers these paid add-ons:

| Feature | Description | Enforcement |
| :--- | :--- | :--- |
| Cloud recording | Record the call locally, upload to cloud storage | Server-side API key check |
| AI transcription | Real-time transcription via external API | Server-side API key check |
| Meeting summaries | AI-generated summaries | Server-side API key check |
| Custom branding | Remove Echo Room logo, add company logo | Client-side license key check |
| Admin dashboard | Web dashboard for business management | Server-side license check |
| Priority support | Email and chat support | Manual verification |

### 23.3 License Enforcement
The application uses a static JSON file hosted on Lovable. The file is `license_config.json`. The file contains a list of licensed device public keys. The application fetches this file on startup. The application caches the file for 24 hours. The application validates the license locally. The application does not require a server.

### 23.4 B2B Model
Business to Business (B2B) means transactions between companies. Businesses pay for management features. Businesses do not pay for calls. Businesses pay for:
- Admin dashboard.
- Usage analytics.
- Compliance logging.
- Single Sign-On (SSO) integration (optional). SSO means Single Sign-On.
- Custom branding.

Businesses that do not pay can still use the free core features. The application does not block free use. The application does not degrade free use.

### 23.5 Consumer Add-Ons
Consumers pay for consumption-based features. Consumers pay for:
- Cloud recording storage.
- AI transcription minutes.
- Meeting summary credits.

Consumers do not pay for calls. Consumers do not pay for participant count.

### 23.6 Host Cost
The host pays Cloudflare directly. The free tier covers typical use. The developer never pays for host infrastructure. The host never pays the developer.

### 23.7 No Privacy Policy Requirement
The core application does not collect personal data. The core application does not need a privacy policy. The core application does not need a terms of service page. Business add-ons that collect data require a separate policy.

### 23.8 Developer Revenue
The developer earns revenue from paid add-ons and B2B licenses. The developer does not earn revenue from host infrastructure. The developer does not take a share of host Cloudflare bills.

Licensing keeps the project sustainable. Implementation turns the plan into software. The next section sets the roadmap.

---

## 24. Implementation Roadmap and Testing

### 24.1 Phase 1: Foundation (Weeks 1–4)

| Task | Deliverable |
| :--- | :--- |
| Set up Electron project | Working Electron skeleton |
| Implement Worker template and schema | Deployable Worker with D1 |
| Implement Wrangler setup wizard | Host can deploy their own stack |
| Implement signaling through Worker | Two peers exchange SDP |
| Implement WebRTC audio | Two peers hear each other |
| Implement ICE candidate cap | Connection under 5 seconds |
| Implement toast service | Notifications work |
| Implement basic UI | Participant list, mute button |

### 24.2 Phase 2: SFU Tree (Weeks 5–8)

| Task | Deliverable |
| :--- | :--- |
| Implement bandwidth test | Peer role assignment |
| Implement SFU forwarding | Three-peer audio call |
| Implement load balancing | SFU rebalancing works |
| Implement failover | SFU disconnect recovery |
| Implement tree depth limit | Three-hop maximum |
| Implement connection progress ring | Visual feedback for setup |

### 24.3 Phase 3: Features (Weeks 9–12)

| Task | Deliverable |
| :--- | :--- |
| Implement screen share | Selective subscription works |
| Implement profile photos | Photo exchange works |
| Implement soundboard | Data channel file transfer works |
| Implement document viewer | PDF render and sync works |
| Implement permissions | Host control panel works |
| Implement loading indicators | All async operations show feedback |

### 24.4 Phase 4: Hardening (Weeks 13–16)

| Task | Deliverable |
| :--- | :--- |
| Implement logging | Log files rotate correctly |
| Implement cache management | Cache limit enforced |
| Implement security checks | Threat model mitigated |
| Implement performance tests | Targets met |
| Implement packaging | Installers for Windows, macOS, Linux |

### 24.5 Testing Plan

| Test Type | Description |
| :--- | :--- |
| Unit tests | Test individual modules (SFU logic, manifest parsing) |
| Integration tests | Test two-peer and three-peer calls |
| Load tests | Test 100-peer and 500-peer rooms |
| Failover tests | Kill SFU peers during calls |
| Network tests | Simulate packet loss and latency |
| Timeout tests | Force Worker and STUN failures |
| Security tests | Attempt unauthorized room join |
| Toast tests | Verify publish-subscribe behavior across subsystems |
| Loading tests | Verify all async operations show feedback |
| Usability tests | Test with real podcasters and voice actors |

### 24.6 Success Criteria
Echo Room is ready for release when:
- A 10-person audio call works for 60 minutes without dropout.
- A 100-person audio call works with 3 screen shares.
- An SFU failover completes in under 2 seconds.
- Audio latency stays under 150 ms in a 500-person room.
- Connection setup completes in under 5 seconds on home networks.
- The soundboard transfers a 5 MB file in under 1 second.
- The document viewer syncs scrolls within 500 ms.
- Every async operation shows visible loading feedback.
- Every system message appears as a toast.
- The application uses less than 500 MB of memory.

The roadmap turns the plan into a product. The final section lists additional details that the main sections did not cover.

---

## 25. Additional Steel-Man Details

This section covers items that the main design did not explicitly address. These items are important for a complete system.

### 25.1 Echo Cancellation in the SFU Tree
An SFU mixes multiple audio streams. The mixer can create echo if the SFU also plays audio through speakers. The application must mute the SFU's local playback during mixing. The application must use headphones on SFU peers. The application shows a toast: *"Use headphones when you are an SFU peer."*

### 25.2 Audio Clock Synchronization
Different peers have different audio clocks. Clock drift causes buffer overruns or underruns. The application uses WebRTC's built-in clock synchronization. The application monitors buffer levels. The application adjusts the jitter buffer size dynamically. The adjustment range is 500 ms to 3000 ms.

### 25.3 Packet Loss Concealment
Opus includes Packet Loss Concealment (PLC). PLC means Packet Loss Concealment. The application enables PLC. The application uses in-band FEC. The application tolerates 5% packet loss without audible artifacts. The application logs packet loss statistics.

### 25.4 NAT Types
Some NAT devices are symmetric. Symmetric NAT blocks hole punching. Standard WebRTC can fail on symmetric NAT. The application detects symmetric NAT through ICE failure. The application shows a toast: *"Your network blocks direct connections. Echo Room cannot work on this network."* The application recommends a different network. The application does not fall back to TURN.

### 25.5 Firewall Rules
Corporate firewalls often block UDP. WebRTC uses UDP. The application cannot work through a UDP-blocking firewall. The application detects UDP blockage through STUN timeout. The application shows a toast: *"Your firewall blocks UDP. Echo Room requires UDP."*

### 25.6 Room Discovery Privacy
Room IDs are random. The Worker stores room IDs in D1. The host shares the URL only with intended guests. The Worker does not publish room IDs to a public list. An attacker must know the URL to attempt a join. The host must approve every join.

### 25.7 Soundboard File Validation
The application validates soundboard files before adding them. The application checks the file extension. The application checks the file size (max 10 MB). The application checks the file header. The application rejects files that fail validation. The application shows a toast with the reason.

### 25.8 PDF Security
The application renders PDFs in a sandboxed iframe. The application disables JavaScript in PDFs. The application disables external links. The application limits PDF size to 50 MB. The application does not execute embedded scripts.

### 25.9 Document Cache Limits
The document viewer caches rendered pages. The application limits the cache to 20 pages. The application pre-renders the next page. The application does not pre-render all pages. This design keeps memory usage low.

### 25.10 SFU CPU Usage
An SFU peer mixes audio for up to 117 children. Mixing 117 streams is CPU-intensive. The application uses a WebAssembly audio mixer. WebAssembly is a binary instruction format for web applications. The mixer processes 10 ms frames. The mixer uses Single Instruction Multiple Data (SIMD) instructions when available. SIMD means Single Instruction Multiple Data. The application targets under 25% CPU usage on a modern laptop.

### 25.11 Bandwidth Test Accuracy
The bandwidth test uses a temporary UDP echo server. The server is a public service. The server may be unavailable. The application falls back to a conservative estimate. The conservative estimate is 10 Mbps. The application marks the peer as an end-user if the test fails. The application shows a toast: *"Connection test failed. You join as a listener."*

### 25.12 Room State Persistence
The application does not persist room state across restarts. If the application crashes, the user must rejoin. The application saves the last room topic in the config file. The application offers a "Rejoin Last Room" button on the home screen.

### 25.13 Accessibility
The application supports keyboard navigation. The application supports screen readers. The application provides high-contrast mode. The application provides adjustable font sizes. The application provides captions for the soundboard (sound names). All toasts have accessible roles. All loading indicators have accessible labels.

### 25.14 Internationalization
Version 2.1 supports English only. The application uses a string table for future translations. The application does not hard-code UI strings. The application supports right-to-left layouts in future versions.

### 25.15 Auto-Update
The application uses Electron's auto-updater. The application checks for updates on startup. The application downloads updates in the background. The application shows a toast when an update is ready. The application prompts the user to restart. The application does not force updates during a call.

### 25.16 Crash Reporting
The application does not send crash reports automatically. The application writes crash logs locally. The user can choose to share crash logs manually. The application does not include personal data in crash logs.

### 25.17 Offline Mode
The application does not support offline mode. Echo Room requires an internet connection. The application shows a toast when offline: *"No internet connection."*

### 25.18 Multi-Device Support
A user can join the same room from two devices. The application treats each device as a separate peer. The application shows both devices in the participant list. The application does not merge identities.

### 25.19 Room Recording
The application supports local recording. The host can start a local recording. The recording captures the mixed audio. The recording does not capture screen share video in version 2.1. The recording saves to `~/EchoRoom/Recordings/`. The application shows a toast to all participants: *"Recording started by host."* The application shows a persistent red dot in the UI while recording.

### 25.20 Soundboard Latency
Soundboard sounds play locally. The trigger message travels over the data channel. The data channel latency is under 100 ms. The sound playback starts within 150 ms of the trigger. This latency is acceptable for sound effects.

### 25.21 Worker Rate Limiting
The host Worker uses Cloudflare rate limiting. The Worker limits each IP address to 60 requests per minute. The Worker limits each peer key to 30 requests per minute. The Worker returns a `429` status when a limit is exceeded. The application shows a toast: *"Too many requests. Wait a moment."*

### 25.22 D1 Schema Migration
When the application updates, the Worker may need schema changes. The Worker handles migrations with idempotent Structured Query Language (SQL). SQL means Structured Query Language. The Worker checks the current schema version in a `meta` table. The Worker applies migrations in order. The application shows a toast during the migration: *"Updating server schema…"*

### 25.23 Testing with Real Users
The development team must test Echo Room with real podcasters and voice actors. The team must record usability issues. The team must fix critical issues before release. The team must prioritize audio quality over feature count.

### 25.24 Documentation for Users
The application includes a help guide. The help guide covers:
- How to set up a host server.
- How to create a room.
- How to join a room.
- How to use the soundboard.
- How to share a screen.
- How to use the document viewer.
- How to troubleshoot connection problems.

The help guide uses simple language. The help guide includes screenshots.

### 25.25 Legal Considerations
Echo Room does not collect personal data. Echo Room does not require a privacy policy for core use. Echo Room does not require a terms of service for core use. Business add-ons may require separate legal documents. The development team must consult a lawyer before offering business add-ons.

### 25.26 Future Features
The following features are out of scope for version 2.1 but are planned for future versions:
- Host migration.
- Camera video.
- Public rooms.
- Mobile applications.
- Web browser support.
- End-to-end encryption for data at rest.
- Multi-language support.
- Advanced audio effects (reverb, compression).
- Optional TURN fallback via self-hosted coturn.
- Optional central registry for room discovery.

### 25.27 Glossary of Toast Messages
This section lists the toast messages the application uses. The list is not final. The list helps keep wording consistent.

| Trigger | Type | Title |
| :--- | :--- | :--- |
| Connection starts | `info` | Connecting… |
| Connection succeeds | `success` | Connected |
| Connection fails | `error` | Connection failed |
| Host approval pending | `info` | Waiting for host |
| Host approves | `success` | Joined room |
| Host denies | `warning` | Join denied |
| Room ends | `info` | Room ended by host |
| Bandwidth test completes | `info` | Your connection: N Mbps |
| SFU role assigned | `info` | You are forwarding audio |
| SFU failover begins | `warning` | Reconnecting to a new forwarder |
| Sound added | `success` | Sound added |
| Sound saved | `success` | Saved to My Soundboard |
| Soundboard permission changes | `info` | Soundboard permissions changed |
| Screen share starts | `info` | Sharing your screen |
| Screen share stops | `info` | Screen share stopped |
| Document fails to load | `warning` | Google Docs cannot be embedded |
| Worker unavailable | `error` | Server unavailable |
| Network changes | `warning` | Network changed. Reconnecting… |
| Update ready | `info` | Update available |

---

**End of document.**