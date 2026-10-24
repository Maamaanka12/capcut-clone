# Phase 2 — CapCut Config, UI, and Native Surface Evidence

**Date:** 2026-10-23
**Analyst:** REA 6.5.0 via CLI fallback (MCP server unavailable in session)
**Target:** CapCut Desktop 9.6.0.4097

---

## 1. Evidence Index

| Evidence ID | Artifact | Operation | Status |
|-------------|----------|-----------|--------|
| `ev_7ccf1cd...660a` | `9.6.0.4097\CapCut.exe` (90 KB) | `inspect_pe_resources` | observed |
| `ev_4bcb266...b6d3` | `9.6.0.4097\VECreator.dll` (271 MB) | `inspect_pe_resources` | observed |
| `ev_7c4c13c...f64df3` | `Apps\CapCut.exe` (4.9 MB) | `inspect_pe_resources` | observed |
| `ev_7989114...d9b0e` | `Resources\lumi_js_resources` | `analyze_javascript_application` | observed |
| `ev_4ae3abe...9ccb2` | `Resources\lumi_js_resources_video` | `analyze_javascript_application` | observed |
| `ev_554cfb...e701` | `Resources\lynx_config` | `analyze_javascript_application` | observed |
| `ev_fb1983c...d4dad` | `Cache\GeckoCpp\...\image_lynx_videocutpc_modal\1155441196` | `analyze_javascript_application` | observed (binary Lynx bundles) |
| `ev_76d6034...bdaa` | `videoeditor_addon.node` (3.6 MB) | `inspect_artifact` | observed |
| `ev_0acb321...e35cc9` | `agent_runtime_addon.node` (521 KB) | `inspect_artifact` | observed |
| `ev_7c23c1e...f12ae` | `ttnet_frontier_addon.node` (132 KB) | `inspect_artifact` | observed |

All Evidence files saved under `research\ev_*.json`.

---

## 2. User Data & Config Evidence (Observation)

### 2.1 CapCut `User Data` Inventory
- **Total:** 2,351 files / 115 MB
- **Projects\** — **empty** (no saved drafts/projects)
- **Config\** — 50 KB of readable JSON/INI (high value)
- **MMKV\settings_json** — 8 MB binary KV store (format unknown, not JSON despite name)
- **Cache\** — dominates size: ML models, LUTs, Lynx page bundles, CEF caches
- **Tracking\tracking_v4.db**, `Cache\ressdk_db\*\rp.db`, `Cache\ILASDKDB\scan.db` — telemetry/resource DBs (low value)

### 2.2 Key Config Files (Observation)

#### `globalSetting` (INI, 164 lines)
- Render/engine: `qt6RenderEngine=d3d11`, `hardwareEncodeEnable=true`, `hardwareDecodeEnable=true`, `hardwareRenderEnable=true`
- Paths: `screenRecordLocation`, `prerenderCacheLocation`, `remuxFileLocation`, `currentCustomDraftPath` → `User Data\Projects\com.lveditor.draft`
- Quality/performance: `qualityModel=1`, `performanceQualityMode=0`, `fpsNumerator=30`, `fpsDenominator=1`
- Feature flags: `autoPrerenderEnable=false`, `enableAudioPreviewDefaultFalse=false`, `linkageEnableTypes=text,effect,sticker,filter,adjust,sound,tts`
- Qt6 tuning: `ui_memory_optimize.*`, `qsg_memory_optimize.*`, `pad_memory_optimizations.*`, `qt_performance_monitor_config.*`

#### `getCfg` (JSON)
- **block_config.modules** — 3 AI modules blocked: `img2img_controlnet_magicmixReverie`, `img2img_anime_async`, `img2img_idstyle_felt_jy`
- **block_config.play** — 175+ AI features blocked (e.g., `veo_3`, `seedance_2.5`, `runway_gen_4`, `img2img_anime_cc@jy_beauty1`, `face_swap_morphing@*`, `external_i2i_seedream_4_*`, etc.)
- **replicate_config.enable=true** — server-side feature replication enabled

#### `EnvDetect.json` (GPU/OS detection)
- GPU: Intel UHD Graphics 620 (vendor 8086, device 5917, driver 31.0.101.2121)
- OS: Windows 10.0.26300
- Checks: Intel 11th-gen discrete, graphics driver update, GPU bug list (Intel 8086_5917 → no decode/encode disabled), Qt6 render env (d3d11, hw_render=1, hw_enable=1), NVIDIA G-Sync (off), OpenGL version check skipped

#### `Shortcut` JSONs (Command/Action Model) — **Highest Value for Clone**
| Preset | `exportVideo` | `cutoff` | `togglePlay` | `switchToCut` | `toggleFullscreen` | `linkage` | `preview` |
|--------|---------------|----------|--------------|---------------|--------------------|-----------|-----------|
| Final Cut Pro X | Ctrl+E | Ctrl+B | Space | B | Ctrl+Shift+F | ` ` / `·` | S |
| Premiere Pro | Ctrl+M | Ctrl+K | Space | C | ` ` / `·` | Ctrl+L | Shift+P |
| Custom1/2/3 | Ctrl+M | Ctrl+K | Space | C | ` ` / `·` | Ctrl+L | Shift+P |

All presets share ~240 action→keybinding mappings covering: timeline nav, segment ops, keyframes, zoom, audio, subtitles, markers, speed control, AI voice, export, fullscreen, etc. **Directly reusable for clone's keymap system.**

#### `ve_ab_conf.ini` (A/B Test Flags, 115 entries)
Key flags: `ve_enable_svtav1_encode=true`, `ve_enable_cuda_10=true`, `ve_enable_win_bytevc0_enc=true`, `ve_enable_gaussian_blur_opt=720`, `ve_abtest_smart_codec_version=2`, `ve_abtest_hevc_seek_optimization=false`, `ve_enable_bytevc1_decoder=true`, `ve_enable_win_gpu_performance=false`, `ve_enable_svtav1_encode=true`, `ve_enable_intel_best_speed_preset=true`, `ve_abtest_cuda_to_angle=true`, `veabtest_d3d11va_to_angle11=true`, `vesdk_enable_reader_refactor=true`

#### `ve_hw_check.ini` (HW Capability Report)
- GPU 0: `mobilecv2: 1.8.27`
- GPU 1: `bytenn espresso version: 3.11.20`
- GPU 2: Intel UHD Graphics 620, driver 31.0.101.2121, **OK**
- DX11 codec support: H.264 (1920x1080), HEVC (4096x4096), VP9 (4096x4096), AV1 (8192x8192), AV1 10-bit (8192x8192)
- QSV: H.264 (4096x4096), HEVC (3840x2160), VP9 (3840x2160), AV1 (8192x8192)
- New QSV enc: AV1, HEVC alpha, HEVC 4:2:2 — **all failed** (`ttdaemon_check_result=4294967295`)

#### `vesdk.ini` (SDK Config)
- `HardwareEncoderEnable=1`, `HardwareDecoderEnable=1`, `GPUTranscodeEnable=1`
- `MaxDecoderCount=8`, `MaxEncoderCount=4`
- `EnableSVTAV1Encoder=1`, `EnableByteVC1Encoder=1`, `EnableByteVC1Decoder=1`

---

## 3. Lynx Runtime UI Evidence (Observation)

### 3.1 Static `lynx_config` (from Phase 1)
14 page configs: `credits_center`, `credits_center_ng`, `credits_history_ng`, `global`, `message_center`, `modal`, `restore_page`, `subscription*`, `commerce_popup`, `vicut_lynx_commerce_comp_popup`.

### 3.2 Runtime Lynx Page Bundles (Cache)
Location: `User Data\Cache\GeckoCpp\resources\3d692397b923263594be6bf2b778858c\image_lynx_videocutpc_modal\1155441196\res\pages\`

| Page | `template.js` Size | Status |
|------|-------------------|--------|
| `commerce_splash_modal` | 1,035,890 B | Binary Lynx bundle |
| `editor_vip_benefit_popup` | 871,182 B | Binary Lynx bundle |

**REA `analyze_javascript_application`** (`ev_fb1983c...d4dad`): parsed 3 sources but binary `template.js` blobs could not be decompiled (Lynx-compiled bytecode). Limitation: Lynx bundles are compiled bytecode, not JavaScript source.

**Missing:** No `editor_video` page found in cache — may require app interaction to fetch.

---

## 4. Native Addon Surface (Observation)

| Addon | Size | SHA-256 | Evidence ID | Format |
|-------|------|---------|-------------|--------|
| `videoeditor_addon.node` | 3,689,984 B | `25c89e933a90...` | `ev_76d6034...bdaa` | PE32+ |
| `agent_runtime_addon.node` | 521,216 B | `82c9a4ab2bd2...` | `ev_0acb321...e35cc9` | PE32+ |
| `ttnet_frontier_addon.node` | 132,608 B | `f60f56f75ec9...` | `ev_7c23c1e...f12ae` | PE32+ |

All three are **PE32+ native addons** (Node.js native modules). REA `inspect_artifact` returns artifact graph only; no export table or symbol enumeration. Hypothesis: these are **Node.js native addons** exposing C++ functions to JS (likely via N-API or NAN). Next probe: `open_binary` + Ghidra for export enumeration.

---

## 5. Feature ↔ Native Correlation (Deferred)

| Feature (JS) | Candidate Native DLL | Search Status |
|--------------|---------------------|---------------|
| `featureMask` | `VECreator.dll`, `videoeditor.dll` | CLI `search-bytes` failed (tool parses pattern as path) |
| `featureBlend` | `VECreator.dll`, `cccreator.dll` | Deferred |
| `featureBlit` | `VECreator.dll`, `cccreator.dll` | Deferred |
| `featureMask/Blend/Blit` configs | `lumi_js_resources` JSON | Observed in `featureMask/config.json`, `featureBlend/config.json`, `featureBlit/config.json` |

**Blocker:** REA CLI `search-bytes` command parses pattern as file path (bug/design). MCP `search_strings` unavailable in this session. Correlation deferred.

---

## 6. Limitations & Unknowns

| Limitation | Impact |
|------------|--------|
| REA MCP server failed in this session (30s startup timeout) | No retained Evidence across CLI calls; each npx call is cold |
| Lynx `template.js` bundles are binary bytecode | Cannot decompile UI logic; need Lynx-specific decompiler or runtime capture |
| `MMKV\settings_json` (8 MB) format unknown | Not JSON/protobuf/flatbuffers identified; binary blob |
| `Projects\` empty | No draft/project schema to reverse; clone must design its own |
| REA CLI `search-bytes` parses pattern as file path | Cannot search DLL strings for feature names via CLI |
| No Ghidra/Hopper installed | Native export enumeration & decompilation unavailable |
| No `editor_video` Lynx page in cache | Requires app interaction (user action) to fetch |
| CapCut not running | No runtime IPC/CEF message capture possible |

---

## 7. Evidence Portability

All Evidence saved under `research\ev_*.json`:
- `ev_76d6034d5770ed28270984908275fbeb08f601001855b56c60c8299bb373bdaa` (videoeditor_addon)
- `ev_0acb3216e1faeea413b3521fc922d687a9a0666fcd99f0be6d2c6073e9e35cc9` (agent_runtime_addon)
- `ev_7c23c1eceadb26fbe594ff68e7460a9d33cc37a047c66f9076d45cf6d1af12ae` (ttnet_frontier_addon)
- `ev_fb1983ccdf62ea30f938469ac0af8a38aa6a44d04c0420805883cac4781f4dad` (Lynx modal)
- Prior Phase 1 Evidence: `ev_79891145...`, `ev_4ae3abe4...`, `ev_554cfbe5...`, `ev_7ccf1cd0...`, `ev_4bcb266...`, `ev_7c4c13c...`

---

## 8. Next Step Proposal (Phase 2 Complete → Phase 3 Gate)

Phase 2 evidence collected. Remaining high-value targets requiring explicit approval:

| Next Probe | Rationale | Approval Needed |
|------------|-----------|-----------------|
| **Restart opencode** → REA MCP tools appear | Retained Evidence, `binary_session`, `search_strings` | Session restart |
| **Ghidra install + `open_binary` on `.node` addons** | Export enumeration, N-API function signatures | Engine install (separate dry-run) |
| **User creates throwaway project in CapCut** | Materialize draft DB in `Projects\` for schema extraction | User action (not mine) |
| **Runtime CEF capture** (`capture_browser_scenario` on CapCut CEF view) | IPC/JS↔native message flow | CapCut execution + CDP exposure check |
| **Lynx bundle decompilation** | Need Lynx-specific tool or runtime DOM capture | Tool availability check |

**Gate decision:** Phase 2 evidence complete for static/config surface. Remaining work requires either (a) MCP restoration, (b) engine install, or (c) user-performed runtime action. Await your choice.

---

**All Evidence files in `research\`. Config copies in `research\appdata\UserData\`.**