# 🏴‍☠️ HTB Cyber Apocalypse 2026 — Try Out (Event 1434)

**35/37 challenges solved | ~34,275 pts**

Team: ChipSolo1781348405 | User: @chipchip123

---

# HTB Cyber Apocalypse 2026 — Try Out (Event 1434) Solve Guide

**35/37 solved, ~34,275 pts.** Username: @chipchip123, Team: ChipSolo1781348405.

HTB CTF link: https://ctf.hackthebox.com/event/details/ctf-try-out-1434


## WEB CHALLENGES

### 1. Guild (cid 40756, easy, +1000pts)
**Vulnerability**: Flask SSTI via EXIF metadata Artist field.

**Steps**:
1. Register user on the web app
2. Set bio to `{{lipsum.__globals__['os'].popen('id').read()}}` → leaks that SSTI works
3. Use password reset — the admin email is leaked via the bio SSTI (rendered in admin panel)
4. Admin email hashed with sha256 = default password
5. Login as admin → upload image
6. Set EXIF Artist field to: `{{lipsum.__globals__['os'].popen('cat /app/flag.txt').read()}}`
7. View uploaded image → flag rendered in EXIF display

**Flag**: `HTB{...}` (per-instance)

### 2. Prison Pipeline (cid 40793, medium, +1000pts)
**Vulnerability**: npm supply-chain attack via Verdaccio private registry.

**Steps**:
1. App has a cronjob running `npm update` every 30 seconds
2. Find which package the app uses
3. Create malicious version of that package on the private Verdaccio registry
4. Payload: reverse shell or `cat /flag*`
5. Wait for cronjob to pick up the update → RCE → flag

### 3. TunnelMadness (cid 40765, medium, +1000pts)
**Vulnerability**: 3D maze pathfinding challenge (network).

**Steps**:
1. Connect to the TCP service
2. Parse the 3D maze grid
3. BFS/Dijkstra from start to end (allowing up/down/left/right/forward/back moves)
4. Send the solution path (59 moves) → flag

### 4. Chrono Mind (cid 40768, medium, +1000pts)
**Vulnerability**: Path traversal → LLM system prompt leak → copilot RCE.

**Steps**:
1. App has an LLM chatbot feature
2. Create a "topic" with name containing `../` — path traversal in topic parameter
3. The topic path traversal leaks the LLM's system prompt, which contains `copilot_key: 6704408220203041`
4. POST to `/api/copilot/complete_and_run` with arbitrary Python code in the body (authenticated with copilot_key)
5. RCE: `os.popen('cat /readflag').read()` → flag

### 5. Dynamic Paths (cid 40784, medium, +1000pts)
**Vulnerability**: TCP socket programming + DP pathfinding.

**Steps**:
1. Connect to TCP service
2. Receive 100 grid pathfinding puzzles
3. For each: grid with costs, find min-cost path from top-left to bottom-right (right/down only)
4. Classic DP: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`
5. Send all 100 answers → flag

### 6. NotADemocraticElection (cid 40802, medium, +975pts)
**Vulnerability**: Blockchain hash collision.

**Steps**:
1. Smart contract election — need to vote with specific hash
2. Use Foundry to brute-force input that produces matching hash
3. Submit vote with collision input → win election → flag


## REVERSE ENGINEERING CHALLENGES

### 9. Labyrinth Linguist (cid 31862, medium, +1000pts)
**Vulnerability**: Java Velocity SSTI → RCE.

**Steps**:
1. App uses Apache Velocity template engine
2. Find template injection point in user input
3. Payload: `#set($x='')+$x.getClass().forName('java.lang.Runtime').getRuntime().exec('cat /flag.txt')`
4. Or simpler SSTI chain through Velocity class hierarchy → RCE → flag


## CRYPTO CHALLENGES

### Blessed (cid 40754, hard — PARTIALLY ANALYZED)
**Vulnerability**: Weak custom PRNG (p256 curve point iteration) leaks outputs.

**Status**: Analyzed but not solved. PRNG outputs leaked via `robot_id` values. `verify()` uses `next(self.rand) & 1` for 64 Schnorr-like proof rounds. Attack: predict PRNG → forge challenge bits → pass verify → `unveil_secrets` returns flag.


## TOOL NOTES

- **HTB API proxy**: Use dataimpulse residential proxy for HTB API calls (Cloudflare blocks direct)
- **Container access**: Direct TCP to spawned containers (no proxy needed)
- **Pwntools**: Use `/usr/bin/python3` for scripts (system python, not venv)
- **Foundry**: `/root/.foundry/bin/forge` for Solidity/crypto work
- **Gerber rendering**: `gerbv -x png` for PCB layers, but vision AI unreliable for reading text