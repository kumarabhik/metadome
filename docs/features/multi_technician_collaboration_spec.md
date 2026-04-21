# Multi-Technician Collaboration Mode

**Phase:** 5
**Roadmap Item:** Feature: Multi-technician collaboration — two headsets, same overlay, same vehicle (collaborative repair)

---

## Objective

Allow two technicians wearing headsets to work on the same vehicle simultaneously with a shared spatial overlay — same CAD layers, same step position, same sensor readings visible to both. Use cases: senior tech guiding a junior tech remotely or co-located; two techs performing a two-person repair procedure (some EV battery replacements require two people); remote OEM technical support joining a live repair session.

---

## Use Cases

### 1. Co-Located Collaboration (Two Techs, Same Bay)
Two technicians physically present at the vehicle. Each wears a headset. They see the same overlay; can see each other's gaze direction as a visual cursor ("Tech B is looking at the coolant pump"). Either can advance steps or issue voice commands — shared session state.

### 2. Remote Expert Assist
Senior tech or OEM field service engineer connects remotely — sees the local tech's first-person spatial view streamed from the headset. Can annotate the overlay ("draw" a circle on the component the tech should look at) from a web dashboard. Remote participant does not need a headset — browser-based view.

### 3. Training Supervision
Matches with Apprentice Training Mode — supervisor can join an apprentice's session remotely to observe in real time. Can flag steps for post-session review without interrupting the tech. Separate from grading (scoring is automatic), this is for live coaching.

---

## Technical Architecture

### Session State Synchronization

In single-tech mode, session state lives on the local edge server — one client (headset) reads/writes. Multi-tech mode requires synchronized session state across two headsets.

**Architecture: Leader-Follower with Conflict-Free Step State**

- One headset is designated **Session Leader** (the one who initiated or was assigned the repair)
- Session state is authoritative on the edge server; both headsets subscribe via WebSocket
- Step advancement: only Session Leader can advance to next step — Follower sees a prompt "Request to advance" which Leader must confirm, OR Leader has granted Follower co-leader rights
- CAD layer visibility: both headsets render independently from the same layer manifest — no streaming of rendered frames between devices (avoids bandwidth and latency issues)
- Sensor data: both headsets receive the same UnifiedSensorState broadcast from SensorFusion — no delta

**Session State Schema Extension:**
```json
{
  "session_id": "uuid",
  "mode": "single | collaborative",
  "participants": [
    {
      "tech_id": "string",
      "role": "leader | follower | remote_observer",
      "headset_id": "string | null",
      "connection": "local | remote",
      "connected_at": "ISO8601"
    }
  ],
  "current_step": "int",
  "step_confirmed_by": ["tech_id"],
  "gaze_cursors": {
    "tech_id": {"x": "float", "y": "float", "z": "float", "anchor": "component_id"}
  }
}
```

### Gaze Cursor Rendering
- Each tech's gaze direction is transmitted as a 3D ray from headset to edge server at 10Hz
- Edge server broadcasts all participant gaze rays back to all headsets
- Each headset renders other participants' gaze as a colored dot on the relevant CAD component
- Color coding: Leader = blue, Follower = orange, Remote = purple
- Gaze cursor is an indicator only — does not affect interaction model

### Remote Observer Stream
For remote participants (use case 2 and 3):
- HoloLens 2 supports Miracast / Research Mode streaming — capture first-person RGB view at 1080p/30fps
- Stream sent to edge server → transcoded to H.264 → forwarded to remote observer via HTTPS WebRTC
- Latency target: < 500ms end-to-end for remote view (not real-time surgery — acceptable for annotation use case)
- Remote observer annotates via browser: draws on 2D projected view; annotation converted to 3D spatial coordinate and broadcast to headset as overlay element

### Remote Annotation Protocol
```
Remote draws circle at pixel (x, y) on 2D stream →
Edge server: unproject using depth map from HoloLens →
Generate SpatialAnnotation: {type: circle, world_position: vec3, radius: float, author: tech_id, ttl: 30s} →
Broadcast to local headset(s) →
Headset renders annotation as glowing ring in world space
```
TTL (time-to-live): annotations auto-expire after 30 seconds to avoid visual clutter.

---

## Safety Considerations

### HV-STOP in Multi-Tech Sessions
- HV-STOP triggers on **any** participant's proximity sensor — if either tech approaches HV zone, stop fires for all participants
- Both headsets receive simultaneous HV-STOP alert
- Session cannot resume until both participants have acknowledged the HV-STOP (dual acknowledgment required — same as single-tech but extended to both)
- Remote observer cannot trigger or acknowledge HV-STOP — safety actions require physical presence confirmation

### Step Confirmation
- In standard collaborative mode: Leader confirms step completion
- For safety-critical steps (flagged in step manifest as `safety_critical: true`): both co-located participants must confirm (dual confirmation required)
- Remote observer cannot confirm safety-critical steps

---

## Session Initiation

### Flow
1. Tech A starts repair session normally
2. Voice command: "Invite technician" OR manager assigns co-tech from dashboard
3. Edge server generates session invite token (valid 5 minutes)
4. Tech B's headset shows invite: "Tech A has invited you to join the Toyota bZ4X coolant diagnosis session"
5. Tech B accepts → joins as Follower
6. Both see shared overlay; Tech A retains Leader role

### Remote Observer Invitation (Use Case 2 & 3)
1. Tech A says "Request remote support" OR manager initiates from dashboard
2. Edge server generates one-time URL (valid 30 minutes, HTTPS, token-auth)
3. URL sent to remote expert via SMS or email through dealership notification system
4. Remote expert opens URL in browser — no app install required
5. WebRTC session established; remote expert sees live stream and can annotate

---

## Permissions Model

| Action | Leader | Follower (default) | Co-Leader | Remote Observer |
|---|---|---|---|---|
| Advance step | Yes | No (can request) | Yes | No |
| Add CAD layers | Yes | No | Yes | No |
| Acknowledge HV-STOP | Yes | Yes (co-located only) | Yes | No |
| Confirm safety-critical step | Yes | Yes (co-located) | Yes | No |
| Annotate overlay | Yes | Yes | Yes | Yes (remote annotation only) |
| End session | Yes | No | Yes | No |

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 24 | Design session state sync architecture; select WebSocket library and conflict resolution strategy |
| 25 | Build collaborative session state server on edge hardware |
| 26 | Build gaze cursor transmission and rendering |
| 27 | Build remote observer stream (HoloLens Research Mode → WebRTC) |
| 27 | Build remote annotation protocol |
| 28 | Safety review: HV-STOP dual-acknowledgment tested across 50 simulated scenarios |
| 29 | Closed beta: 2 dealerships with active apprentice programs (use case 3 as entry point) |
| 30 | GA — available on Professional and Enterprise tiers |

---

## Success Metrics

| Metric | Target |
|---|---|
| FTFR for collaborative sessions | ≥ 95% (higher than solo — two techs = fewer errors) |
| Remote expert session resolution rate (fault resolved without in-person visit) | ≥ 60% |
| Session setup time (invite → both headsets connected) | ≤ 60 seconds |
| HV-STOP dual-acknowledgment time | ≤ 15 seconds (from trigger to both confirmed) |
| Latency: gaze cursor update | ≤ 100ms p95 |
| Latency: remote observer stream | ≤ 500ms p95 |
