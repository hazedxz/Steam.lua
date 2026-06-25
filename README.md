Version: 8.0.5

Backend: Lua (Millennium)

Frontend: JavaScript (Injected into Steam UI)

Technical Description
LuaTools is a plugin for Millennium that integrates a seamless system for downloading and installing game fixes (patches) directly into the Steam Store UI. It acts as a bridge between the user and multiple remote download sources (APIs), automating the retrieval, extraction, and installation of files right into the game's directory.

The Lua backend exposes functions via RPC (Millennium.callServerMethod), which the frontend (JS) consumes to manage:

Fix Search & Download: Fetching patches from configurable APIs (Morrenus, custom, etc.).

Availability Verification: Checking host status via HEAD/GET requests with built-in timeouts.

Asynchronous Extraction: Running extraction tasks in the background (using native tar/curl) without blocking the Steam UI.

Script & Manifest Installation: Deploying Lua scripts to steam/config/stplug-in/ and manifests to depotcache/.

Download State Management: Real-time tracking of download progress, handling errors, and allowing cancellations.

Auto-Updates: Automatically updating the plugin itself via a remote manifest file.

Key Donation: Voluntarily contributing decryption keys extracted from config.vdf to improve the community database.

Main Modules
File	Function
main.lua	Entry point; registers RPC functions, injects webkit resources, and orchestrates overall initialization.
downloads.lua	Handles asynchronous ZIP downloading, extraction, and installation in the game folder. Supports multiple APIs and states.
fixes.lua	Queries the remote fix index (generic/online) and handles the application of downloads.
api_manifest.lua	Manages the API source list (active/disabled states, priority ordering, renaming, and custom API creation).
settings.manager.lua	Handles configuration persistence (theme, language, Morrenus key, fast download, etc.).
donate_keys.lua	Extracts appid:DecryptionKey pairs from config.vdf and sends them to a central server (voluntary donation).
auto_update.lua	Checks for plugin updates and automatically applies pending patches.
http_client.lua	A custom wrapper for HTTP requests featuring timeout handling and custom headers.
Typical Workflow
The user navigates to any game's page within the Steam client.

The frontend injects two functional buttons into the interface: "Add via LuaTools" and "Restart Steam".

Upon clicking "Add", the backend orchestrates the following:

It queries all active APIs (sorted by priority) to verify if the specific appid has a fix available via HEAD/GET requests.

If multiple sources host the fix, it prompts a selection UI (or skips this and automatically picks the top source if "fast download" is enabled).

It spawns an asynchronous background download process (using a hidden CMD window on Windows, or nohup on Linux).

The frontend continuously polls the download status (GetAddViaLuaToolsStatus) to update the visual progress bar.

Once finished, the backend script extracts the ZIP archive directly into the game folder, copies the .lua script to stplug-in/, and shifts manifests to depotcache/.

If the applied fix is a Lua script, the setManifestid() line is automatically commented out to prevent manifest overwriting conflicts.

The user receives a detailed notification summarizing the added assets (including workshop content and included or missing DLCs).

APIs and Download Sources
All source APIs are stored locally in api.json (backend) and can be modified directly from the frontend configuration panel.

Each configured API contains:

name: A unique identifier.

url: A URL template supporting <appid> and optional tokens like <apikey> or <moapikey> (specifically for Morrenus).

success_code: The expected HTTP success status code (e.g., 200).

enabled: A boolean toggle flag.

The backend validates each endpoint using a fast HEAD request (falling back to GET if HEAD isn't supported) with a strict 5-second timeout. APIs can be dynamically reordered, renamed, toggled, or deleted straight from the interface.

Persistent Configuration
User settings are saved in data/settings.json (backend) and exposed to the user interface via GetSettingsConfig. Configurable options include:

language: The interface display language (defaults to Steam's active language).

theme: The visual styling theme (Original, Light, etc.).

fastDownload: When enabled, bypasses the source selection prompt and automatically pulls from the highest priority available API.

morrenusApiKey: Authentication key for the Morrenus API (validated against hubcapmanifest.com).

useSteamLanguage: Dynamically synchronizes the plugin's UI language directly with Steam's environment.

Frontend ↔ Backend Communication
The frontend (JS) invokes backend procedures utilizing Millennium's bridge protocol:

JavaScript
Millennium.callServerMethod("luatools", "FunctionName", { ...args })
Mirroring the architecture of the original Python codebase, all exposed functions return structured JSON payloads encoded via cjson.encode.

Key Exposed Functions Examples
GetApiList, AddCustomApi, ToggleApi, RemoveApi, ReorderApis

StartAddViaLuaTools, GetAddViaLuaToolsStatus, CheckApisForApp

CheckForFixes, ApplyGameFix, GetApplyFixStatus

GetSettingsConfig, ApplySettingsChanges, GetTranslations

GetInstalledFixes, GetInstalledLuaScripts, DeleteLuaToolsForApp

Dependencies
Millennium: A version featuring native support for custom Lua modules and RPC execution.

Lua Libraries: json, fs, http, utils, datetime, steam_utils, etc. (all inherently provided by the Millennium environment).

External Tools: curl, tar (Windows execution safely leverages the native curl.exe and tar.exe binaries already shipped with Steam).

Implementation Notes
All downloads run on entirely separate threads/processes to prevent freezing the Steam client UI or blocking Millennium's master routine thread.

Active download tracking and current tasks write their statuses to temporary manifest files in temp_download/.

Archive extraction utilizes native tar -xf commands across both Windows and Linux to avoid requiring custom unpacking dependencies.

Installed Lua scripts automatically have their setManifestid() lines stripped/commented out to safely mitigate accidental manifest overwrites during Steam integrity checks.

Community decryption key donation triggers strictly once per unique appid and caches the state locally, expiring automatically after 7 days.

Useful Links
Discord: https://discord.gg/luatools

Repository: (link)

Millennium Documentation: https://github.com/SteamClientHomebrew/Millennium
