# Release v4.4
## Bugfixes
- Fixed macOS Gatekeeper rejection (Zero-Trust CFBundleExecutable sync). 
- Resolved issue where internal binary `FractalOps` was mismatched with `Info.plist` target name.
- Enforced unified Version String (`v4.4`) across Info.plist and DMG filename to prevent cache invalidation anomalies.
