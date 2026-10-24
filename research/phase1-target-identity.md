# Phase 1 — CapCut Target Identity & Architecture Evidence

**Date:** 2026-10-23
**Analyst:** REA 6.5.0 via CLI fallback (MCP server unavailable in session)
**Target:** CapCut Desktop 9.6.0.4097
**Installation:** `%LOCALAPPDATA%\CapCut\Apps\9.6.0.4097\`

---

## 1. Target Verification (Observation)

| Source | Value | Evidence Type |
|--------|-------|---------------|
| Shortcut `CapCut.lnk` | Target: `Apps\CapCut.exe --src1` | Filesystem |
| Registry `HKCU\...\Uninstall\CapCut` | `DisplayVersion=9.6.0.4097`, `UninstallString=...\uninst.exe` | Registry (read-only) |
| `ProductInfo.xml` | `full_appver=9.6.0.4097`, `cid=capcutpc_0`, `release_type=release` | Shipped artifact |
| FileVersion (stub) | `1.0.0.000000` (placeholder) | PE metadata — **not authoritative** |

**Confirmed:** CapCut 9.6.0.4097, ByteDance Pte. Ltd., per-user install under `%LOCALAPPDATA%`.

---

## 2. Installation Layout (Observation)

```
%LOCALAPPDATA%\CapCut\
├── Apps\
│   ├── CapCut.exe              # 4,974,504 B — launcher/stub (rich PE resources)
│   ├── uninst.exe              # 4,115,880 B
│   ├── Configure.ini           # 28 B (last_version=)
│   ├── ProductInfo.xml         # 368 B
│   └── 9.6.0.4097\             # Versioned payload
│       ├── CapCut.exe          # 90,536 B — minimal entry point
│       ├── VECreator.dll       # 271,139,752 B — render/editor core
│       ├── videoeditor.dll     # 72,501,160 B
│       ├── cccreator.dll       # 158,564,776 B
│       ├── 167 .dll            # Qt6, CEF, FFmpeg, Lynx, etc.
│       ├── 3 .node addons      # videoeditor_addon, agent_runtime_addon, ttnet_frontier_addon
│       ├── 2 .js               # lynx_core.js, lynx_core_dev.js
│       ├── Resources\          # JS bundles, assets, configs
│       │   ├── lumi_js_resources\
│       │   ├── lumi_js_resources_video\
│       │   ├── lynx_config\    # 14 Lynx UI configs
│       │   ├── builtin_skills\ # 6 AI agent skills
│       │   └── ...
│       ├── cef\                # CEF  (libcef.dll 217 MB)
│       ├── Qt\                 # Qt6 (Core, Gui, Quick, Widgets, Qml, Network, etc.)
│       └── ...
└── User Data\                  # Runtime state (Projects, Config, MMKV, SettingsSDK, CEF, etc.)
```

---

## 3. PE Resource Evidence (Observation)

| Artifact | Evidence ID | SHA-256 | Size | Resource Summary |
|----------|-------------|---------|------|------------------|
| `9.6.0.4097\CapCut.exe` (entry) | `ev_7ccf1cd...660a` | `1bf33b74a263...` | 90 KB | 7 resources: 4×RT_ICON, 1×RT_GROUP_ICON, 1×RT_VERSION, 1×RT_MANIFEST |
| `9.6.0.4097\VECreator.dll` | `ev_4bcb266...b6d3` | `5bac0b46e9e0...` | 271 MB | 2 resources: RT_VERSION, RT_MANIFEST |
| `Apps\CapCut.exe` (launcher) | `ev_7c4c13c...f64df3` | `db0a1848273f...` | 4.9 MB | 45 resources: 21×BINDATA (lang 2052), 4×RT_DIALOG, 9×RT_STRING, 2×RT_VERSION (1033/2052), 1×RT_MANIFEST, 1×RT_GROUP_ICON |

**Inference:** The `Apps\CapCut.exe` is the **launcher/stub** with full Windows UI resources (dialogs, strings, BINDATA for Chinese UI). The versioned `9.6.0.4097\CapCut.exe` is a minimal entry point (90 KB). VECreator.dll is a pure DLL with no UI resources.

---

## 4. JavaScript / Lynx Bundle Evidence (Observation)

| Bundle | Evidence ID | Files Analyzed | Key Findings |
|--------|-------------|----------------|--------------|
| `Resources\lumi_js_resources` | `ev_7989114...d9b0e` | 20 sources | Feature configs: `featureMask`, `featureBlit`, `featureBlend`, `lumi_hub`; JSON + Lua meta; `config.json` with `ae_tool`, `bALG_BACH_CONFIG`, `effect`, `name`, `version` |
| `Resources\lumi_js_resources_video` | `ev_4ae3ab...9ccb2` | (similar structure) | Video-specific feature configs |
| `Resources\lynx_config` | `ev_554cfb...e701` | 14 configs | Lynx UI page configs: credits_center, message_center, modal, subscription*, commerce_popup, global, restore_page, etc. |

**Inference:** CapCut's UI is built on **Lynx** (ByteDance's cross-platform UI framework) with JSON-driven feature definitions. The `lumi_js_resources` bundles define editing features (mask, blit, blend) with algorithm configs and Lua meta. `builtin_skills\` contains 6 AI agent skills (highlight, material_understanding, script, speaker, align_bgm_with_video, lvc).

---

## 5. Native Dependencies Inventory (Observation)

**Core frameworks (from filesystem):**
- **Qt 6** — `Qt6Core`, `Qt6Gui`, `Qt6Quick`, `Qt6Widgets`, `Qt6Qml`, `Qt6Network`, `Qt6Sql`, `Qt6Svg`, `Qt6OpenGL`, `Qt6Concurrent`, `Qt6QuickControls2`, `Qt6QuickTemplates2`, `Qt6Labs*`
- **CEF** — `libcef.dll` (217 MB), `chrome_elf.dll`, `v8_context_snapshot.bin`, `snapshot_blob.bin` → Chromium Embedded Framework for web views
- **FFmpeg 6/7** — `avcodec-61`, `avformat-61`, `avfilter-10`, `avutil-59`, `swresample-5`, `swscale-8` → Media decode/encode
- **Lynx** — `lynx_core.js`, `lynx_core_dev.js`, `lynx_config\*` → ByteDance UI framework
- **Bun** — `bun.exe` (107 MB) → JavaScript runtime for tooling
- **OpenVINO** — `openvino.dll`, `openvino_intel_npu_plugin.dll` → AI inference acceleration
- **AI/ML** — `deepagents_capi.dll` (42 MB), `DeepAgentsService.dll`, `bytebenchsdk.dll`, `bytenn_openvinowrapper.dll`
- **Video Editor Core** — `VECreator.dll` (271 MB), `videoeditor.dll` (72 MB), `cccreator.dll` (158 MB), `clipflow_sdk.dll`, `EffectPlatform.dll`, `FusionUI.dll`
- **Node addons** — `videoeditor_addon.node`, `agent_runtime_addon.node`, `ttnet_frontier_addon.node` → Native bindings

---

## 6. Analysis Method Selection (Decision Record)

| Question | Selected Tool | Rationale |
|----------|---------------|-----------|
| Version/identity/icons/manifest | `inspect_pe_resources` | Portable, no engine, complete for PE32+ |
| JS feature configs, Lynx UI schema | `analyze_javascript_application` | Static, no native engine, covers directory bundles |
| Archive/package inventory | `inspect_artifact` | If MSIX/AppX/ZIP encountered |
| Settings/draft DBs | `inspect_sqlite_database` | Node 24.21 ✓; requires quiescent copy |
| Native editor pipeline (codec dispatch, render graph) | `open_binary` + Ghidra | Only if static evidence insufficient; Windows Ghidra P0 is read-only |
| Runtime behavior (IPC, CEF messages) | `list_browser_targets` / process capture | Last resort; executes target |

**Not applicable:** Electron tools (`list_electron_targets`, `inspect_electron_fuses`) — no ASAR, no `electron.exe`.

---

## 7. Limitations & Unknowns

| Limitation | Impact |
|------------|--------|
| MCP server failed to start in this session (30s timeout) | No retained Evidence across CLI invocations; each npx call is a fresh session |
| CLI `inspect-analysis-view` cannot use retained Evidence references | Must pass inline Evidence (large JSON) or re-run producer |
| VECreator.dll (271 MB) exceeds default 64 MB PE resource limit | Worked with `--max-file-bytes 300000000` |
| `lynx_core.js` / `lynx_core_dev.js` are single files, not directories | `analyze_javascript_application` rejects them; need directory wrapper or different tool |
| `User Data\` databases not yet inspected | High-value target for clone data model; need quiescent copies |
| No runtime observation performed | Feature behavior (export pipeline, AI skills, CEF↔native IPC) unobserved |
| Ghidra/Hopper/IDA not installed | Native decompilation unavailable; would require separate approval & setup |

---

## 8. Evidence Index (for traceability)

| Evidence ID | Artifact | Operation | Confidence |
|-------------|----------|-----------|------------|
| `ev_7ccf1cd0f43cdac7df13d008e091d22503d0d1b7eb7decd60394c0aa34ca660a` | `9.6.0.4097\CapCut.exe` | `inspect_pe_resources` | observed |
| `ev_4bcb266ddc83a3f8d3fb6c1982fcea0a3df14638f320c20ba21ab5b4ffe3b6d3` | `9.6.0.4097\VECreator.dll` | `inspect_pe_resources` | observed |
| `ev_7c4c13cabc0ffd29ebfae9daf11b58cdde72416e9040e2613d6b32dd54f64df3` | `Apps\CapCut.exe` (launcher) | `inspect_pe_resources` | observed |
| `ev_79891145a2ded34d25ab28af31b24042ce9265177d7aae50cab7ead42ead9b0e` | `Resources\lumi_js_resources` | `analyze_javascript_application` | observed |
| `ev_4ae3abe4b69337176c5d915b95040af709f9ebdefa3d2256d3af6fe6a899ccb2` | `Resources\lumi_js_resources_video` | `analyze_javascript_application` | observed |
| `ev_554cfbe560ee7be51ba88a981ac46a86e8e62c7a08f78dc486f5d348cd1ae701` | `Resources\lynx_config` | `analyze_javascript_application` | observed |

---

## 9. Proposed Phase 2 Plan

1. **Inspect CapCut settings/draft databases** — copy `User Data\Config`, `SettingsSDK`, `Projects` to quiescent location; run `inspect_sqlite_database` for schema + key tables.
2. **Deep-dive Lynx UI configs** — extract component hierarchy, prop schemas, and page routing from `lynx_config` Evidence.
3. **Map feature JSON → native entrypoints** — correlate `lumi_js_resources` feature names (`featureMask`, `featureBlend`, etc.) with native DLL exports (`VECreator.dll`, `videoeditor.dll`, `cccreator.dll`).
4. **Analyze `.node` addons** — `inspect_artifact` on `videoeditor_addon.node` et al. for exported symbols.
5. **Export Evidence bundles** — `export_evidence_bundle` for each analysis to make them portable across sessions.
6. **Decide on native decompilation** — if feature mapping requires it, evaluate Ghidra setup (needs approval for engine install).

---

**All Evidence files saved under `research\` with `_clean.json` suffix. Raw CLI outputs in `research\` with original names.**