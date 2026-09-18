# Yet Another Steam Plugin (YASP) — Primary API Documentation

This document covers **only the recommended simple / Primary API** of **Yet Another Steam Plugin (YASP)**. It intentionally excludes `Legacy API` and `Advanced API`.

**Source-of-truth count:** 184 Blueprint-visible Primary nodes. The count is taken directly from the compiling v1.8.1 YASP headers, including `Is Steam YASP Ready`.

## Mental model

```text
Get Steam YASP
├─ Start Here
├─ Multiplayer
│  ├─ Core
│  ├─ Dedicated Server
│  ├─ Host Migration
│  ├─ Lobby
│  ├─ Messaging
│  ├─ Players
│  ├─ Voice
├─ Social
│  ├─ Core
│  ├─ Overlay
│  ├─ Parties
│  ├─ Remote Play
├─ Progression
│  ├─ Core
│  ├─ Achievements
│  ├─ Leaderboards
│  ├─ Stats
├─ Cloud
│  ├─ Core
├─ Mods
│  ├─ Core
│  ├─ Creator
│  ├─ Social
│  ├─ Workshop
├─ Input
│  ├─ Core
├─ Inventory
│  ├─ Core
│  ├─ Crafting
│  ├─ Definitions
│  ├─ Drops
│  ├─ Promo
│  ├─ Proof
│  ├─ Stacks
│  ├─ Store
├─ Platform
│  ├─ App
│  ├─ DLC
│  ├─ Screenshots
│  ├─ Timeline
│  ├─ Utils
```

The intended Blueprint workflow is: **Get Steam YASP → choose a typed service → drag from that service pin → choose only the nodes relevant to that domain.**

### Node type legend

- **Service** — typed branch returned by `Get Steam YASP`.
- **Pure** — reads state and has no intended side effect.
- **Immediate** — performs an operation and returns immediately with a result/status.
- **Async** — starts an asynchronous Steam workflow and completes through async success/failure outputs.

## Start Here and service accessors

### Start Here

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Last Steam YASP Setup Result` | Pure | Returns the result produced by the most recent setup call. | None |
| `Get Recommended Steam YASP Setup` | Pure | Builds the recommended setup options for a preset without applying them. | Preset: ESteamYASPPreset |
| `Get Steam YASP` | Pure | Returns the main SteamYASP subsystem. This is the single recommended entry point for the Primary API. | WorldContextObject: UObject |
| `Get Steam YASP Status` | Pure | Returns a compact readiness/status snapshot for the simple API. | None |
| `Is Steam YASP Ready` | Pure | Fast readiness check for the Primary API. | None |
| `Setup Steam YASP` | Immediate | Applies the recommended runtime setup for a selected preset such as Coop or Dedicated Server. | Preset: ESteamYASPPreset = ESteamYASPPreset::Coop |
| `Setup Steam YASP Custom` | Immediate | Runs the setup pipeline using explicit setup options instead of a preset. | Options: FSteamYASPSetupOptions |
| `Run Steam YASP Setup Check` | Immediate | Runs the safe health/diagnostic check and returns a detailed report. | None |

### Typed service pins

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Cloud` | Service | Returns the typed Cloud service for Steam Remote Storage / Steam Cloud workflows. | None |
| `Input` | Service | Returns the typed Steam Input service. | None |
| `Inventory` | Service | Returns the typed Steam Inventory service. | None |
| `Mods` | Service | Returns the typed Mods service for Workshop and runtime mod-loader workflows. | None |
| `Multiplayer` | Service | Returns the typed Multiplayer service. Drag from this pin to discover only multiplayer workflows. | None |
| `Platform` | Service | Returns the typed Platform service for app/DLC/utils/screenshots/timeline workflows. | None |
| `Progression` | Service | Returns the typed Progression service for achievements, stats and leaderboards. | None |
| `Social` | Service | Returns the typed Social service for friends, presence, overlay, Remote Play and Parties. | None |

## Multiplayer (45 nodes)

Hosting, matchmaking, lobby/player data, peer messaging, voice, host migration and dedicated servers.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Find Games` | Async | Searches for compatible Steam games using map/game-mode/slot/distance/custom lobby filters. | Config: FSteamYASPGameSearchConfig; TimeoutSeconds: Float = 0.0f |
| `Get Active Game` | Pure | Returns the current high-level SteamYASP multiplayer session/lobby state. | None |
| `Get Recommended Game Config` | Pure | Builds an intent-oriented host configuration from a SteamYASP preset. | Preset: ESteamYASPPreset = ESteamYASPPreset::Coop |
| `Host Game` | Async | Creates and configures a Steam game using one compact Game Config: lobby, metadata, networking, optional voice, host migration and Rich Presence. | Config: FSteamYASPGameConfig; TimeoutSeconds: Float = 0.0f |
| `Is In Game` | Pure | Returns whether SteamYASP currently considers the player to be in an active game. | None |
| `Join Friend` | Async | Attempts to join a friend through their Steam game/lobby presence information. | FriendId: FSteamUserId; TimeoutSeconds: Float = 0.0f |
| `Join Game` | Async | Joins a selected result returned by Find Games. | LobbyId: FSteamLobbyId; TimeoutSeconds: Float = 0.0f |
| `Leave Game` | Immediate | Leaves the active game/session and closes the associated SteamYASP networking state. | bLingerReliableData: Boolean = false |
| `Quick Play` | Async | Searches with a Game Search Config and automatically attempts to join the best matching result. | Config: FSteamYASPGameSearchConfig; TimeoutSeconds: Float = 0.0f |

### Dedicated Server

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Add Server To Favorites` | Immediate | Adds server to favorites using Steam. | Server: FSteamServerBrowserServer |
| `Connect To Dedicated Server` | Immediate | Primary SteamYASP workflow: Connect To Dedicated Server. | Server: FSteamServerBrowserServer |
| `Find And Connect Best Dedicated Server` | Async | Searches the dedicated-server browser and automatically connects to the best matching server. | Options: FSteamYASPQuickServerFindOptions; TimeoutSeconds: Float = 0.0f |
| `Find Dedicated Servers` | Async | Primary SteamYASP workflow: Find Dedicated Servers. | Options: FSteamYASPQuickServerFindOptions; TimeoutSeconds: Float = 0.0f |
| `Get Dedicated Server Status` | Pure | Returns dedicated server status through the Multiplayer service. | None |
| `Remove Server From Favorites` | Immediate | Removes server from favorites using Steam. | Server: FSteamServerBrowserServer |
| `Start Dedicated Server Anonymous` | Immediate | Starts dedicated server anonymous. | Config: FSteamDedicatedServerConfig |
| `Start Dedicated Server With GSLT` | Immediate | Starts/authenticates a dedicated server using a Game Server Login Token. Use only on trusted server builds. | Config: FSteamDedicatedServerConfig; GameServerLoginToken: String |

### Host Migration

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Host Migration Status` | Pure | Returns host migration status through the Multiplayer service. | None |
| `Get Latest Host Migration State Text` | Pure | Returns latest host migration state text through the Multiplayer service. | StateText: String; Revision: Integer |
| `Hand Off Host` | Immediate | Primary SteamYASP workflow: Hand Off Host. | NewHostUserId: FSteamUserId |
| `Update Host Migration State Text` | Immediate | Primary SteamYASP workflow: Update Host Migration State Text. | StateText: String; Revision: Integer; SentPeerCount: Integer |

### Lobby

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Delete Lobby Data` | Immediate | Deletes lobby data. | LobbyId: FSteamLobbyId; Key: String |
| `Get Current Lobby Info` | Immediate | Returns current lobby info through the Multiplayer service. | Lobby: FSteamLobbyInfo |
| `Get Lobby Data` | Immediate | Returns lobby data through the Multiplayer service. | LobbyId: FSteamLobbyId; Key: String; Value: String |
| `Get Lobby Info` | Immediate | Returns lobby info through the Multiplayer service. | LobbyId: FSteamLobbyId; Lobby: FSteamLobbyInfo |
| `Set Lobby Data` | Immediate | Updates lobby data for the current SteamYASP context. | LobbyId: FSteamLobbyId; Key: String; Value: String |

### Messaging

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Broadcast Binary` | Immediate | Broadcasts binary to all connected SteamYASP peers. | Data: Bytes; SendMode: ESteamNetworkSendMode; SentPeerCount: Integer |
| `Broadcast Text` | Immediate | Broadcasts text to all connected SteamYASP peers. | Message: String; SendMode: ESteamNetworkSendMode; SentPeerCount: Integer |
| `Send Binary To Player` | Immediate | Sends binary to player through the current SteamYASP multiplayer transport. | UserId: FSteamUserId; Data: Bytes; SendMode: ESteamNetworkSendMode; MessageNumber: Integer64 |
| `Send Text To Player` | Immediate | Sends text to player through the current SteamYASP multiplayer transport. | UserId: FSteamUserId; Message: String; SendMode: ESteamNetworkSendMode; MessageNumber: Integer64 |

### Players

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Current Lobby Players` | Immediate | Returns enriched player records for the current lobby, combining Steam persona/social state, lobby roles and transport state. | Players: Array<FSteamYASPLobbyPlayer> |
| `Get Lobby Players` | Immediate | Returns enriched player records for a specified lobby. | LobbyId: FSteamLobbyId; Players: Array<FSteamYASPLobbyPlayer> |
| `Get Player Lobby Data` | Immediate | Returns player lobby data through the Multiplayer service. | LobbyId: FSteamLobbyId; UserId: FSteamUserId; Key: String; Value: String |
| `Get Session Peers` | Immediate | Returns the currently known SteamYASP networking peers. | Peers: Array<FSteamMultiplayerPeer> |
| `Invite Player To Lobby` | Immediate | Sends an invitation for player to lobby. | LobbyId: FSteamLobbyId; UserId: FSteamUserId |
| `Set My Lobby Player Data` | Immediate | Updates my lobby player data for the current SteamYASP context. | LobbyId: FSteamLobbyId; Key: String; Value: String |

### Voice

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Disable Voice Chat` | Immediate | Disables voice chat for the current SteamYASP context. | None |
| `Enable Proximity Voice` | Immediate | Enables voice chat with distance attenuation using a compact proximity configuration. | MaxDistance: Float = 2000.0f; FullVolumeDistance: Float = 300.0f; InputMode: ESteamVoiceInputMode = ESteamVoiceInputMode::PushToTalk |
| `Enable Voice Chat` | Immediate | Enables voice chat for the current SteamYASP context. | Config: FSteamVoiceChatConfig |
| `Mute Player` | Immediate | Primary SteamYASP workflow: Mute Player. | UserId: FSteamUserId; bMuted: Boolean |
| `Set Player Voice Volume` | Immediate | Updates player voice volume for the current SteamYASP context. | UserId: FSteamUserId; Volume: Float |
| `Set Push To Talk` | Immediate | Updates push to talk for the current SteamYASP context. | bPressed: Boolean |
| `Set Voice Channel` | Immediate | Updates voice channel for the current SteamYASP context. | Channel: ESteamVoiceChannel |
| `Set Voice Position` | Immediate | Updates voice position for the current SteamYASP context. | Position: FVector |
| `Set Voice Team` | Immediate | Updates voice team for the current SteamYASP context. | TeamId: Integer |

## Social (22 nodes)

Friends/persona data, Rich Presence, Steam Overlay, Remote Play Together and Steam Parties.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Clear Rich Presence` | Immediate | Primary SteamYASP workflow: Clear Rich Presence. | None |
| `Get Friend Rich Presence` | Immediate | Reads a Rich Presence value published by a friend. | FriendId: FSteamUserId; Key: String; Value: String |
| `Get Friends` | Immediate | Returns Steam friend/persona information in a Blueprint-friendly array. | Friends: Array<FSteamFriendInfo>; bIncludeNonFriends: Boolean = false |
| `Get My Persona Name` | Pure | Returns my persona name through the Social service. | None |
| `Get My Steam ID` | Pure | Returns my steam id through the Social service. | None |
| `Get User Avatar` | Immediate | Returns the requested avatar size for a Steam user. | UserId: FSteamUserId; Size: ESteamAvatarSize; Avatar: FSteamAvatarImage |
| `Get User Info` | Immediate | Returns cached Steam persona/social information for a user. | UserId: FSteamUserId; User: FSteamFriendInfo |
| `Invite Friend To Game` | Immediate | Sends a Steam game invite with an optional connection string. | FriendId: FSteamUserId; ConnectString: String |
| `Request User Info` | Immediate | Requests Steam to refresh/cache persona information for a user. | UserId: FSteamUserId; bRequireNameOnly: Boolean = false |
| `Set Rich Presence` | Immediate | Sets the common Rich Presence keys from a compact options struct. | Options: FSteamYASPRichPresenceOptions |

### Overlay

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Open Friend Chat` | Immediate | Opens friend chat through Steam. | FriendId: FSteamUserId |
| `Open Friend Profile` | Immediate | Opens friend profile through Steam. | FriendId: FSteamUserId |
| `Open Friends Overlay` | Immediate | Opens friends overlay through Steam. | None |
| `Open Web Page` | Immediate | Opens web page through Steam. | Url: String; bModal: Boolean = false |

### Parties

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Create Party` | Async | Creates party asynchronously through Steam. | OpenSlots: Integer; Metadata: String; ConnectString: String; TimeoutSeconds: Float |
| `Destroy Party` | Immediate | Primary SteamYASP workflow: Destroy Party. | BeaconId: FSteamPartyBeaconId |
| `Get Parties` | Immediate | Returns parties through the Social service. | Parties: Array<FSteamPartyBeaconInfo> |
| `Join Party` | Async | Primary SteamYASP workflow: Join Party. | BeaconId: FSteamPartyBeaconId; TimeoutSeconds: Float = 20.0f |

### Remote Play

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Create Remote Play Guest Invite` | Async | Creates remote play guest invite asynchronously through Steam. | TimeoutSeconds: Float = 15.0f |
| `Get Remote Play Sessions` | Immediate | Returns remote play sessions through the Social service. | Sessions: Array<FSteamRemotePlaySessionInfo> |
| `Invite Friend To Remote Play Together` | Immediate | Sends an invitation for friend to remote play together. | FriendId: FSteamUserId |
| `Open Remote Play Together` | Immediate | Opens remote play together through Steam. | None |

## Progression (15 nodes)

Achievements, integer/float/average-rate stats and leaderboards.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Store Progression` | Immediate | Flushes pending achievement/stat changes to Steam. | None |

### Achievements

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Achievement` | Pure | Reads one achievement state/details by API name. | AchievementApiName: String; Achievement: FSteamAchievementInfo |
| `Get All Achievements` | Pure | Returns all available achievement states known to the client. | Achievements: Array<FSteamAchievementInfo> |
| `Show Achievement Progress` | Immediate | Displays Steam achievement progress for a partially completed achievement. | AchievementApiName: String; CurrentProgress: Integer; MaxProgress: Integer |
| `Unlock Achievement` | Immediate | Unlocks an achievement by API name and optionally stores immediately. | AchievementApiName: String; bStoreImmediately: Boolean = true |

### Leaderboards

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Find Leaderboard` | Async | Finds a leaderboard by name, optionally creating it when missing. | LeaderboardName: String; bCreateIfMissing: Boolean = false; SortMethod: ESteamLeaderboardSortMethod = ESteamLeaderboardSortMethod::Descending; DisplayType: ESteamLeaderboardDisplayType = ESteamLeaderboardDisplayType::Numeric; TimeoutSeconds: Float = 0.0f |
| `Get Leaderboard Entries` | Async | Downloads a range of leaderboard entries using the selected request mode. | LeaderboardHandle: FSteamLeaderboardHandle; RequestType: ESteamLeaderboardDataRequest = ESteamLeaderboardDataRequest::Global; RangeStart: Integer = 1; RangeEnd: Integer = 10; TimeoutSeconds: Float = 0.0f |
| `Get Leaderboard Entries For Players` | Async | Downloads leaderboard entries for an explicit list of Steam users. | LeaderboardHandle: FSteamLeaderboardHandle; UserIds: Array<FSteamUserId>; TimeoutSeconds: Float = 0.0f |
| `Get Leaderboard Info` | Pure | Returns cached metadata for a leaderboard handle. | LeaderboardHandle: FSteamLeaderboardHandle; Leaderboard: FSteamLeaderboardInfo |
| `Submit Score` | Async | Uploads a score and optional detail values to a leaderboard. | LeaderboardHandle: FSteamLeaderboardHandle; Score: Integer; ScoreDetails: Array<Integer>; UploadMethod: ESteamLeaderboardUploadMethod = ESteamLeaderboardUploadMethod::KeepBest; TimeoutSeconds: Float = 0.0f |

### Stats

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Float Stat` | Pure | Returns float stat through the Progression service. | StatApiName: String; Value: Float |
| `Get Int Stat` | Pure | Returns int stat through the Progression service. | StatApiName: String; Value: Integer |
| `Set Float Stat` | Immediate | Updates float stat for the current SteamYASP context. | StatApiName: String; Value: Float; bStoreImmediately: Boolean = true |
| `Set Int Stat` | Immediate | Updates int stat for the current SteamYASP context. | StatApiName: String; Value: Integer; bStoreImmediately: Boolean = true |
| `Update Average Rate Stat` | Immediate | Primary SteamYASP workflow: Update Average Rate Stat. | StatApiName: String; CountThisSession: Float; SessionLengthSeconds: Float |

## Cloud (11 nodes)

Steam Cloud file status, inspection, text/binary save and load.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Cloud File Exists` | Pure | Primary SteamYASP workflow: Cloud File Exists. | FileName: String; bExists: Boolean |
| `Delete Cloud File` | Immediate | Deletes cloud file. | FileName: String |
| `Get Cloud File Info` | Pure | Returns cloud file info through the Cloud service. | FileName: String; File: FSteamCloudFileInfo |
| `Get Cloud Files` | Pure | Returns cloud files through the Cloud service. | Files: Array<FSteamCloudFileInfo> |
| `Get Cloud Status` | Pure | Returns Steam Cloud availability/usage status. | Status: FSteamCloudStatus |
| `Get Local Cloud Changes` | Pure | Returns local Remote Storage file changes known by Steam. | Changes: Array<FSteamCloudLocalFileChange> |
| `Load Bytes` | Async | Reads arbitrary binary data from Steam Cloud asynchronously. | FileName: String; TimeoutSeconds: Float = 0.0f |
| `Load Text` | Async | Reads a Steam Cloud file and exposes it as text asynchronously. | FileName: String; TimeoutSeconds: Float = 0.0f |
| `Save Bytes` | Async | Writes arbitrary binary data to Steam Cloud asynchronously. | FileName: String; Data: Bytes; TimeoutSeconds: Float = 0.0f |
| `Save Text` | Async | Writes a UTF-8 text payload to Steam Cloud asynchronously. | FileName: String; Text: String; TimeoutSeconds: Float = 0.0f |
| `Set Cloud Enabled` | Immediate | Updates cloud enabled for the current SteamYASP context. | bEnabled: Boolean |

## Mods (29 nodes)

Steam Workshop consumer/creator flows plus SteamYASP mod discovery, enable/disable and mounting.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Disable Mod` | Immediate | Disables mod for the current SteamYASP context. | ModId: String; bApplyImmediately: Boolean = true |
| `Download Mod` | Async | Downloads mod asynchronously. | ItemId: FSteamWorkshopItemId; bHighPriority: Boolean = false; TimeoutSeconds: Float = 0.0f |
| `Enable Mod` | Immediate | Enables mod for the current SteamYASP context. | ModId: String; bApplyImmediately: Boolean = true |
| `Get Mod` | Pure | Returns mod through the Mods service. | ModId: String; Mod: FSteamModDescriptor |
| `Get Mod Details` | Async | Downloads detailed Workshop metadata for one item, optionally including dependencies/metadata. | ItemId: FSteamWorkshopItemId; bIncludeDependencies: Boolean = true; bIncludeMetadata: Boolean = true; TimeoutSeconds: Float = 0.0f |
| `Get Mods` | Pure | Returns mods through the Mods service. | None |
| `Install Mod` | Async | One-click consumer workflow that prepares/downloads and loads a Workshop mod. | ItemId: FSteamWorkshopItemId; bHighPriority: Boolean = true; TimeoutSeconds: Float = 0.0f |
| `Load Enabled Mods` | Immediate | Mounts/loads all mods currently enabled in SteamYASP. | MountedMods: Integer |
| `Mount Mod` | Immediate | Primary SteamYASP workflow: Mount Mod. | ModId: String |
| `Prepare Mod` | Async | Ensures a Workshop item is subscribed/downloaded and prepared for use without necessarily performing the full install/load workflow. | ItemId: FSteamWorkshopItemId; bHighPriority: Boolean = false; TimeoutSeconds: Float = 0.0f |
| `Refresh Mods` | Immediate | Refreshes mods from the current runtime state. | DiscoveredMods: Integer |
| `Search Mods` | Async | Runs a Workshop query using the high-level search options struct. | Options: FSteamWorkshopSearchOptions; TimeoutSeconds: Float = 0.0f |
| `Subscribe Mod` | Async | Subscribes the current user to mod. | ItemId: FSteamWorkshopItemId; TimeoutSeconds: Float = 0.0f |
| `Unmount Mod` | Immediate | Primary SteamYASP workflow: Unmount Mod. | ModId: String |
| `Unsubscribe Mod` | Async | Unsubscribes the current user from mod. | ItemId: FSteamWorkshopItemId; TimeoutSeconds: Float = 0.0f |
| `Validate Enabled Mods` | Pure | Validates the enabled mod set and reports conflicts or problems. | None |

### Creator

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Create Mod Item` | Async | Creates a new Workshop item for the app. | bOpenLegalAgreementIfRequired: Boolean = true; TimeoutSeconds: Float = 0.0f |
| `Get Workshop Legal Agreement Status` | Async | Checks whether the user must accept the Steam Workshop legal agreement before publishing. | TimeoutSeconds: Float = 0.0f |
| `Publish Mod` | Async | Publishes a Workshop item from a publish/update configuration. | Request: FSteamWorkshopPublishRequest; TimeoutSeconds: Float = 0.0f |
| `Update Mod` | Async | Updates an existing Workshop item. | ItemId: FSteamWorkshopItemId; Request: FSteamWorkshopUpdateRequest; TimeoutSeconds: Float = 0.0f |

### Social

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get My Mod Vote` | Async | Returns my mod vote through the Mods service. | ItemId: FSteamWorkshopItemId; TimeoutSeconds: Float = 0.0f |
| `Set Mod Favorite` | Async | Updates mod favorite for the current SteamYASP context. | ItemId: FSteamWorkshopItemId; bFavorite: Boolean; TimeoutSeconds: Float = 0.0f |
| `Vote On Mod` | Async | Sets the current user’s Workshop vote for a mod. | ItemId: FSteamWorkshopItemId; Vote: ESteamWorkshopVoteChoice; TimeoutSeconds: Float = 0.0f |

### Workshop

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get Subscribed Workshop Items` | Immediate | Returns subscribed workshop items through the Mods service. | Items: Array<FSteamWorkshopItemId> |
| `Get Workshop Download Progress` | Immediate | Returns workshop download progress through the Mods service. | ItemId: FSteamWorkshopItemId; Info: FSteamWorkshopDownloadInfo |
| `Get Workshop Install Info` | Immediate | Returns workshop install info through the Mods service. | ItemId: FSteamWorkshopItemId; Info: FSteamWorkshopInstallInfo |
| `Get Workshop Item State` | Immediate | Returns workshop item state through the Mods service. | ItemId: FSteamWorkshopItemId; State: FSteamWorkshopItemState |
| `Open Workshop` | Immediate | Opens workshop through Steam. | None |
| `Open Workshop Item` | Immediate | Opens workshop item through Steam. | ItemId: FSteamWorkshopItemId |

## Input (9 nodes)

Steam Input controllers, actions, prompts, haptics and binding UI.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Activate Action Set` | Immediate | Primary SteamYASP workflow: Activate Action Set. | ActionSetName: String; Controller: FSteamInputControllerHandle; bAllControllers: Boolean = true |
| `Get Action Prompt` | Immediate | Returns the appropriate controller glyph/prompt for an action. | Controller: FSteamInputControllerHandle; ActionSetName: String; ActionName: String; bAnalogAction: Boolean; Prompt: FSteamInputPrompt |
| `Get Analog Action` | Immediate | Returns analog action through the Input service. | Controller: FSteamInputControllerHandle; ActionName: String; Data: FSteamInputAnalogActionData |
| `Get Controllers` | Immediate | Returns controllers through the Input service. | Controllers: Array<FSteamInputControllerInfo> |
| `Get Digital Action` | Immediate | Returns digital action through the Input service. | Controller: FSteamInputControllerHandle; ActionName: String; Data: FSteamInputDigitalActionData |
| `Get Primary Controller` | Immediate | Returns primary controller through the Input service. | Controller: FSteamInputControllerInfo |
| `Initialize Steam Input` | Immediate | Initializes the Steam Input API for the current game. | None |
| `Open Controller Bindings` | Immediate | Opens the Steam controller binding UI for a controller. | Controller: FSteamInputControllerHandle |
| `Rumble Controller` | Immediate | Triggers controller haptics with independent left/right intensity. | Controller: FSteamInputControllerHandle; LeftIntensity: Float = 1.0f; RightIntensity: Float = 1.0f |

## Inventory (16 nodes)

Steam Inventory definitions, items, mutations, crafting, drops, proofs and store purchase flows.

### Core

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Consume Item` | Async | Consumes a quantity from an inventory item instance. | ItemInstanceId: FSteamItemInstanceId; Quantity: Integer = 1; TimeoutSeconds: Float = 0.0f |
| `Get All Items` | Async | Retrieves the player’s full Steam Inventory asynchronously. | TimeoutSeconds: Float = 0.0f |
| `Get Item Definition IDs` | Pure | Returns item definition ids through the Inventory service. | DefinitionIds: Array<Integer> |
| `Get Item Definition Property` | Pure | Returns item definition property through the Inventory service. | DefinitionId: Integer; PropertyName: String; Value: String |
| `Get Item Definition Property Names` | Pure | Returns item definition property names through the Inventory service. | DefinitionId: Integer; PropertyNames: Array<String> |
| `Get Items By ID` | Async | Retrieves specific inventory instances by item instance ID. | ItemInstanceIds: Array<FSteamItemInstanceId>; TimeoutSeconds: Float = 0.0f |

### Crafting

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Craft Item` | Async | Generates an item definition by consuming specified item quantities. | GenerateDefinitionId: Integer; ConsumeItems: Array<FSteamInventoryInstanceQuantity>; TimeoutSeconds: Float = 0.0f |

### Definitions

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Load Item Definitions` | Async | Loads/refreshes Steam Inventory item definitions. | TimeoutSeconds: Float = 0.0f |

### Drops

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Trigger Item Drop` | Async | Requests a playtime-generator item drop. | PlaytimeGeneratorDefinitionId: Integer; TimeoutSeconds: Float = 0.0f |

### Promo

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Grant Promo Items` | Async | Requests eligible promotional items for the current user. | TimeoutSeconds: Float = 0.0f |

### Proof

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Create Item Proof` | Async | Serializes selected inventory items into a signed/portable ownership proof. | ItemInstanceIds: Array<FSteamItemInstanceId>; TimeoutSeconds: Float = 0.0f |
| `Verify Item Proof` | Async | Deserializes and verifies an inventory proof against the expected owner. | SerializedProof: Bytes; ExpectedOwnerUserId: FSteamUserId; TimeoutSeconds: Float = 0.0f |

### Stacks

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Move Item Quantity` | Async | Primary SteamYASP workflow: Move Item Quantity. | SourceItem: FSteamItemInstanceId; Quantity: Integer; DestinationItem: FSteamItemInstanceId; TimeoutSeconds: Float = 0.0f |
| `Split Item Stack` | Async | Primary SteamYASP workflow: Split Item Stack. | SourceItem: FSteamItemInstanceId; Quantity: Integer; TimeoutSeconds: Float = 0.0f |

### Store

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Buy Items` | Async | Starts a Steam Inventory purchase for the requested definition quantities. | Cart: Array<FSteamInventoryDefinitionQuantity>; TimeoutSeconds: Float = 0.0f |
| `Get Item Prices` | Async | Requests current item prices/currency information from Steam Inventory. | TimeoutSeconds: Float = 0.0f |

## Platform (21 nodes)

App/DLC utilities, environment/text input, screenshots and Steam Timeline.

### App

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get App Info` | Immediate | Returns common app/runtime information for the current Steam app. | Info: FSteamAppInfo |
| `Get File Details` | Async | Asynchronously gets Steam file details for a local file. | Filename: String; TimeoutSeconds: Float = 20.0f |
| `Get Launch Command Line` | Pure | Returns launch command line through the Platform service. | None |
| `Get Steam App ID` | Pure | Returns the current Steam App ID. | None |

### DLC

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Get DLC Download Progress` | Immediate | Returns current downloaded/total byte counts and percentage for a DLC. | DlcAppId: Integer64; BytesDownloaded: Integer64; BytesTotal: Integer64; Percent: Float |
| `Get DLCs` | Immediate | Enumerates DLC metadata exposed by Steam for the app. | Dlcs: Array<FSteamDlcInfo> |
| `Has DLC` | Pure | Checks whether the current user owns the specified DLC App ID. | DlcAppId: Integer64 |
| `Install DLC` | Immediate | Requests Steam to install an owned DLC. | DlcAppId: Integer64 |
| `Is DLC Installed` | Pure | Checks whether the specified owned DLC is installed. | DlcAppId: Integer64 |
| `Uninstall DLC` | Immediate | Requests Steam to uninstall a DLC. | DlcAppId: Integer64 |

### Screenshots

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Add Screenshot File` | Immediate | Registers an existing image file with Steam Screenshots. | Filename: String; Width: Integer; Height: Integer; Location: String; Screenshot: FSteamScreenshotHandle |
| `Take Screenshot` | Immediate | Triggers a Steam screenshot capture. | None |

### Timeline

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `End Timeline Moment` | Immediate | Ends a previously started Timeline event. | Event: FSteamTimelineEventHandle |
| `End Timeline Phase` | Immediate | Ends the active Timeline phase. | None |
| `Is Timeline Supported` | Pure | Checks whether Steam Timeline is available in the current runtime. | None |
| `Mark Timeline Moment` | Immediate | Adds an instantaneous Steam Timeline event. | Title: String; Description: String; Icon: String; Priority: Integer; ClipPriority: ESteamTimelineClipPriority; Event: FSteamTimelineEventHandle |
| `Start Timeline Moment` | Immediate | Starts a duration-based Steam Timeline event and returns a handle. | Title: String; Description: String; Icon: String; Priority: Integer; ClipPriority: ESteamTimelineClipPriority; Event: FSteamTimelineEventHandle |
| `Start Timeline Phase` | Immediate | Starts a named Timeline phase/game mode. | PhaseId: String; GameMode: ESteamTimelineGameMode = ESteamTimelineGameMode::Playing |

### Utils

| Node | Type | What it does | Main pins |
|---|---|---|---|
| `Filter Text` | Immediate | Runs Steam text filtering for user-generated text. | Context: ESteamYASPTextFilterContext; Input: String; SourceUser: FSteamUserId; FilteredText: String; FilteredCharacterCount: Integer |
| `Get Steam Environment` | Immediate | Returns a snapshot of useful Steam client/environment information. | Snapshot: FSteamUtilsSnapshot |
| `Show Gamepad Text Input` | Immediate | Opens Steam’s gamepad-friendly text input UI. | Description: String; MaxCharacters: Integer; bMultiline: Boolean; ExistingText: String |

## Key Primary API structs

### Steam YASP Game Config
`FSteamYASPGameConfig` is the main Host Game config. It contains Lobby Name, Privacy, Max Players, Joinable, Host Migration, Map, Game Mode, custom Metadata, Voice settings, Rich Presence settings and an advanced NetworkingSockets Virtual Port.

### Steam YASP Game Search Config
`FSteamYASPGameSearchConfig` is used by Find Games and Quick Play. It supports Map, Game Mode, Minimum Open Slots, Steam lobby Distance, Max Results, string filters and numerical filters.

### Steam YASP Lobby Player
`FSteamYASPLobbyPlayer` is the recommended lobby/player UI struct. It combines Steam User ID, Persona Name, local/lobby-owner/session-host roles, transport connected/handshake state, friend relationship, persona state, online state and current Steam game information.

## Important behavior notes

- A **DLC product is created/configured in Steamworks Partner**, not from a shipped runtime node. The Primary API handles runtime enumeration, ownership, installed state, install/uninstall and download progress after the DLC App ID exists.
- `Start Dedicated Server With GSLT` should be used only on trusted server builds; do not ship a reusable GSLT in a client build.
- Workshop publishing can require the user to accept the Steam Workshop legal agreement. Check `Get Workshop Legal Agreement Status` and/or allow `Create Mod Item` to open it when required.
- Inventory item definitions, achievement API names, stats, DLC App IDs and most other Steam content identifiers must exist in Steamworks configuration before normal runtime use.

## Usage examples

### 1. Initialize SteamYASP once

`Get Steam YASP` → `Setup Steam YASP (Coop)` → `Is Steam YASP Ready` → optional `Run Steam YASP Setup Check` if the status is not ready.

### 2. Host a 4-player coop lobby

`Get Steam YASP` → `Multiplayer` → `Get Recommended Game Config (Coop)` → set Map / Game Mode / Max Players / Voice if needed → `Host Game`.

### 3. Build a server browser

`Multiplayer` → `Find Games` with a `Game Search Config` → populate UI from results → user selects one → `Join Game`.

### 4. One-click matchmaking

`Multiplayer` → `Quick Play` with Map/GameMode/open-slot filters. SteamYASP searches and joins the best compatible result.

### 5. Show lobby members with Steam info

`Multiplayer` → `Get Current Lobby Players` → for each `Steam YASP Lobby Player`, display Persona Name / role / online state. Use `Social → Get User Avatar` when an avatar texture is needed.

### 6. Invite a friend and expose Join Game

`Social` → `Set Rich Presence` → `Get Friends` → choose friend → `Invite Friend To Game`. On the other client, `Multiplayer → Join Friend` can use the Steam presence/lobby information.

### 7. Proximity voice

After Host/Join succeeds: `Multiplayer → Enable Proximity Voice` → every update, call `Set Voice Position` with the local player position → use `Mute Player` / `Set Player Voice Volume` for UI controls.

### 8. Achievement + stat + leaderboard

`Progression → Unlock Achievement` and/or `Set Int Stat` / `Set Float Stat` → `Store Progression` when batching changes. For scores: `Find Leaderboard` → `Submit Score` → `Get Leaderboard Entries`.

### 9. Steam Cloud save

`Cloud → Save Text` for JSON/text saves or `Save Bytes` for binary saves. To load: `Cloud → Cloud File Exists` → `Load Text` / `Load Bytes`.

### 10. Workshop mod browser and install

`Mods → Search Mods` → display results → `Get Mod Details` when opening a details page → `Install Mod` for the one-click download/prepare/load workflow → `Get Mods` to refresh the local mod UI.

### 11. Publish a Workshop mod

`Mods → Get Workshop Legal Agreement Status` → `Create Mod Item` → `Publish Mod`. Later edits use `Update Mod`; user-facing voting/favorites use `Vote On Mod` and `Set Mod Favorite`.

### 12. Steam Inventory shop/crafting

`Inventory → Load Item Definitions` → `Get All Items`. Shop: `Get Item Prices` → `Buy Items`. Crafting: `Craft Item`; consumables: `Consume Item`; stacks: `Split Item Stack` / `Move Item Quantity`.

### 13. DLC-gated content

`Platform → Has DLC (DlcAppId)` decides entitlement. If owned but not installed, call `Install DLC`, show `Get DLC Download Progress`, then confirm with `Is DLC Installed` before enabling local content.

### 14. Dedicated server browser

`Multiplayer → Find Dedicated Servers` → display server list → `Connect To Dedicated Server`, or use `Find And Connect Best Dedicated Server` for a one-click path.

### 15. Dedicated server startup

On a server build: `Multiplayer → Start Dedicated Server Anonymous` for anonymous auth or `Start Dedicated Server With GSLT` for token auth → inspect `Get Dedicated Server Status`.

### 16. Steam Input UI

`Input → Initialize Steam Input` → `Get Primary Controller` → `Activate Action Set` → query gameplay with `Get Digital Action` / `Get Analog Action`; UI can call `Get Action Prompt` to show the correct Steam glyph.

### 17. Remote Play Together

`Social → Open Remote Play Together` or `Invite Friend To Remote Play Together`. Use `Get Remote Play Sessions` to inspect active sessions; `Create Remote Play Guest Invite` can create a guest invitation flow.

### 18. Timeline + screenshots

Check `Platform → Is Timeline Supported`. Use `Mark Timeline Moment` for instant events or `Start Timeline Moment` / `End Timeline Moment` for durations. `Take Screenshot` integrates captures with Steam Screenshots.

## Discovery rule: when to leave the Primary API

For ordinary game-facing Steam features, stay in the Primary API. Use `Legacy API` only for compatibility with older project Blueprints, and use `Advanced API` only when you deliberately need a low-level Steamworks primitive or a specialized feature that is not exposed as a high-level workflow.

---
Generated from the Yet Another Steam Plugin v1.8.1 source headers, based on the v1.8.0 codebase validated in UE 5.6–5.8.