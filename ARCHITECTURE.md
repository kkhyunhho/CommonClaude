# Lab architecture — Levels, Composition, and Phases

The target system the SDLClaude projects build into: a self-driving
electrochemistry lab. Three distinct concepts; keep them separate.
(Counts are the current plan; TBD items will firm up.)

## Level (L) — control-code depth of a device/subsystem

How deep the control stack is built for a given device. Built **per need**
— not every device reaches L1 or L2.

| Level | What | Example |
|-------|------|---------|
| **L0** | Hardware-driving **driver** library | `sy01b`, `entris_ii`, `picus2`, `mks_motor` |
| **L1** | **Server** — expose the driver over HTTP | FastAPI `/v1` (sy01b-server) |
| **L2** | **ESP touchscreen / physical UI** | ESP32-S3-BOX-3 LVGL touch UI |

L0 is the baseline every device ships. L1/L2 are added only where remote /
web / on-device control is actually wanted.

## Composition — how units combine (a SEPARATE axis from Level)

Build small, compose up; each tier is itself a "device-shaped unit." This
is orthogonal to Level — it is about wiring multiple things together, not
about control depth.

```
device instance   one physical unit (its own USB port)
      | compose
cell               heterogeneous instances for one co-located function
                   (e.g. 3 motors + 1 pump); owns its real-time safety
                   interlocks IN-PROCESS
      | compose
Phase-system       the cells/devices on one NUC, composed into a working
                   stage (see Phase below)
      | network
whole SDL          Phases coordinated across NUCs (networked substrate)
```

- **One USB port per device unit** (expanded via USB hubs). No RS-485 /
  CAN multidrop or shared-bus addressing — chosen for debugging simplicity
  (each unit isolatable on its own port). Trade-off: more adapters/ports.
  Units are identified by USB identity (`VID:PID`, or `VID:PID:SERIAL` when
  several share a `VID:PID`).
- **Real-time safety stays in-process within a cell** (e.g. paired-motor
  interlock: one motor faults → stop the whole gantry now). Never route a
  safety interlock over the network.

### Cell boundary rule — "must they be coordinated to move?"

Put devices in the **same cell** (one process, in-process control) when any
holds:

- they must move **together / in lockstep** (e.g. paired Z motors),
- moving them **at the same time risks collision or interference** so a
  controller must coordinate/interlock them in real time,
- a **fault in one must instantly stop another**.

Put them in **separate cells** (network-coordinated) when they are
**independent — moving simultaneously is always safe** and loose
coordination over the network suffices.

Adding a device later: if it is coordination-coupled to an existing cell,
**the cell grows** (same server, same port); if it is independent, it is a
**new cell** (new server, new port). The cell boundary = the real-time
coordination/safety boundary, nothing else.

## Phase — SDL hardware stage

Which hardware, on which NUC. The whole SDL is built in two phases, each on
its **own NUC**; cross-phase coordination is networked.

### Phase 1 (NUC A) — actual bench layout

| Cell | Devices | `/v1` port |
|------|---------|-----------|
| cell1 | XZ gantry (3 MKS motors) + syringe pump | 17054 |
| cell2 | XZ gantry (3 MKS motors) + syringe pump | 17056 |
| cell3 | XZ gantry (3 MKS motors) + syringe pump | 17058 |
| cell4 | linear motor (Y axis) + **balance** | 17060 |
| (Phase-1 orchestrator) | composes cell1–4 (built when ≥2 cells run) | 17062 |

Totals: 9 MKS motors + 3 pumps + 1 linear motor + **1 balance**. There is
**one balance for the whole Phase** — it is mounted on cell4's Y-axis linear
motor and shuttles under cell1–3 to weigh each one's dispense, so the
balance belongs to **cell4**, not the dispensing cells. cell1–3 are
dispense-only (XZ + pump). Weighing a dispense is therefore a **cross-cell**
interaction (cell4 positions its balance under the target cell) coordinated
by the Phase orchestrator.

The **SyringeLiquidHandler** repo is the reference implementation for the
dispensing-cell shape (its balance code seeds cell4's). A robot arm is
deferred — when added it is its own cell (or Phase-level shared); an arm
reaching **into** a cell's workspace is cross-cell motion whose collision
avoidance is a dedicated design point (e.g. lock/halt the target cell
during transfer).

### Port assignment rule

- A port is allocated **per running `/v1` server** (per cell, plus the
  orchestrator). Ports are **per-NUC**: a different NUC (different IP) may
  reuse the same numbers — uniqueness is only required within one host.
- This bench: **even numbers from 17054** (17040–17052 are already taken by
  other containers on the shared NUC). cell1=17054, cell2=17056, …, +2 per
  cell; the Phase orchestrator takes the next even (17062).
- Before opening a port, confirm it is free on the host (`sudo ss -ltnp` /
  `docker ps`); record the assignment here so the team does not collide.

### Phase 2 (NUC B)

1 MKS motor, 1 syringe pump, 1 robot arm, plus 3–4 further devices (TBD).

### Robot arms (external / senior-owned)

Two arms, added later by a different owner: arm #1 is used **before Phase 1
starts**, arm #2 **at the end of Phase 2**. Their driver code follows a
different coding style. **Mitigation: wrap each arm behind a thin adapter
that satisfies the standard cell / `/v1` interface.** The foreign style
stays isolated behind the adapter; conversion happens only at that
boundary, so neither side has to adopt the other's style.

## Coordination substrate — DECIDED: recursive HTTP `/v1`

- **Within a cell** → in-process Python calls (real-time, safe). No network.
- **Across cells / NUCs** → **HTTP `/v1`, applied recursively.** Each cell
  runs one FastAPI `/v1` server (one port). A higher orchestrator is just an
  HTTP **client** of those cell servers, and may itself expose its own `/v1`
  server to the layer above — the same contract repeats at every tier.

Why HTTP over ROS 2 / MQTT (for now): the `/v1` servers already exist and
the web + ESP32 are already HTTP clients; the standard already wraps foreign
robot arms behind a `/v1` adapter; cross-cell work is mostly **sequential
orchestration** (dispense → transfer → weigh), not hard-real-time (that
stays in-process per cell); and HTTP is far simpler to learn, debug (curl /
OpenAPI), and test. **ROS 2 is the fallback** to revisit *only* when a
concrete need appears — camera video streaming, rich arm action
feedback/cancel, or tight multi-cell motion sync — and even then only for
that part; the cell edge stays HTTP.

Ports appear **once per running `/v1` server** (i.e. per cell, and per
orchestrator that exposes one). A port is a *server's network address on a
NUC*, unrelated to the device's USB/serial port. Assign distinct ports per
server on a shared NUC; record them in a port table.

```
                      ┌─────────────────────────── whole SDL ───────────────┐
   client (browser/   │  SDL orchestrator  (optional /v1 server, own port)   │
   ESP32/CLI) ──HTTP──▶        │ HTTP client of each Phase                    │
                      │        ▼                                             │
                      │  Phase-1 orchestrator (NUC A, /v1 server, own port)  │
                      │        │ HTTP client of each cell                    │
                      │   ┌────┼───────────────┬──────────────┐             │
                      │   ▼    ▼               ▼              ▼              │
                      │ cell  cell           cell           cell   ← each:   │
                      │ :17054 :17056         :17058         …      one /v1  │
                      └───┼──────────────────────────────────────── server ─┘
                          │ in-process Python (NO network, real-time safety)
                     ┌────┼────┬─────────┐
                     ▼    ▼    ▼         ▼
                  pump  balance  motor … (L0 drivers)  ── USB/serial ─▶ hardware
```

Build order this implies: a single cell + its `/v1` server first (done for
SyringeLiquidHandler); add an orchestrator only when a **second** cell
exists to coordinate. Until then there is nothing to orchestrate.

## Operator web — ONE UI per Phase, via the orchestrator

There is **one operator web per Phase, not one per cell.** It talks to a
single backend — the Phase orchestrator — which:

- holds the **cell registry** (cell1→:17054, cell2→:17056, …),
- **aggregates** status (fans out to each cell's `GET /v1/status`),
- **routes** per-cell commands (web → orchestrator → the right cell `/v1`),
- runs **cross-cell protocols server-side** (dispense → arm transfer →
  weigh) — never in the browser, so a closed tab can't abort a run,
- MAY serve the web's static files itself, so the operator has one URL.

The web is structured as a **Phase shell** (all cells at a glance + a
cross-cell protocol runner) wrapping a reusable **cell view** — the
per-cell dashboard (`SyringeLiquidHandler/web`) is that cell view. Adding a
cell = one registry entry, not a new site.

Single-cell interim: with only one cell the web points straight at that
cell's `/v1`; introduce the orchestrator + shell when the second cell lands.
Keep the web's API base URL configurable so the repoint is trivial.

## Why incremental + uniform contract

Building one device/cell at a time is safe **because** every unit follows
the same SDLClaude contract (`/v1` shape, error envelope, lifecycle).
Uniformity is what keeps the later "compose into one class" cheap; the one
risk of incremental work — interface drift — is prevented by the standard.
