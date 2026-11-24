[Türkçe](README.md)

# Serverless P2P Encrypted Video Call (WebRTC)

A minimalist WebRTC video calling app that provides **direct end-to-end encrypted (P2P)** audio and video between two devices, with no third-party media servers (SFU/MCU), databases, or cloud recording systems.

---

## Key Features

* **100% Direct P2P Media Streaming:** Audio and video packets are encrypted with DTLS/SRTP and flow directly between the two devices; they never pass through a central server.
* **Zero-Logging:** The signaling server (`server.js`) only relays the SDP and ICE candidates needed for the connection handshake, in memory and in real time. Nothing is written to disk, a database, or the browser cache.
* **Zero-Install Client:** The other party does not need to install any app, extension, or VPN. It works directly in iOS Safari, Android Chrome, and desktop browsers via the generated HTTPS link.
* **Smart UI & Mobile Support:**
  * Uncropped (`object-fit: contain`) full-screen remote video.
  * Mirrored Picture-in-Picture (PiP) local camera preview in the bottom-right corner.
  * Controls auto-hide after 3 seconds of inactivity during a call.
  * One-tap front/rear camera switching (`facingMode`) on mobile devices, plus echo cancellation and noise suppression.

---

## Requirements (Host Computer Only)

* [Node.js](https://nodejs.org/) (v18 or higher)
* [Cloudflare Tunnel (`cloudflared`)](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)

---

## Installation and Usage

### 1. Download the Project and Install Dependencies

Open a terminal in the project directory and install the dependencies:

```bash
npm install
```

### 2. Start the Signaling Server

In the first terminal window, start the local HTTP and WebSocket signaling server:

```bash
node server.js
```

Output: `Sunucu 3000 portunda hazir.` (Server is ready on port 3000.)

### 3. Create a Secure HTTPS Tunnel

In a second terminal window, start Cloudflare Tunnel to expose the local server to the internet with an SSL/HTTPS certificate:

```bash
cloudflared tunnel --url http://localhost:3000
```

### 4. Start the Call

1. Copy the `https://<random-subdomain>.trycloudflare.com` address generated in the terminal output.
2. Open the link in your own browser and also send it to the person you want to call. (The client detects the WebSocket address dynamically from the current URL; you don't need to edit any code.)
3. When both sides show the "Ready to connect" status, start the P2P call using the **Call** and **Accept Incoming Call** buttons.

---

## Project Structure

```plaintext
├── index.html      # Single-file WebRTC client UI and signaling logic
├── server.js       # Minimalist HTTP + WebSocket signaling server with 2-person room capacity
├── package.json    # Node.js project dependencies (ws)
├── package-lock.json
├── .gitignore      # node_modules exclusion rule
└── README.md       # Project documentation
```

---

## Network and Firewall Notes

* **CGNAT / Corporate Networks:** Direct P2P connections are established over UDP. Corporate, school, or public Wi-Fi networks with strict firewalls (Symmetric NAT) may block UDP traffic; in these cases, using a standard home network or a mobile hotspot is recommended.
* **Hotspot NAT Loopback:** If you share a phone's hotspot and try to establish a P2P connection between that phone and a computer connected to the same hotspot, you may run into a local network loop (NAT hairpinning). Tests should be done between two devices on different networks.

---

## Security Notes

* **Be careful when sharing the link:** Send the `trycloudflare.com` address only to the person you will call. There is no password on the room; anyone who knows the link and joins while the room is empty can take part in the call. The address changes on every run, but do not paste it into public places such as the README, commit messages, or issues.
* **Signaling server:** Media is P2P and encrypted with DTLS/SRTP; however, the connection setup messages (SDP/ICE) pass through your server and Cloudflare Tunnel. "Zero-logging" applies only to this server's own code behavior.
