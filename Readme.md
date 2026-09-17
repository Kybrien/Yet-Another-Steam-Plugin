# SteamComplete — Easy API Complete Documentation

**Version documented:** SteamComplete v1.0.3 — Easy API / Developer Experience  
**Unreal Engine:** 5.6–5.8  
**Easy API surface:** 32 Blueprint nodes  
**Primary facade:** `Steam Complete (Easy API)` / `USteamCompleteSubsystem`

---

## Table of contents

1. [What the Easy API is](#1-what-the-easy-api-is)
2. [The three API levels](#2-the-three-api-levels)
3. [30-second Quick Start](#3-30-second-quick-start)
4. [The result pattern used everywhere](#4-the-result-pattern-used-everywhere)
5. [Presets](#5-presets)
6. [Complete Easy API node index](#6-complete-easy-api-node-index)
7. [Setup nodes](#7-setup-nodes)
8. [Status and diagnostics nodes](#8-status-and-diagnostics-nodes)
9. [Multiplayer nodes](#9-multiplayer-nodes)
10. [Voice nodes](#10-voice-nodes)
11. [Cloud nodes](#11-cloud-nodes)
12. [Progression nodes](#12-progression-nodes)
13. [Mods / Workshop nodes](#13-mods--workshop-nodes)
14. [Dedicated Server nodes](#14-dedicated-server-nodes)
15. [Common structs and enums](#15-common-structs-and-enums)
16. [Recommended use cases](#16-recommended-use-cases)
17. [When to leave Easy API and use Systems](#17-when-to-leave-easy-api-and-use-systems)
18. [Current Easy API limitations / gotchas](#18-current-easy-api-limitations--gotchas)
19. [Recommended project architecture](#19-recommended-project-architecture)
20. [Debugging checklist](#20-debugging-checklist)
21. [Default timeouts](#21-default-timeouts)
22. [Cheat sheet](#22-cheat-sheet)

---

# 1. What the Easy API is

SteamComplete contains a large amount of Steamworks functionality. The Easy API is a facade over the lower-level subsystems so that a normal game does not need to understand every Steam subsystem before it can host a game, save a file, enable voice, load a Workshop mod, or unlock an achievement.

The intended workflow is:

```text
Get Steam Complete
        ↓
Setup Steam Complete
        ↓
Use Easy nodes for normal game actions
        ↓
Use Systems / Advanced only when you need more control
```

The Easy API does **not** replace the complete API. It orchestrates the complete API.

A developer should be able to build most normal Steam functionality while only seeing these categories:

```text
SteamComplete
└── Easy
    ├── Setup
    ├── Status
    ├── Diagnostics
    ├── Multiplayer
    ├── Voice
    ├── Cloud
    ├── Progression
    ├── Mods
    └── Dedicated Server
```

---

# 2. The three API levels

## Easy

Use this first.

Best for:

- normal game logic;
- prototypes;
- Blueprint-only projects;
- standard co-op;
- common Steam Cloud saves;
- achievements and stats;
- common Workshop/mod workflows;
- normal voice chat;
- common dedicated-server startup.

The Easy API currently exposes **32 nodes**.

## Systems

Use Systems when the Easy node intentionally hides something you need.

Examples:

- custom lobby metadata;
- detailed lobby filters;
- team voice IDs;
- continuously updating proximity positions;
- per-player mute/volume;
- advanced Workshop queries;
- custom mod dependency management;
- detailed dedicated-server authentication;
- raw Cloud file/byte operations;
- custom session messages.

## Advanced

Use Advanced only when you deliberately need low-level Steam/SteamComplete behavior.

Examples include native-style IDs, diagnostic fields, specialized transport behavior, or lower-level Steamworks-facing operations.

---

# 3. 30-second Quick Start

For a normal co-op game:

```text
Event Init / GameInstance startup
        ↓
Get Steam Complete
        ↓
Setup Steam Complete
Preset = Co-op
        ↓
Break Steam Complete Setup Result
        ↓
Result.bSuccess ? Continue : Show/log error
```

The Co-op preset currently:

- initializes the SteamComplete runtime;
- requests Steam Datagram Relay prewarm;
- enables Global Steam Voice;
- uses Push To Talk;
- runs the health/setup diagnostic.

It does **not** automatically host or join a session. Use `Quick Host Steam Game`, `Quick Find Steam Games`, `Quick Join Steam Game`, or `Find And Join Steam Game` after setup.

---

# 4. The result pattern used everywhere

Most synchronous Easy nodes return `FSteamOperationResult`.

Important fields:

| Field | Meaning |
|---|---|
| `bSuccess` | The operation was accepted/completed successfully. |
| `Code` | High-level result code. |
| `Message` | Human-readable result suitable for logs/UI debugging. |
| `DebugMessage` | More technical information. Usually only needed while debugging. |
| `NativeCode` | Native error/result code when SteamComplete has one. |

Common result codes include:

- `Success`
- `SteamUnavailable`
- `NotLoggedIn`
- `InvalidParameter`
- `InvalidState`
- `NotFound`
- `AccessDenied`
- `Timeout`
- `NetworkFailure`
- `RateLimited`
- `Cancelled`
- `SteamError`
- `UnsupportedBySteamworksSDK`
- `UnsupportedByEngineVersion`
- `NotImplemented`
- `Unknown`

### Recommended Blueprint rule

Never assume a synchronous SteamComplete node succeeded simply because execution continued.

```text
Easy Node
   ↓
Break Steam Operation Result
   ↓
bSuccess
 ├─ true  → continue
 └─ false → log Message + optionally DebugMessage
```

Async Easy nodes instead expose `Success` and `Failed` execution delegates and also return an `FSteamOperationResult` in the callback.

---

# 5. Presets

The Easy API provides seven presets through `ESteamCompletePreset`.

| Preset | Relay prewarm | Voice | Voice channel | Input | Load enabled mods |
|---|---:|---:|---|---|---:|
| Single Player | No | No | — | — | No |
| Co-op | Yes | Yes | Global | Push To Talk | No |
| Listen Server Multiplayer | Yes | Yes | Global | Push To Talk | No |
| Dedicated Server Multiplayer | No | No | — | — | No |
| Competitive | Yes | Yes | Team | Push To Talk | No |
| Workshop / Modded Game | No | No | — | — | Yes |
| Custom | No | No | — | — | No |

All recommended presets currently run the health check after setup.

## Important preset behavior

Presets are **configuration recommendations**, not a hidden game framework.

For example:

- `Co-op` prepares relay and voice, but does not automatically create a lobby.
- `Dedicated Server Multiplayer` does not know your server description, ports, tags, map, or GSLT, so the dedicated server still needs an explicit start node.
- `Competitive` enables Team voice, but your game must still assign team IDs through the Systems Voice API.
- `Workshop / Modded Game` refreshes the mod catalog and mounts enabled mods, but it does not subscribe to arbitrary new Workshop items automatically.

---

# 6. Complete Easy API node index

## Setup — 5 nodes

1. `Get Steam Complete`
2. `Setup Steam Complete`
3. `Setup Steam Complete Custom`
4. `Get Recommended Steam Complete Setup`
5. `Get Last Steam Complete Setup Result`

## Status — 2 nodes

6. `Get Steam Complete Easy Status`
7. `Is Steam Complete Ready`

## Diagnostics — 1 node

8. `Run Steam Complete Setup Check`

## Multiplayer — 6 nodes

9. `Quick Host Steam Game`
10. `Quick Find Steam Games`
11. `Quick Join Steam Game`
12. `Find And Join Steam Game`
13. `Get Active Steam Game`
14. `Leave Steam Game`

## Voice — 4 nodes

15. `Enable Steam Voice Chat`
16. `Enable Steam Proximity Voice`
17. `Disable Steam Voice Chat`
18. `Set Steam Push To Talk`

## Cloud — 2 nodes

19. `Quick Save Steam Cloud Text`
20. `Quick Load Steam Cloud Text`

## Progression — 4 nodes

21. `Unlock Steam Achievement`
22. `Set Steam Int Stat`
23. `Set Steam Float Stat`
24. `Store Steam Progression`

## Mods — 6 nodes

25. `Refresh Steam Mods`
26. `Load All Enabled Steam Mods`
27. `Enable Steam Mod`
28. `Disable Steam Mod`
29. `Open Steam Workshop`
30. `Install And Load Steam Mod`

## Dedicated Server — 2 nodes

31. `Start Steam Dedicated Server Anonymous`
32. `Start Steam Dedicated Server With GSLT`

---

# 7. Setup nodes

## 7.1 Get Steam Complete

**Category:** `SteamComplete > Easy > Setup`  
**Type:** Pure  
**Returns:** `Steam Complete (Easy API)` subsystem reference

### What it does

Returns the `USteamCompleteSubsystem` facade attached to the current GameInstance.

This is the recommended entry point for the Easy API.

### Inputs

- `World Context Object` — resolved automatically in normal Blueprint usage.

### Output

- `Steam Complete` subsystem reference.

### Recommended use

Store the returned reference in your GameInstance, GameInstance subsystem, menu manager, or another long-lived object if you call Easy nodes frequently.

### Example

```text
Event BeginPlay
   ↓
Get Steam Complete
   ↓
Setup Steam Complete (Co-op)
```

---

## 7.2 Setup Steam Complete

**Category:** `SteamComplete > Easy > Setup`  
**Type:** Synchronous  
**Input:** `Preset`  
**Returns:** `FSteamCompleteSetupResult`

### What it does

Runs the recommended startup pipeline for a selected game style.

Internally it:

1. initializes the SteamComplete runtime;
2. verifies that the runtime is Ready;
3. obtains the GameInstance;
4. optionally prewarms Steam Datagram Relay;
5. optionally enables Steam Voice using the preset configuration;
6. optionally refreshes and mounts enabled mods;
7. optionally runs the SteamComplete health check;
8. stores the result as the Last Setup Result.

### Default preset

`Co-op`.

### Output — FSteamCompleteSetupResult

| Field | Meaning |
|---|---|
| `Preset` | Preset that was applied. |
| `bRuntimeReady` | SteamComplete runtime initialized successfully. |
| `bRelayPrewarmRequested` | Relay initialization request was successfully issued. |
| `bVoiceEnabled` | Voice subsystem accepted enable request. |
| `bModsLoaded` | Enabled mods were mounted successfully. |
| `Actions` | Human-readable list of steps that were attempted. |
| `Result` | Overall setup operation result. |
| `Health` | Health report when health checking was enabled. |

### Important behavior

A relay, voice, or mod warning is treated as **non-fatal** if the Steam runtime itself initialized correctly. In that case `Result.bSuccess` may still be true while `DebugMessage`, `Actions`, or `Health.Warnings` contain useful warnings.

### Recommended use

Call once near GameInstance/game startup, before normal Steam gameplay actions.

Do not call it every frame or every time you open a menu.

---

## 7.3 Setup Steam Complete Custom

**Category:** `SteamComplete > Easy > Setup`  
**Type:** Synchronous  
**Input:** `FSteamCompleteSetupOptions`  
**Returns:** `FSteamCompleteSetupResult`

### What it does

Runs the same setup pipeline as `Setup Steam Complete`, but exposes the individual switches.

### Setup options

| Option | Meaning |
|---|---|
| `Preset` | Label stored as the active preset. |
| `bPrewarmRelay` | Request Steam Datagram Relay initialization. |
| `bEnableVoiceChat` | Enable voice using `VoiceConfig`. |
| `bLoadEnabledMods` | Refresh mod catalog and mount enabled mods. |
| `VoiceConfig` | Full voice configuration. |
| `bRunHealthCheck` | Run safe diagnostic after setup. |

### Use this when

- your game is co-op but does not want voice at startup;
- you want Open Mic instead of Push To Talk;
- you want a modded multiplayer title;
- you want Global/Team/Proximity voice with a custom sample rate or distances;
- you want to postpone health checking.

### Example — modded co-op

```text
Make Steam Complete Setup Options
Preset = Custom
Prewarm Relay = true
Enable Voice Chat = true
Load Enabled Mods = true
Voice Channel = Global
Input Mode = Push To Talk
Run Health Check = true
        ↓
Setup Steam Complete Custom
```

---

## 7.4 Get Recommended Steam Complete Setup

**Category:** `SteamComplete > Easy > Setup`  
**Type:** Pure  
**Input:** Preset  
**Returns:** `FSteamCompleteSetupOptions`

### What it does

Returns the exact options SteamComplete would use for a preset.

### Why it is useful

This is the best way to make a “mostly standard” setup while changing one thing.

### Example — Co-op preset but Open Mic

```text
Get Recommended Steam Complete Setup (Co-op)
        ↓
Break / modify struct
Input Mode = Open Mic
        ↓
Setup Steam Complete Custom
```

This avoids manually recreating the preset defaults.

---

## 7.5 Get Last Steam Complete Setup Result

**Category:** `SteamComplete > Easy > Setup`  
**Type:** Pure  
**Returns:** `FSteamCompleteSetupResult`

### What it does

Returns the last result produced by `Setup Steam Complete` or `Setup Steam Complete Custom`.

### Good use cases

- settings/debug menu;
- support screen;
- telemetry;
- showing “Steam initialization failed” without rerunning setup;
- QA screenshots.

### Do not use it as

A live status node. For live state use `Get Steam Complete Easy Status` or `Run Steam Complete Setup Check`.

---

# 8. Status and diagnostics nodes

## 8.1 Get Steam Complete Easy Status

**Category:** `SteamComplete > Easy > Status`  
**Type:** Pure  
**Returns:** `FSteamCompleteEasyStatus`

### What it does

Returns a compact snapshot of the most useful SteamComplete state without exposing all twelve subsystems.

### Fields

| Field | Meaning |
|---|---|
| `bReady` | Runtime is Ready and Steam is available. |
| `bOnline` | Steam user/GameServer reports logged on. |
| `ActivePreset` | Last preset applied through Easy setup. |
| `LocalUserId` | Local Steam ID. |
| `PersonaName` | Local Steam persona name. |
| `AppId` | Current Steam App ID. |
| `bInMultiplayerSession` | SteamComplete session subsystem currently has an active session. |
| `bVoiceEnabled` | Voice system is enabled. |
| `DiscoveredModCount` | Number of mods in current catalog. |
| `MountedModCount` | Number of mounted mods. |

### Recommended use

Perfect for:

- main menu Steam status widget;
- debug overlay;
- QA build;
- “Steam connected” icon;
- mod count display.

### Performance

It is a pure snapshot getter. It does not perform network searches or Steam backend calls.

---

## 8.2 Is Steam Complete Ready

**Category:** `SteamComplete > Easy > Status`  
**Type:** Pure  
**Returns:** Bool

### What it does

Returns true when the SteamComplete runtime is Ready and Steam is available.

### Use this for

Simple branching:

```text
Is Steam Complete Ready
  ├─ true  → enable Steam menu actions
  └─ false → disable / show fallback UI
```

### Important

`Ready` does not mean every optional feature is configured. A project may be Ready while Cloud is disabled, relay is still initializing, or no multiplayer session exists.

Use `Run Steam Complete Setup Check` for a complete diagnostic.

---

## 8.3 Run Steam Complete Setup Check

**Category:** `SteamComplete > Easy > Diagnostics`  
**Type:** Synchronous diagnostic  
**Returns:** `FSteamCompleteHealthReport`

### What it does

Runs SteamComplete's safe health check.

It checks the current runtime and relevant client/server interfaces, and produces:

- errors;
- warnings;
- suggested fixes;
- a summary;
- feature-specific state.

### Safety behavior

If Steam runtime initialization fails, the diagnostic returns early and **does not touch native Steam accessors**. This is specifically designed to prevent a missing/unavailable Steam runtime from turning a diagnostic call into a DLL delay-load crash.

### Important health fields

The report includes, depending on client/server context:

- `bHealthy`
- `Core`
- `bDedicatedServerContext`
- GameServer availability/login/security
- required interface availability
- authentication availability
- user stats availability
- inventory availability / definition count
- Remote Storage availability
- Steam Cloud account/app state
- Cloud quota and file count
- local subscription/license state
- VAC state reported by Steam
- SteamNetworkingSockets availability
- relay readiness and relay status
- session state consistency
- voice state consistency
- mod loader availability
- mod-set validation
- callback pump mode/owner
- `Warnings`
- `Errors`
- `SuggestedFixes`
- `Summary`

### Recommended use

Run it:

- after setup during development;
- in a hidden support/debug menu;
- before shipping a QA build;
- when a user reports “Steam doesn't work”;
- after changing OnlineSubsystemSteam/SteamShared settings.

### Do not run

Every frame.

---

# 9. Multiplayer nodes

The Easy multiplayer layer uses Steam lobbies plus SteamNetworkingSockets P2P/SDR session orchestration.

Easy searches automatically identify SteamComplete-compatible sessions by requiring:

```text
steamcomplete_session_protocol = 1
steamcomplete_session_transport = p2p_sdr
```

This prevents `Quick Find Steam Games` from returning arbitrary unrelated Steam lobbies.

---

## 9.1 Quick Host Steam Game

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Async  
**Inputs:** `FSteamCompleteQuickHostOptions`, optional Timeout  
**Outputs:** `Success(Session, Result)`, `Failed(Session, Result)`

### What it does

Creates a complete SteamComplete host session.

Internally it:

1. validates Steam and the host options;
2. creates a SteamNetworkingSockets P2P listen socket;
3. creates a Steam lobby;
4. applies the lobby settings;
5. adds SteamComplete protocol metadata;
6. starts the hosted SteamComplete session;
7. returns the resulting `FSteamMultiplayerSessionInfo`.

### Host options

| Field | Default | Meaning |
|---|---:|---|
| `LobbyName` | `Steam Game` | Human-readable lobby metadata name. |
| `Privacy` | Friends Only | Private / Friends Only / Public / Invisible. |
| `MaxPlayers` | 4 | Maximum lobby members. Range 1–250. |
| `bJoinable` | true | Whether the lobby accepts joins. |
| `VirtualPort` | 0 | SteamNetworkingSockets virtual port. Keep 0 for normal games. |

### Recommended use

Use for normal listen-server/co-op hosting.

### Typical graph

```text
Setup Steam Complete (Co-op)
        ↓
Quick Host Steam Game
Options:
  LobbyName = "My Lobby"
  Privacy = Friends Only
  MaxPlayers = 4
        ↓ Success
Open / travel to gameplay map as host
```

### Important

The node creates Steam session connectivity, but **your game still controls Unreal map travel and gameplay state**.

### Timeout

`0` means use the configured session timeout. Current default: **35 seconds**.

---

## 9.2 Quick Find Steam Games

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Async  
**Inputs:** `FSteamCompleteQuickFindOptions`, optional Timeout  
**Outputs:** `Success(SearchResult, Result)`, `Failed(SearchResult, Result)`

### What it does

Searches for compatible SteamComplete lobbies.

It automatically adds the SteamComplete protocol and P2P/SDR transport filters.

### Find options

| Field | Default | Meaning |
|---|---:|---|
| `MinimumOpenSlots` | 1 | Require at least this many open slots. Set 0 to disable the open-slot filter. |
| `Distance` | Default | Steam geographic lobby distance filter. |
| `MaxResults` | 20 | Maximum returned lobbies, 1–50. |

### Output — FSteamLobbySearchResult

- `Lobbies` — array of `FSteamLobbyInfo`.
- `Count` — number of returned lobbies.

Useful `FSteamLobbyInfo` fields:

- `LobbyId`
- `LobbyName`
- `OwnerId`
- `MaxMembers`
- `CurrentMembers`
- `AvailableSlots`
- `bHasMemberDetails`
- `bIsLocalUserMember`
- `bIsLocalUserOwner`
- `LobbyData`

### Use this when

You want to build an actual lobby browser/menu and let the player choose a game.

### Typical graph

```text
Quick Find Steam Games
        ↓ Success
ForEach SearchResult.Lobbies
        ↓
Create server row widget
        ↓
Store LobbyId on row
```

Then on user click:

```text
Quick Join Steam Game (Selected LobbyId)
```

### Timeout

`0` means configured lobby timeout. Current default: **25 seconds**.

### Steam limitation

Only one Steam lobby search should be active at a time. SteamComplete returns `InvalidState` if another search is already active.

---

## 9.3 Quick Join Steam Game

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Async  
**Input:** `FSteamLobbyId`, optional Timeout  
**Outputs:** `Success(Session, Result)`, `Failed(Session, Result)`

### What it does

Joins a selected SteamComplete lobby and establishes the corresponding SteamComplete multiplayer connection/handshake.

The node validates that the lobby contains compatible SteamComplete session metadata.

### Use this when

- player selected a row from `Quick Find Steam Games`;
- you already have a lobby ID from another SteamComplete flow;
- you are implementing invite/friend join logic at a higher level.

### Important

You cannot join while another SteamComplete multiplayer host/join operation is active.

### Output session

`FSteamMultiplayerSessionInfo` gives you:

- Role (`Host` / `Client` / `None`)
- State
- LobbyId
- HostUserId
- host listen socket or client host connection
- virtual port
- peer counts
- active flag
- last result

### Timeout

`0` = configured session timeout, currently **35 seconds**.

---

## 9.4 Find And Join Steam Game

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Async one-click workflow  
**Inputs:** `FSteamCompleteQuickFindOptions`, optional Timeout  
**Outputs:** `Success(Session, Result)`, `Failed(Session, Result)`

### What it does

Combines search and join into one node.

Pipeline:

```text
Search compatible SteamComplete lobbies
        ↓
No result? → Failed / NotFound
        ↓
Take first lobby returned by Steam
        ↓
Join Steam multiplayer session
        ↓
Success / Failed
```

### Important selection behavior

The current implementation joins **the first compatible lobby returned by Steam**.

It does not currently:

- ping/rank all lobbies;
- pick by player count;
- pick by custom skill rating;
- retry the second lobby when the first join fails.

Use `Quick Find Steam Games` + your own selection logic when you need control.

### Best use cases

- “Quick Play” button;
- very simple co-op matchmaking;
- internal testing;
- games where any available lobby is acceptable.

---

## 9.5 Get Active Steam Game

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Pure  
**Returns:** `FSteamMultiplayerSessionInfo`

### What it does

Returns the current high-level SteamComplete multiplayer session state.

### Recommended use

Use in:

- pause menu;
- lobby UI;
- debug overlay;
- conditional UI;
- connection state handling.

### Important session states

- `Idle`
- `Hosting`
- `Joining Lobby`
- `Connecting`
- `Handshaking`
- `Ready`
- `Leaving`
- `Failed`

For normal gameplay networking, `Ready` is the state you usually care about.

---

## 9.6 Leave Steam Game

**Category:** `SteamComplete > Easy > Multiplayer`  
**Type:** Synchronous  
**Input:** `bLingerReliableData`  
**Returns:** `FSteamOperationResult`

### What it does

Leaves/tears down the active SteamComplete multiplayer session.

### bLingerReliableData

Controls whether reliable Steam networking data is allowed to linger during connection close.

For normal menu/disconnect behavior, the default `false` is appropriate.

Use `true` only when you deliberately want pending reliable networking data to be given a chance to flush during shutdown.

### Typical flow

```text
Leave Game button
       ↓
Leave Steam Game
       ↓ success
Return to main menu
```

---

# 10. Voice nodes

SteamComplete Voice uses Steam Voice for capture/compression/decompression and the SteamComplete multiplayer/session transport for packet routing.

**Important:** enabling voice does not mean audio is immediately transmitted. The voice subsystem requires a Ready SteamComplete multiplayer session before capture/transmission is active.

---

## 10.1 Enable Steam Voice Chat

**Category:** `SteamComplete > Easy > Voice`  
**Type:** Synchronous  
**Input:** Input Mode  
**Returns:** `FSteamOperationResult`

### What it does

Enables **Global** voice chat.

Creates a default voice config with:

- Channel = Global
- InputMode = selected `Push To Talk` or `Open Mic`

### Use this when

Every connected player should be able to hear everyone else.

### Dedicated server

Local voice capture is not available in `UE_SERVER` builds. The node returns a failure in a dedicated-server build.

---

## 10.2 Enable Steam Proximity Voice

**Category:** `SteamComplete > Easy > Voice`  
**Type:** Synchronous  
**Inputs:** Max Distance, Full Volume Distance, Input Mode  
**Returns:** `FSteamOperationResult`

### Defaults

- Max Distance = `2000` Unreal units
- Full Volume Distance = `300` Unreal units
- Input = Push To Talk

### What it does

Enables Steam voice with Channel = Proximity.

The values are sanitized:

- max distance is at least `1`;
- full-volume distance is clamped between `0` and max distance.

### Critical usage requirement

The Easy node configures proximity attenuation, but **your game must keep SteamComplete informed about the local player's current world position**.

At present that requires the Systems node:

```text
SteamComplete > Systems > Voice > Routing
Set Steam Voice Position
```

Recommended pattern:

```text
Pawn / Character periodic update
        ↓
Get actor location
        ↓
Set Steam Voice Position
```

You do not need to call it literally every render frame; update frequently enough for your game's movement speed and desired voice spatial responsiveness.

### When Easy alone is not enough

If you use Proximity voice, you currently need this one Systems node for position updates.

---

## 10.3 Disable Steam Voice Chat

**Category:** `SteamComplete > Easy > Voice`  
**Type:** Synchronous  
**Returns:** `FSteamOperationResult`

### What it does

Disables the SteamComplete voice subsystem and stops local Steam voice capture/routing.

Use for:

- voice off option;
- leaving voice-enabled game mode;
- parental/user settings;
- temporary voice shutdown.

---

## 10.4 Set Steam Push To Talk

**Category:** `SteamComplete > Easy > Voice`  
**Type:** Synchronous  
**Input:** `bPressed`  
**Returns:** `FSteamOperationResult`

### What it does

Sets the current Push To Talk button state.

### Correct Enhanced Input pattern

```text
PTT Input Action - Started
        ↓
Set Steam Push To Talk(true)

PTT Input Action - Completed / Canceled
        ↓
Set Steam Push To Talk(false)
```

### Important

The value is ignored when the current input mode is Open Mic.

---

# 11. Cloud nodes

The Easy Cloud nodes intentionally expose the most common case: **UTF-8 text files**.

Use Systems Cloud if you need raw byte arrays, detailed file management, quota management, sync platforms, or dynamic-sync features.

---

## 11.1 Quick Save Steam Cloud Text

**Category:** `SteamComplete > Easy > Cloud`  
**Type:** Async  
**Inputs:** File Name, Text, optional Timeout  
**Outputs:** `Success(File, Result)`, `Failed(File, Result)`

### What it does

Encodes the supplied Unreal string as UTF-8 and writes it asynchronously to Steam Remote Storage.

### Output — FSteamCloudFileInfo

- FileName
- SizeBytes
- ModifiedUnixTime
- bPersistedInCloud
- SyncPlatforms

### Typical use

Serialize a small save/settings object to JSON:

```text
SaveGame / Settings struct
       ↓
Convert to JSON string
       ↓
Quick Save Steam Cloud Text
FileName = "profile.json"
       ↓ Success
Show "Saved"
```

### File-size behavior

The Easy node uses Steam's async whole-file write path. Very large files may exceed Steam's per-call limit; use the Systems API if you need specialized chunked/large-file behavior.

### Timeout

`0` uses the configured Cloud timeout. Current default: **30 seconds**.

---

## 11.2 Quick Load Steam Cloud Text

**Category:** `SteamComplete > Easy > Cloud`  
**Type:** Async  
**Inputs:** File Name, optional Timeout  
**Outputs:** `Success(Read, Result)`, `Failed(Read, Result)`

### What it does

Reads a whole Steam Cloud file and decodes it as UTF-8 text.

### Output — FSteamCloudReadResult

- `File` — file metadata
- `Data` — raw bytes
- `Text` — decoded text
- `bDecodedAsText` — true for this Easy text node

### Typical use

```text
Quick Load Steam Cloud Text("profile.json")
       ↓ Success
Read.Text
       ↓
Parse JSON
       ↓
Apply profile/settings
```

### Missing file

If the file does not exist, the node fails with `NotFound`.

That can be a normal first-launch condition. Handle it by creating default data rather than treating it as a fatal error.

---

# 12. Progression nodes

These nodes wrap the common achievements/stats workflow.

A key usability feature is `bStoreImmediately`: Steam stats and achievements normally require a StoreStats-style commit. Easy nodes can do that automatically.

---

## 12.1 Unlock Steam Achievement

**Category:** `SteamComplete > Easy > Progression`  
**Type:** Synchronous  
**Inputs:** Achievement API Name, bStoreImmediately  
**Returns:** `FSteamOperationResult`

### Default

`bStoreImmediately = true`

### What it does

1. calls the progression subsystem's achievement unlock;
2. if successful and `bStoreImmediately` is true, immediately stores stats/achievements.

### Important

Use the **Steamworks API name**, not the display title shown to players.

### Typical use

```text
Boss defeated
    ↓
Unlock Steam Achievement
AchievementApiName = "ACH_FIRST_BOSS"
Store Immediately = true
```

### Batch optimization

If you unlock/update many values at once:

- set `bStoreImmediately = false` on each mutation;
- call `Store Steam Progression` once at the end.

---

## 12.2 Set Steam Int Stat

**Category:** `SteamComplete > Easy > Progression`  
**Type:** Synchronous  
**Inputs:** Stat API Name, integer Value, bStoreImmediately  
**Returns:** `FSteamOperationResult`

### Use for

- kills;
- wins;
- collectibles;
- integer counters;
- integer-backed progression.

### Example

```text
Set Steam Int Stat
StatApiName = "STAT_TOTAL_WINS"
Value = 42
Store Immediately = true
```

---

## 12.3 Set Steam Float Stat

**Category:** `SteamComplete > Easy > Progression`  
**Type:** Synchronous  
**Inputs:** Stat API Name, float Value, bStoreImmediately  
**Returns:** `FSteamOperationResult`

### Use for

- best time;
- distance;
- floating-point scores;
- averaged/continuous metrics configured as float stats in Steamworks.

---

## 12.4 Store Steam Progression

**Category:** `SteamComplete > Easy > Progression`  
**Type:** Synchronous  
**Returns:** `FSteamOperationResult`

### What it does

Commits pending Steam stats and achievement changes.

### Recommended pattern for batches

```text
Set Steam Int Stat(..., StoreImmediately=false)
Set Steam Float Stat(..., StoreImmediately=false)
Unlock Steam Achievement(..., StoreImmediately=false)
        ↓
Store Steam Progression
```

This is preferable when several values change during one checkpoint/end-of-match operation.

---

# 13. Mods / Workshop nodes

The Easy Mods API combines the Workshop acquisition layer with the SteamComplete Unreal Mod Loader.

The Mod Loader understands:

- Steam Workshop mods;
- local mods;
- optional `steamcomplete.mod.json` manifests;
- Pak containers;
- Pak + IoStore layouts;
- cooked loose content;
- dependencies;
- optional dependencies;
- conflicts;
- mount priority;
- persisted enabled state;
- restart-required mods.

---

## 13.1 Refresh Steam Mods

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Synchronous  
**Output:** Discovered Mods count + `FSteamOperationResult`

### What it does

Refreshes the Mod Loader catalog from its configured sources.

If successful, `DiscoveredMods` is the size of the refreshed mod catalog.

### Use this when

- opening a Mod Manager screen;
- after Workshop download/subscribe state changes;
- after adding/removing local mod files;
- before displaying current mod state.

### It does not

Automatically mount every discovered mod. Use `Load All Enabled Steam Mods` for that.

---

## 13.2 Load All Enabled Steam Mods

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Synchronous  
**Output:** Mounted Mods count + `FSteamOperationResult`

### What it does

1. refreshes the mod catalog;
2. mounts all mods currently marked enabled;
3. returns the current mounted-mod count.

### Recommended use

- startup of a modded game;
- applying a saved mod configuration;
- “Reload Enabled Mods” button where runtime mounting is supported.

### Dependency behavior

The lower-level Mod Loader resolves dependency order and validates enabled mods. Invalid dependencies/conflicts can produce failures or health-check warnings.

### Restart-required behavior

Some mods cannot be safely live-applied. Their enabled state can be persisted while actual loading waits for restart.

---

## 13.3 Enable Steam Mod

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Synchronous  
**Inputs:** ModId, bApplyImmediately  
**Returns:** `FSteamOperationResult`

### Default

`bApplyImmediately = true`

### What it does

Marks a catalog mod as enabled and optionally attempts to apply/mount it immediately.

### ModId vs Workshop Item ID

This node expects the SteamComplete **ModId**, not necessarily the Workshop numeric item ID.

For a newly selected Workshop item where you only know the Workshop Item ID, use `Install And Load Steam Mod` instead.

### Use `bApplyImmediately = false` when

- you are building a mod selection screen;
- the player will enable/disable several mods before pressing Apply;
- you prefer to apply changes on next restart.

---

## 13.4 Disable Steam Mod

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Synchronous  
**Inputs:** ModId, bApplyImmediately  
**Returns:** `FSteamOperationResult`

### What it does

Marks a mod disabled and optionally attempts to unmount/apply the change immediately.

### Dependency protection

The Mod Loader can refuse an unsafe unmount when another enabled/mounted mod depends on the target.

### Recommended UI behavior

If disable fails:

- show `Result.Message`;
- refresh the mod catalog;
- optionally inspect the Systems validation report for detailed dependency/conflict information.

---

## 13.5 Open Steam Workshop

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Synchronous  
**Returns:** `FSteamOperationResult`

### What it does

Opens the current app's Steam Workshop hub using the Steam overlay/Workshop subsystem.

### Use this for

A simple in-game `Browse Workshop` button.

### Dedicated server

Not available in a dedicated-server build.

### Important

This requires the Steam UI/overlay context to be available. If overlay behavior is unavailable, inspect the returned result and health report.

---

## 13.6 Install And Load Steam Mod

**Category:** `SteamComplete > Easy > Mods`  
**Type:** Async one-click workflow  
**Inputs:** Workshop Item ID, bHighPriority, optional Timeout  
**Outputs:** Progress, Success(Mod, Result), Failed(Mod, Result)

### Default

- `bHighPriority = true`
- timeout `0` = Workshop download default, currently **300 seconds**

### What it does

This is the main one-click Workshop-to-Unreal workflow.

Pipeline:

```text
Validate Workshop Item ID
      ↓
Check Workshop state
      ↓
Subscribe if necessary
      ↓
Download / update if necessary
      ↓
Refresh SteamComplete mod catalog
      ↓
Find the mod matching the Workshop Item ID
      ↓
Enable the SteamComplete ModId
      ↓
Attempt immediate mount
      ↓
Success
```

### Progress output

The Progress delegate provides:

- ItemId
- BytesDownloaded
- BytesTotal
- Progress01

Use it directly for a download progress bar.

### Success behavior

There are two valid success cases:

1. **Installed, enabled, mounted** — ready now.
2. **Installed and enabled, restart required** — node still succeeds, but `Mod.bRequiresRestart` is true and the result message explains that restart is required.

### Output Mod descriptor

Useful fields include:

- ModId
- DisplayName
- Version
- Description
- Source
- WorkshopItemId
- InstallDirectory
- ManifestPath
- VirtualRoot
- Dependencies
- OptionalDependencies
- Conflicts
- MountPriority
- bInstalled
- bNeedsUpdate
- bDownloading
- bEnabled
- bMounted
- bHasManifest
- bHasPak
- bHasIoStore
- bHasLooseContent
- bRequiresRestart
- SizeOnDiskBytes
- Warning

### Recommended use

Steam Workshop browser row:

```text
User clicks Install
       ↓
Install And Load Steam Mod(ItemId)
       ↓ Progress
Update progress bar
       ↓ Success
If Mod.bRequiresRestart
   → show Restart Required
Else
   → show Installed / Enabled
```

---

# 14. Dedicated Server nodes

These nodes are only intended for **actual UE dedicated-server builds** (`UE_SERVER`).

They combine server metadata initialization and the first Steam GameServer LogOn call.

---

## 14.1 Start Steam Dedicated Server Anonymous

**Category:** `SteamComplete > Easy > Dedicated Server`  
**Type:** Synchronous request  
**Input:** `FSteamDedicatedServerConfig`  
**Returns:** `FSteamOperationResult`

### What it does

1. initializes Steam GameServer metadata using the supplied config;
2. if initialization succeeds, calls anonymous GameServer LogOn.

### Important asynchronous reality

A successful return means **the LogOn request was issued**, not that Steam has already confirmed the server is connected.

The underlying result message explicitly indicates that you should wait for the dedicated-server connection event/state.

### Config fields

| Field | Meaning |
|---|---|
| `Product` | Steam master-server product identifier. Empty uses current App ID string. |
| `GameDescription` | Human-readable description. Required. |
| `ModDirectory` | Install/mod directory name, not a path. Required. |
| `ServerName` | Browser display name. |
| `MapName` | Current map name. |
| `MaxPlayers` | Maximum human/bot capacity. |
| `BotPlayers` | Current bot player count. |
| `bPasswordProtected` | Browser metadata flag. |
| `Region` | Optional region string. Empty = Steam/default world. |
| `GameTags` | Browser filter tags. |
| `GameData` | Opaque game/browser data. |
| `Rules` | Key/value server rules. |
| `bEnableHeartbeats` | Enable Steam server heartbeats. |

### Validation

- `MaxPlayers >= 1`
- `BotPlayers` between 0 and MaxPlayers
- rule keys cannot be empty
- Product must resolve non-empty
- GameDescription required
- ModDirectory required
- ModDirectory must be a folder name, not a full path

### Client build behavior

In a normal client/editor build this node returns failure: it requires a dedicated-server build.

---

## 14.2 Start Steam Dedicated Server With GSLT

**Category:** `SteamComplete > Easy > Dedicated Server`  
**Type:** Synchronous request  
**Inputs:** `FSteamDedicatedServerConfig`, Game Server Login Token  
**Returns:** `FSteamOperationResult`

### What it does

Same initialization pipeline as anonymous startup, then performs authenticated Steam GameServer LogOn with a GSLT.

### Security invariant

SteamComplete passes the GSLT directly into the native Steam call.

It is **not persisted** and **not logged** by SteamComplete.

### Important

Do not put production GSLTs in:

- Blueprint defaults committed to source control;
- public config repositories;
- logs;
- screenshots/tutorial material.

Provide the token at runtime from your server deployment environment/secrets configuration.

### Successful return

As with anonymous login, success means the connection request was started. Wait for server connection state/callback before assuming the GameServer is fully online.

---

# 15. Common structs and enums

## 15.1 ESteamCompletePreset

```text
Single Player
Co-op
Listen Server Multiplayer
Dedicated Server Multiplayer
Competitive
Workshop / Modded Game
Custom
```

---

## 15.2 FSteamCompleteSetupOptions

```text
Preset
bPrewarmRelay
bEnableVoiceChat
bLoadEnabledMods
VoiceConfig
bRunHealthCheck
```

---

## 15.3 FSteamCompleteSetupResult

```text
Preset
bRuntimeReady
bRelayPrewarmRequested
bVoiceEnabled
bModsLoaded
Actions[]
Result
Health
```

---

## 15.4 FSteamCompleteEasyStatus

```text
bReady
bOnline
ActivePreset
LocalUserId
PersonaName
AppId
bInMultiplayerSession
bVoiceEnabled
DiscoveredModCount
MountedModCount
```

---

## 15.5 FSteamCompleteQuickHostOptions

```text
LobbyName = "Steam Game"
Privacy = Friends Only
MaxPlayers = 4
bJoinable = true
VirtualPort = 0
```

---

## 15.6 FSteamCompleteQuickFindOptions

```text
MinimumOpenSlots = 1
Distance = Default
MaxResults = 20
```

---

## 15.7 ESteamLobbyType

```text
Private
Friends Only
Public
Invisible
```

---

## 15.8 ESteamLobbyDistance

```text
Close
Default
Far
Worldwide
```

---

## 15.9 ESteamVoiceInputMode

```text
Push To Talk
Open Mic
```

---

## 15.10 FSteamMultiplayerSessionInfo

Important fields:

```text
Role
State
LobbyId
HostUserId
ListenSocketId      // host
HostConnectionId    // client
VirtualPort
PeerCount
ReadyPeerCount
bActive
LastResult
```

---

# 16. Recommended use cases

## 16.1 Single-player Steam game with achievements + Cloud

### Startup

```text
Get Steam Complete
        ↓
Setup Steam Complete (Single Player)
```

### Save

```text
Serialize profile to JSON string
        ↓
Quick Save Steam Cloud Text("profile.json", Json)
```

### Load

```text
Quick Load Steam Cloud Text("profile.json")
   ├─ Success → parse Read.Text
   └─ Failed + NotFound → create default profile
```

### Achievement

```text
Unlock Steam Achievement("ACH_FINISH_CHAPTER_1")
```

This project may never need to touch the specialized subsystems.

---

## 16.2 Basic 4-player friends-only co-op

### Startup

```text
Setup Steam Complete(Co-op)
```

### Host button

```text
Quick Host Steam Game
LobbyName = PlayerName + "'s Game"
Privacy = Friends Only
MaxPlayers = 4
```

### Join browser

```text
Quick Find Steam Games
Distance = Default
MaxResults = 20
        ↓
Populate UI rows
        ↓ player selects row
Quick Join Steam Game(Row.LobbyId)
```

### PTT

```text
Input Started   → Set Steam Push To Talk(true)
Input Completed → Set Steam Push To Talk(false)
```

### Leave

```text
Leave Steam Game
```

---

## 16.3 One-button Quick Play

```text
Setup Steam Complete(Co-op)
        ↓
Find And Join Steam Game
        ├─ Success → enter joined game
        └─ Failed / NotFound
                 ↓
             Quick Host Steam Game
```

This creates a simple “join if possible, otherwise host” experience.

### Caveat

`Find And Join` currently picks the first Steam-returned compatible lobby. For skill/rank/ping-aware matchmaking, use Systems or wait for a richer matchmaking layer.

---

## 16.4 Public lobby browser

Use:

1. `Quick Find Steam Games`
2. build one UI row per `FSteamLobbyInfo`
3. display lobby name and available data
4. store LobbyId in the row widget
5. call `Quick Join Steam Game` when clicked

If you need custom filters such as map, difficulty, region tag, game mode, version, or custom numerical filters, move the **search step only** to `SteamComplete > Systems > Lobby`. You can still use `Quick Join Steam Game` afterward if the lobby is SteamComplete-session compatible.

---

## 16.5 Proximity voice survival game

### Setup

```text
Setup Steam Complete(Listen Server Multiplayer)
        ↓
Enable Steam Proximity Voice
MaxDistance = 2500
FullVolumeDistance = 250
InputMode = Push To Talk
```

### Position feed

Current Easy API needs one Systems call:

```text
Character periodic update
        ↓
GetActorLocation
        ↓
Set Steam Voice Position
```

### PTT

```text
Input Started   → Set Steam Push To Talk(true)
Input Completed → Set Steam Push To Talk(false)
```

---

## 16.6 Competitive team voice

The `Competitive` preset configures Team voice + Push To Talk.

Your game still owns team assignment.

When team changes, call the Systems node:

```text
Set Steam Voice Team(TeamId)
```

Then use the Easy `Set Steam Push To Talk` node for input.

This is intentional because SteamComplete cannot infer whether your game's Team 0/1/2/etc. corresponds to Unreal teams, factions, squads, parties, or another gameplay concept.

---

## 16.7 Workshop-based mod manager

### On opening Mod Manager

```text
Refresh Steam Mods
        ↓
Display discovered mod catalog using Systems getter if full descriptors are needed
```

### Browse online content

```text
Open Steam Workshop
```

### Install from known Workshop Item ID

```text
Install And Load Steam Mod(ItemId)
       ↓ Progress
Update progress bar
       ↓ Success
Check Mod.bRequiresRestart
```

### Toggle existing mod

```text
Enable Steam Mod(ModId, ApplyImmediately)
Disable Steam Mod(ModId, ApplyImmediately)
```

### Apply enabled set

```text
Load All Enabled Steam Mods
```

For a sophisticated in-game Workshop browser with search, tags, voting, descriptions, creator tools, etc., use the Workshop Systems nodes.

---

## 16.8 Modded game startup

Simplest version:

```text
Setup Steam Complete(Workshop / Modded Game)
```

This refreshes and mounts enabled mods as part of setup.

If the game is also multiplayer, use a Custom setup:

```text
Get Recommended Setup(Co-op)
        ↓ modify
bLoadEnabledMods = true
        ↓
Setup Steam Complete Custom
```

---

## 16.9 Batched end-of-match progression

Avoid three immediate StoreStats calls:

```text
Set Steam Int Stat("STAT_WINS", NewWins, false)
Set Steam Float Stat("STAT_BEST_TIME", BestTime, false)
Unlock Steam Achievement("ACH_10_WINS", false)
        ↓
Store Steam Progression
```

This is the preferred Easy pattern when multiple progression values change at once.

---

## 16.10 Dedicated server startup

### Anonymous

```text
Dedicated server GameInstance startup
        ↓
Get Steam Complete
        ↓
Setup Steam Complete(Dedicated Server Multiplayer)
        ↓
Start Steam Dedicated Server Anonymous(Config)
        ↓
Wait for Systems dedicated-server connection event/state
```

### GSLT

```text
Read token from secure deployment environment
        ↓
Start Steam Dedicated Server With GSLT(Config, Token)
        ↓
Wait for connected state
```

Do not assume the synchronous node return means backend login has already completed.

---

## 16.11 Support / “Steam isn't working” screen

```text
Run Steam Complete Setup Check
        ↓
Display:
Summary
Errors
Warnings
SuggestedFixes
```

This can dramatically reduce support ambiguity because the report distinguishes runtime, login, interfaces, Cloud, relay, session consistency, voice consistency, mods, etc.

---

# 17. When to leave Easy API and use Systems

Easy is deliberately opinionated. Move down to Systems when your game needs explicit control.

| Requirement | Easy | Systems needed? |
|---|---:|---:|
| Host normal Steam co-op | Yes | No |
| Find compatible games | Yes | No |
| Custom lobby metadata | Limited | Yes |
| Custom lobby string/numerical filters | No | Yes |
| Quick Play first available lobby | Yes | No |
| Rank/ping/custom matchmaking | No | Yes |
| Global voice | Yes | No |
| Proximity voice enable | Yes | Position update requires Systems |
| Team voice preset | Yes | Team ID assignment requires Systems |
| Per-player mute/volume | No | Yes |
| Text Cloud file | Yes | No |
| Binary Cloud file | No | Yes |
| Achievement/stat write | Yes | No |
| Leaderboards | Not in Easy v1.0.3 | Yes |
| Install known Workshop Item ID | Yes | No |
| In-game Workshop query/browser | No | Yes |
| Detailed mod catalog/validation UI | Partial | Yes |
| Dedicated server initial LogOn | Yes | No |
| Dedicated server player auth/rules/status callbacks | No | Yes |

---

# 18. Current Easy API limitations / gotchas

These are important for v1.0.3.

## 18.1 Proximity voice still needs position updates

`Enable Steam Proximity Voice` does not magically know Pawn positions.

Use Systems `Set Steam Voice Position` regularly.

## 18.2 Competitive Team voice still needs TeamId

The Competitive preset selects Team voice but defaults `TeamId` to 0. Your game must set the correct TeamId through Systems Voice Routing.

## 18.3 Find And Join is intentionally simple

It joins the **first** compatible lobby returned by Steam.

There is no scoring/retry/ranking layer yet.

## 18.4 Easy multiplayer is SteamComplete-session specific

Easy search intentionally filters for:

```text
steamcomplete_session_protocol=1
steamcomplete_session_transport=p2p_sdr
```

That is good for safety/compatibility, but means Easy search is not a generic browser for every arbitrary Steam lobby.

## 18.5 Unreal map travel remains your responsibility

Easy Host/Join sets up SteamComplete's lobby/session/network connection state. It does not decide your game's map loading, lobby map, seamless travel, PlayerController lifecycle, or gameplay spawning policy.

## 18.6 Dedicated server startup is a request, not final confirmation

The Easy server nodes issue LogOn. Connection confirmation remains asynchronous.

## 18.7 Cloud NotFound is not necessarily an error in game logic

A new player may simply have no save yet.

Handle `NotFound` as “create default save” where appropriate.

## 18.8 Mod install success may still require restart

Always inspect `Mod.bRequiresRestart`.

## 18.9 Easy API does not expose every existing feature

v1.0.3 focuses on the most common workflows. Features such as detailed authentication, inventory, leaderboards, raw networking messages, creator-side Workshop tools, and advanced lobby operations remain under Systems/Advanced.

This is intentional: Easy should stay small enough to understand.

---

# 19. Recommended project architecture

A clean Blueprint architecture is:

```text
BP_GameInstance
or
your own GameInstanceSubsystem
        │
        ├── Get Steam Complete
        ├── Setup Steam Complete
        ├── store setup/status if desired
        │
        ├── Menu / Multiplayer manager
        │      ├── Quick Host
        │      ├── Quick Find
        │      ├── Quick Join
        │      └── Leave
        │
        ├── Save manager
        │      ├── Quick Save Cloud Text
        │      └── Quick Load Cloud Text
        │
        ├── Progression manager
        │      ├── Set Stat
        │      ├── Unlock Achievement
        │      └── Store Progression
        │
        └── Mod manager
               ├── Refresh Mods
               ├── Install And Load
               ├── Enable / Disable
               └── Load Enabled
```

You do **not** need to scatter direct access to all twelve specialized SteamComplete subsystems across gameplay Blueprints.

The intended rule is:

> Use Easy by default. Reach into Systems only for the exact specialized behavior you need.

---

# 20. Debugging checklist

When an Easy node fails:

## Step 1 — inspect FSteamOperationResult

Check:

```text
bSuccess
Code
Message
DebugMessage
NativeCode
```

## Step 2 — check Easy status

```text
Get Steam Complete Easy Status
```

Look at:

- Ready
- Online
- AppId
- multiplayer session state
- voice state
- mod counts

## Step 3 — run setup check

```text
Run Steam Complete Setup Check
```

Read in this order:

1. `Summary`
2. `Errors`
3. `Warnings`
4. `SuggestedFixes`

## Step 4 — only then inspect Systems

If the problem is feature-specific, inspect the corresponding specialized subsystem.

Examples:

- multiplayer → Multiplayer Session / Lobby / Networking
- voice → Voice
- mods → Mods / Workshop
- cloud → Cloud
- progression → Progression
- server → Dedicated Server

---

# 21. Default timeouts

When an Easy async node exposes `TimeoutSeconds`, passing `0` means “use SteamComplete's configured default”.

Current v1.0.3 defaults:

| Area | Default |
|---|---:|
| Short Workshop/API calls | 15 s |
| Workshop download / Install And Load Mod | 300 s |
| Lobby async operations | 25 s |
| Multiplayer host/join | 35 s |
| Session handshake | 10 s |
| Cloud async read/write | 30 s |
| Authentication async | 25 s |
| Inventory async | 30 s |
| Progression async | 20 s |

These values are configurable in:

```text
Project Settings
→ Plugins
→ Steam Complete
```

For most projects, leave the Easy async node timeout at `0` unless you have a specific design reason to override it.

---

# 22. Cheat sheet

## I want to initialize SteamComplete

```text
Get Steam Complete
→ Setup Steam Complete
```

## I want a normal co-op host

```text
Setup(Co-op)
→ Quick Host Steam Game
```

## I want a server list

```text
Quick Find Steam Games
→ display Lobbies
→ Quick Join Steam Game
```

## I want a one-button Quick Play

```text
Find And Join Steam Game
```

If `NotFound`, optionally:

```text
Quick Host Steam Game
```

## I want global voice

```text
Enable Steam Voice Chat
```

## I want proximity voice

```text
Enable Steam Proximity Voice
+ Systems: Set Steam Voice Position regularly
```

## I want push-to-talk

```text
Pressed  → Set Steam Push To Talk(true)
Released → Set Steam Push To Talk(false)
```

## I want to save JSON to Steam Cloud

```text
Quick Save Steam Cloud Text
```

## I want to load JSON

```text
Quick Load Steam Cloud Text
```

## I want to unlock an achievement

```text
Unlock Steam Achievement
```

## I want to update multiple stats efficiently

```text
Set Stat(... StoreImmediately=false)
Set Stat(... StoreImmediately=false)
Unlock Achievement(... StoreImmediately=false)
→ Store Steam Progression
```

## I want to install a Workshop mod

```text
Install And Load Steam Mod
```

## I want to load the user's enabled mods

```text
Load All Enabled Steam Mods
```

## I want to open Workshop

```text
Open Steam Workshop
```

## I want to start a dedicated server

```text
Start Steam Dedicated Server Anonymous
```

or

```text
Start Steam Dedicated Server With GSLT
```

## I want to know why SteamComplete is failing

```text
Run Steam Complete Setup Check
```

---

# Design rule for future SteamComplete versions

The Easy API is intentionally a small facade over the complete Steamworks feature set.

Future features should follow the same rule:

1. add the full Systems/Advanced implementation;
2. identify the common user workflow;
3. expose a small high-level Easy node when a genuine one-click workflow exists;
4. do **not** expose every low-level variation in Easy;
5. keep Easy understandable even as the complete plugin grows.

That means future additions such as Server Browser, Host Migration, Friends/Rich Presence, Steam Input, Game Recording, Remote Play, Parties, Apps/DLC and Utils should integrate into this UX rather than simply increasing the visible node count.

---

**End of SteamComplete Easy API v1.0.3 documentation.**
