# DJI Mini 3 Pro Internet Control with Open-Source Tools

This project enables remote control of a DJI Mini 3 Pro over the internet using open-source software. It mirrors the DJI Fly app (via the RC-N1 controller) with scrcpy, streams the feed using WebRTC, and allows control via a JavaScript/TypeScript web interface. No proprietary tools like TeamViewer are required.

## Overview

- Goal: Control the DJI Mini 3 Pro from anywhere with an internet connection.
- Approach: Mirror the smartphone running DJI Fly locally with scrcpy, relay the feed over the internet using webrtc-streamer, and send commands back using a custom web app.
- Tools:
  - scrcpy: Open-source screen mirroring for Android.
  - webrtc-streamer: Streams the mirrored screen over the internet.
  - Ngrok: Exposes the local stream to a public URL.
  - Node.js with JavaScript/TypeScript: Custom control interface.

## Prerequisites

- Hardware:
  - DJI Mini 3 Pro drone.
  - DJI RC-N1 controller.
  - Android smartphone (iOS not natively supported by scrcpy).
  - Local computer (e.g., laptop or Raspberry Pi) near the drone with internet (Wi-Fi/4G).
  - Remote device (PC, tablet, etc.) with a browser.
- Software:
  - DJI Fly app (https://www.dji.com/downloads/djiapp/dji-fly) on the smartphone.
  - scrcpy (https://github.com/Genymobile/scrcpy) on the local computer.
  - webrtc-streamer (https://github.com/mpromonet/webrtc-streamer) on the local computer.
  - Ngrok (https://ngrok.com/, free tier) for internet tunneling.
  - Node.js (https://nodejs.org/) and npm for the web app.
  - ADB (https://developer.android.com/tools/adb) for phone connectivity.

## Setup Instructions

### 1. Prepare the Drone and Smartphone
1. Power on the DJI Mini 3 Pro and RC-N1 controller.
2. Connect the smartphone to the RC-N1 via USB (USB-C or Lightning).
3. Launch DJI Fly and confirm the drone connects.
4. Enable USB Debugging on the phone: Settings > About Phone > Tap Build Number 7 times > Developer Options > USB Debugging.

### 2. Install Tools on the Local Computer
1. Install ADB:
   - Linux: sudo apt install adb
   - Windows/Mac: Download from Android SDK or via scrcpy installer.
2. Install scrcpy:
   - Linux: sudo apt install scrcpy
   - Windows/Mac: Download from https://github.com/Genymobile/scrcpy/releases.
3. Install webrtc-streamer:
   - Docker (easiest): docker run -p 8000:8000 mpromonet/webrtc-streamer
   - Or build from source: Clone https://github.com/mpromonet/webrtc-streamer, run npm install && npm start.
4. Install Ngrok:
   - Download from https://ngrok.com/download, unzip, and authenticate: ./ngrok authtoken <your-token>.
5. Install Node.js:
   - Linux: sudo apt install nodejs npm
   - Windows/Mac: Download from https://nodejs.org/.

### 3. Mirror the Smartphone Locally
1. Connect the smartphone to the local computer via USB.
2. Run scrcpy: scrcpy
3. The DJI Fly app should appear on the local computer’s screen. Test by clicking the Takeoff button.

### 4. Stream Over the Internet
1. Start webrtc-streamer to capture the screen: webrtc-streamer -u "screen://0" -w 8000
   - Adjust "screen://0" if using multiple displays.
2. Expose the stream with Ngrok: ./ngrok http 8000
   - Copy the public URL (e.g., https://abc123.ngrok.io).

### 5. Set Up the Web Control Interface
1. Create a project folder: mkdir drone-control && cd drone-control; npm init -y; npm install express robotjs
2. Add the server code (save as server.js):
   - const express = require('express');
   - const robot = require('robotjs');
   - const app = express();
   - app.use(express.static('public'));
   - app.get('/takeoff', (req, res) => { robot.moveMouse(500, 800); robot.mouseClick(); res.send('Takeoff sent'); });
   - app.listen(3000, () => console.log('Server on port 3000'));
3. Create a public folder with index.html:
   - <!DOCTYPE html>
   - <html><body>
   - <video id="video" autoplay style="width: 100%;"></video>
   - <button onclick="sendCommand('takeoff')">Takeoff</button>
   - <script>
   - const video = document.getElementById('video');
   - const peer = new RTCPeerConnection();
   - fetch('http://localhost:8000/offer').then(res => res.json()).then(offer => peer.setRemoteDescription(offer)).then(() => peer.createAnswer()).then(answer => peer.setLocalDescription(answer)).then(() => fetch('http://localhost:8000/answer', { method: 'POST', body: JSON.stringify(peer.localDescription) }));
   - peer.ontrack = e => video.srcObject = e.streams[0];
   - function sendCommand(cmd) { fetch(`http://localhost:3000/${cmd}`); }
   - </script>
   - </body></html>
4. Run the server: node server.js
5. Tunnel the web app with Ngrok (in a second terminal): ./ngrok http 3000

### 6. Control Remotely
1. On your remote device, open a browser.
2. Visit the Ngrok URL for the web app (e.g., https://xyz789.ngrok.io).
3. View the live DJI Fly feed and click the Takeoff button to test.

## Usage
- Video Feed: The drone’s camera streams via WebRTC to your browser.
- Control: Click buttons in the web app to trigger actions (e.g., Takeoff). Customize server.js for more commands (e.g., Land, Move).
- Coordinates: Adjust robotjs mouse positions in server.js to match your DJI Fly UI layout.

## Notes
- Latency: Expect 200-500 ms delay over 4G. Use slow inputs or automate commands.
- Android Only: scrcpy works best with Android. iOS requires alternative tools (e.g., ios-webkit-debug-proxy).
- Legal: Beyond-line-of-sight (BLOS) flight may violate regulations (e.g., FAA VLOS rules). Use a spotter or test in a legal area.
- Power-On: Someone must manually start the drone initially.

## Troubleshooting
- scrcpy Fails: Ensure USB Debugging is on and ADB recognizes the device (adb devices).
- WebRTC Black Screen: Check webrtc-streamer logs and confirm screen capture works.
- Ngrok Errors: Verify your auth token and internet connection.

## License
This project uses open-source tools under their respective licenses:
- scrcpy: Apache 2.0
- webrtc-streamer: MIT
- Ngrok: Free tier proprietary (open-source alternatives like localtunnel exist)

Contributions welcome!
