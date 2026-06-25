# LuaTools

* **Version:** 8.0.5
* **Backend:** Lua (Millennium)
* **Frontend:** JavaScript (Injected into Steam UI)

## Technical Description
LuaTools is a plugin for Millennium that integrates a seamless system for downloading and installing game fixes (patches) directly into the Steam Store UI. It acts as a bridge between the user and multiple remote download sources (APIs), automating the retrieval, extraction, and installation of files right into the game's directory.

The Lua backend exposes functions via RPC (Millennium.callServerMethod), which the frontend (JS) consumes to manage:

* **Fix Search & Download:** Fetching patches from configurable APIs (Morrenus, custom, etc.).
* **Availability Verification:** Checking host status via HEAD/GET requests with built-in timeouts.
* **Asynchronous Extraction:** Running extraction tasks in the background (using native tar/curl) without blocking the Steam UI.
* **Script & Manifest Installation:** Deploying Lua scripts to steam/config/stplug-in/ and manifests to depotcache/.
* **Download State Management:** Real-time tracking of download progress, handling errors, and allowing cancellations.
* **Auto-Updates:** Automatically updating the plugin itself via a remote manifest file.
* **Key Donation:** Voluntarily contributing decryption keys extracted from config.vdf to improve the community database.

## Main Modules

| File | Function |
| :--- | :--- |
| `main.lua` | Entry point; registers RPC functions, injects webkit resources, and orchestrates overall initialization. |
| `downloads.lua` | Handles asynchronous ZIP downloading, extraction, and installation in the game folder. Supports multiple APIs and states. |
| `fixes.lua` | Queries the remote fix index (generic/online) and handles the application of downloads. |
| `api_manifest.lua` | Manages the API source list (active/disabled states, priority ordering, renaming, and custom API creation)[cite: 2]. |
| `settings.manager.lua` | Handles configuration persistence (theme, language, Morrenus key, fast download, etc.)[cite: 2]. |
| `donate_keys.lua` | Extracts appid:DecryptionKey pairs from config.vdf and sends them to a central server (voluntary donation)[cite: 2]. |
| `auto_update.lua` | Checks for plugin updates and automatically applies pending patches[cite: 2]. |
| `http_client.lua` | A custom wrapper for HTTP requests featuring timeout handling and custom headers[cite: 2]. |

## Typical Workflow
1. The user navigates to any game's page within the Steam client[cite: 2].
2. The frontend injects two functional buttons into the interface: "Add via LuaTools" and "Restart Steam"[cite: 2].
3. Upon clicking "Add", the backend orchestrates the following[cite: 2]:
   * It queries all active APIs (sorted by priority) to verify if the specific appid has a fix available via HEAD/GET requests[cite: 2].
   * If multiple sources host the fix, it prompts a selection UI (or skips this and automatically picks the top source if "fast download" is enabled)[cite: 2].
   * It spawns an asynchronous background download process (using a hidden CMD window on Windows, or nohup on Linux)[cite: 2].
4. The frontend continuously polls the download status (GetAddViaLuaToolsStatus) to update the visual progress bar[cite: 2].
5. Once finished, the backend script extracts the ZIP archive directly into the game folder, copies the .lua script to stplug-in/, and shifts manifests to depotcache/[cite: 2].
6. If the applied fix is a Lua script, the setManifestid() line is automatically commented out to prevent manifest overwriting conflicts[cite: 2].
7. The user receives a detailed notification summarizing the added assets (including workshop content and included or missing DLCs)[cite: 2].

## APIs and Download Sources
All source APIs are stored locally in api.json (backend) and can be modified directly from the frontend configuration panel[cite: 2].

Each configured API contains[cite: 2]:
* `name`: A unique identifier[cite: 2].
* `url`: A URL template supporting and optional tokens like or (specifically for Morrenus)[cite: 2].
* `success_code`: The expected HTTP success status code (e.g., 200)[cite: 2].
* `enabled`: A boolean toggle flag[cite: 2].

The backend validates each endpoint using a fast HEAD request (falling back to GET if HEAD isn't supported) with a strict 5-second timeout[cite: 2]. APIs can be dynamically reordered, renamed, toggled, or deleted straight from the interface[cite: 2].

## Persistent Configuration
User settings are saved in data/settings.json (backend) and exposed to the user interface via GetSettingsConfig[cite: 2]. Configurable options include[cite: 2]:
* `language`: The interface display language (defaults to Steam's active language)[cite: 2].
* `theme`: The visual styling theme (Original, Light, etc.)[cite: 2].
* `fastDownload`: When enabled, bypasses the source selection prompt and automatically pulls from the highest priority available API[cite: 2].
* `morrenusApiKey`: Authentication key for the Morrenus API (validated against hubcapmanifest.com)[cite: 2].
* `useSteamLanguage`: Dynamically synchronizes the plugin's UI language directly with Steam's environment[cite: 2].

## Frontend ↔ Backend Communication
The frontend (JS) invokes backend procedures utilizing Millennium's bridge protocol[cite: 2]:

```javascript
Millennium.callServerMethod("luatools", "FunctionName", { ...args })
