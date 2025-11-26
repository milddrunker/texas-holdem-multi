# Texas Hold'em Multi

Multiplayer Texas Hold'em (Node.js + Express + Socket.IO). Supports multiplayer online play, automatic stage progression, no-limit betting rules, host controls, showdown allocation, and profit/loss display.

## Features
- Multiplayer real-time play: Socket.IO bidirectional communication with state-specific broadcasts.
- Automated flow: stages and actions advance automatically; host only controls "Start Next Hand" and "Reset Room".
- Ready-up mechanism: after the host starts the next hand, all players become "Not ready"; the system deals cards and sets blinds (10/20) only when all players click "Ready".
- No-limit betting rules:
  - Bet is allowed only when no bet exists in the current round; amount ≥ minimum bet (BB).
  - Raise must reach current max bet + last raise increment (min raise increment updates as raises occur).
  - Call makes up to the current max bet; Check is allowed when even.
  - Server-side authoritative validation; invalid actions trigger client alerts.
- Current actor highlight: UI highlights the player whose turn it is.
- Showdown and distribution: automatically evaluates hands, splits the pot (handles ties), and updates each player's profit/loss (positive/negative).
- Simple frontend: plain HTML/CSS/JS, no framework dependencies.

## Tech Stack & Structure
- Server: `server.js`
  - Express serves `public/`, Socket.IO events and state management.
  - Main events: `join`, `setReady`, `startNextHand`, `bet`, `raise`, `call`, `check`, `fold`, `resetGame`, `disconnect`.
  - Broadcast fields: stage, community cards, dealer/SB/BB seats, current max bet, minimum bet and minimum raise increment, current actor, pot list, player ledger and per-round bet info.
- Client: `public/index.html`
  - Plain page and interactions; renders the table, player status, button availability, and error alerts.

## Directory Structure
```
.
├─ public/
│  └─ index.html         # Client page (vanilla HTML/JS/CSS)
├─ server.js             # Node + Express + Socket.IO server
└─ README.md             # Project docs
```

## Local Run
1. Install Node.js (LTS recommended).
2. In the project root, run:
   - `node server.js`
3. Open in browser:
   - `http://localhost:3000/`
4. With two devices open: host clicks "Start Next Hand", all players click "Ready"; the game starts automatically. Follow UI prompts to bet/raise/call/check.

## Public Deployment (single domain)
Same-origin deployment recommended (static page and Socket.IO under the same domain).

1) Domain & server
- Purchase a domain and a cloud host (public IP).
- Add an `A` record in DNS pointing to the host IP.

2) Deploy Node service
- Install Node.js and an optional process manager (e.g., `pm2`).
- Upload code and start: `pm2 start server.js --name texas-holdem` or `node server.js`.

3) Nginx + HTTPS + WebSocket
```
server {
  listen 80;
  server_name yourdomain.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name yourdomain.com;
  ssl_certificate     /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

  location /socket.io/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
  }
}
```

4) Verification
- Access `https://yourdomain.com` from different networks; after both sides join the room, state and betting interactions should sync in real time.

## Usage
- Enter a nickname and click `Join Room`.
- Host clicks `Start Next Hand`; all players become `Not ready`.
- When all players click `Ready`, the system deals cards and sets blinds.
- Follow UI prompts to bet/raise/call/check; invalid actions pop up alerts.
- `Fold` leaves the current hand; `Reset Game` is host-only.

## FAQ
- Cannot connect: check Nginx WebSocket forwarding (`Upgrade`/`Connection: upgrade`), open inbound 80/443, and certificate validity.
- Cross-origin deployment: same-origin is recommended; if separated, enable Socket.IO CORS on the server and use an explicit backend URL on the client.

## Notes
- This demo focuses on rules/flow and real-time interaction; side-pot distribution in all-in scenarios can be extended as needed.
- Simple structure, easy to extend (kick players, room list, more UI, and test cases).
