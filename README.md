# Audio Conferencing Tool

A Java-based three-way audio conferencing application that streams real-time audio between a server and two clients over UDP.

## Overview

This project implements a 3-party audio conference call using the Java Sound API for audio capture/playback and UDP datagrams for low-latency audio transmission. Each session involves one **Server** and two **Users** (User1 and User2). When both users connect, the server bridges audio between all three parties so that every participant can hear the others in real time. Audio from each participant is also saved locally as a WAV file when the session ends.

## Architecture

```
User1  <---UDP--->  Server  <---UDP--->  User2
  |_______________________UDP_________________|
              (peer-to-peer audio)
```

The server acts as a signalling relay: it accepts connection requests from both users, exchanges their IP addresses so they can stream audio directly to each other, and also streams its own microphone audio to both.

### UDP Port Mapping

| Participant | Receive Ports | Send-to Ports         |
|-------------|---------------|-----------------------|
| Server      | 9991, 9992    | 9993 (User1), 9995 (User2) |
| User1       | 9993, 9994    | 9992 (Server), 9996 (User2) |
| User2       | 9995, 9996    | 9991 (Server), 9994 (User1) |

### Audio Format

| Property      | Value          |
|---------------|----------------|
| Sample Rate   | 12,000 Hz      |
| Sample Size   | 16-bit         |
| Channels      | 2 (Stereo)     |
| Encoding      | PCM Signed     |
| Byte Order    | Little-Endian  |
| Buffer Size   | 10,000 bytes   |

## Project Structure

```
src/
├── Server.java   – Conference server: accepts connections, relays/records audio
├── User1.java    – Client 1: connects to server, streams audio to/from server and User2
├── User2.java    – Client 2: connects to server, streams audio to/from server and User1
└── Main.java     – Placeholder main class
```

## Prerequisites

- Java Development Kit (JDK) 8 or higher
- A working microphone and speakers/headphones on each machine
- All three machines must be on the same local network (or have network access to each other)
- Firewall rules must allow UDP traffic on ports **9991–9996**

## Configuration

Before running, update the hardcoded values in each source file:

### Server.java
```java
// Line 58 – directory where recorded WAV files are saved
filesavelocation = "C:\\path\\to\\your\\save\\directory";

// Lines 146, 177 – replace with the actual IP address of User1/User2
// (or remove the hardcoded override and use packet.getAddress().getHostAddress())
conference_two   = "192.168.x.x";  // User1's IP
conference_three = "192.168.x.x";  // User2's IP
```

### User1.java
```java
// Line 50 – directory where recorded WAV files are saved
filesavelocation = "C:\\path\\to\\your\\save\\directory\\1";

// Line 51 – IP address of the machine running Server
serverip = "192.168.x.x";
```

### User2.java
```java
// Line 51 – directory where recorded WAV files are saved
filesavelocation = "C:\\path\\to\\your\\save\\directory\\2";

// Line 52 – IP address of the machine running Server
serverip = "192.168.x.x";
```

## Building

Compile all source files from the `src/` directory:

```bash
cd src
javac Server.java User1.java User2.java Main.java
```

## Running

Launch each class on its respective machine **in this order**:

1. **Start the server first:**
   ```bash
   java Server
   ```
   A window with a **Start** button appears. Click **Start** to begin listening for connections.

2. **Start User1** (on a second machine or a second terminal on the same machine):
   ```bash
   java User1
   ```
   Click **Start**. User1 sends a `"Connect"` message to the server and waits for the session to begin.

3. **Start User2** (on a third machine or terminal):
   ```bash
   java User2
   ```
   Click **Start**. User2 sends a `"Connect"` message to the server. Once both users have connected, the server exchanges their IP addresses and the audio conference begins automatically.

## Using the Application

- **Start** – Initiates the connection and opens the audio stream.
- **Disconnect** – Gracefully ends the session. A `"Disconnect"` packet is sent to all peers, audio streams are flushed, and WAV recordings are saved to `filesavelocation`.

## Recorded Audio Files

Each participant saves WAV recordings of the session locally upon disconnection:

| File        | Contents                         | Saved by |
|-------------|----------------------------------|----------|
| audio1.wav  | Audio received from User1        | Server   |
| audio2.wav  | Audio received from User2        | Server   |
| audio3.wav  | Server's own microphone audio    | Server   |
| audio4.wav  | Audio received from Server       | User1    |
| audio5.wav  | Audio received from User2        | User1    |
| audio6.wav  | User1's own microphone audio     | User1    |
| audio7.wav  | Audio received from Server       | User2    |
| audio8.wav  | Audio received from User1        | User2    |
| audio9.wav  | User2's own microphone audio     | User2    |

## Known Limitations

- IP addresses and file save paths are hardcoded and must be updated before compiling.
- The conference is limited to exactly 3 participants (1 server + 2 users).
- No encryption or authentication is applied to audio streams.
- UDP delivery is best-effort; packet loss may cause audio dropouts on lossy networks.
