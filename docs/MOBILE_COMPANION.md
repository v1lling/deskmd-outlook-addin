# Mobile Companion Research

> Research notes for future Desk mobile app (iOS/Android)

## Key Questions & Answers

### Can Tauri build for mobile?
**Yes.** Tauri 2.0 supports iOS and Android. iOS requires macOS + Xcode. Same codebase as desktop (web view).

### Can we just read files from the filesystem like on desktop?
**No.** iOS sandboxes apps - they can only access their own container. No reading arbitrary folders like `~/Desk/`. Android is more permissive but still needs explicit permissions.

### Does iCloud "just work"?
**Not really.** iOS controls sync timing (battery, network, thermal state). Apps cannot force sync. Background sync can take "minutes to an hour". New devices take up to 7 days to "learn" usage patterns. Files can get evicted from local storage. No conflict resolution for file-based apps. See [deep dive by Carlo Zottmann](https://zottmann.org/2025/09/08/ios-icloud-drive-synchronization-deep.html).

### How does Obsidian handle this?
- **Tech stack:** Electron (desktop) + Capacitor (mobile) - same web codebase
- **Sync options:**
  1. iCloud Drive (free, Apple-only, unreliable)
  2. Obsidian Sync ($8-10/mo, their own service, most reliable)
  3. Git + Working Copy (free, technical)
- They built paid sync because iCloud is unreliable

### Can we sync with Nextcloud/NAS?
**Yes, via WebDAV API.** The [remotely-save](https://github.com/remotely-save/remotely-save) Obsidian plugin proves this works. Key insight: use native HTTP client to bypass CORS restrictions on mobile WebView.

---

## Architecture Decision

**Don't rely on iOS file providers.** Build API-based sync instead:

```
Desktop:  Read/write ~/Desk/ directly (current)
Mobile:   App sandbox ↔ WebDAV API ↔ Nextcloud/NAS ↔ Files
```

The mobile app keeps a local cache and syncs via HTTP to the server. Works offline, syncs when online.

---

## Technical Approach (from remotely-save)

### CORS Bypass
Mobile WebViews hit CORS restrictions. Solution: route HTTP through native layer.
- Obsidian uses native `requestUrl()`
- Tauri would use `@tauri-apps/plugin-http` or Rust invoke

### WebDAV Client
Use [`webdav`](https://www.npmjs.com/package/webdav) npm package, monkey-patch to use native HTTP.

### Nextcloud Chunked Upload
For files >10MB, use Nextcloud's [chunked upload API](https://docs.nextcloud.com/server/latest/developer_manual/client_apis/WebDAV/chunking.html). remotely-save has working implementation.

### Sync Algorithm
Simple timestamp comparison:
1. List remote files (WebDAV PROPFIND)
2. List local files
3. Compare modification times - later wins
4. Track deletions via metadata file (`.desk-sync.json`)
5. Handle conflicts: prompt user or duplicate file

### Server Compatibility
- Auto-detect server type via DAV compliance headers
- BFS directory walking for servers without `Depth: infinity`
- Handle quirks per-server (Nextcloud, Synology, etc.)

---

## Getting App on Phone (Personal Use)

| Method | Cost | Effort | Notes |
|--------|------|--------|-------|
| **AltStore/SideStore** | Free or $99/yr | Low | Free = refresh every 7 days, max 3 apps. Paid dev account = 1 year certs |
| **Xcode direct** | Free or $99/yr | Low | Same limitations as AltStore |
| **TestFlight** | $99/yr | Medium | 90-day builds, no review for internal testing |
| **Ad-Hoc** | $99/yr | Medium | Up to 100 devices, use Diawi/Firebase for distribution |

**Recommendation:** AltStore + paid Apple Developer ($99/yr) for hassle-free personal use.

---

## Public Release (App Store)

| Requirement | Notes |
|-------------|-------|
| Apple Developer Program | $99/year |
| App Store review | Days to weeks, can reject |
| Privacy manifests | Required for file system APIs |
| iOS Sandbox compliance | Must work within restrictions |
| macOS build machine | Required for iOS builds |

Tauri supports App Store distribution: `tauri ios build --export-method app-store-connect`

---

## Android Comparison

Much easier than iOS:
- External storage access with permissions
- APK sideloading trivial (enable "unknown sources")
- No certificate refresh hassle
- Google Play less strict than App Store

---

## Next Steps (When Ready to Build)

1. Set up Tauri iOS target (`tauri ios init`)
2. Implement Rust HTTP commands for WebDAV (bypasses CORS)
3. Port sync logic from remotely-save concepts
4. Add Nextcloud chunked upload for large files
5. Build conflict resolution UI
6. Test with AltStore on personal device

---

## References

- [Tauri 2.0 Mobile Support](https://v2.tauri.app/blog/tauri-20/)
- [Tauri App Store Distribution](https://v2.tauri.app/distribute/app-store/)
- [remotely-save plugin](https://github.com/remotely-save/remotely-save) - WebDAV sync implementation
- [iOS iCloud Sync Deep Dive](https://zottmann.org/2025/09/08/ios-icloud-drive-synchronization-deep.html)
- [Obsidian iOS Help](https://help.obsidian.md/Obsidian/iOS+app)
- [AltStore](https://altstore.io/) / [SideStore](https://sidestore.io/)
- [Nextcloud Chunked Upload](https://docs.nextcloud.com/server/latest/developer_manual/client_apis/WebDAV/chunking.html)
