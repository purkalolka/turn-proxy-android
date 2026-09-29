# AGENTS.md

## Project Overview
Android client (`com.freeturn.app`) and VPS orchestrator for `free-turn-proxy`.
- **Stack**: Kotlin (2.4.10), Jetpack Compose (BOM 2026.08.00, Material 3 1.5.0-alpha26), AGP 9.1.1, Java 17, Koin DI, Coroutines/Flow, JSch (SSH), CameraX + ML Kit (QR code).
- **Core engine**: Gomobile AAR library (`freeturn.aar`) compiled from `samosvalishe/free-turn-proxy` (Go) running userspace WireGuard/AmneziaWG tunneling and TURN proxying.
- **Server controller**: Modular Bash scripts in `server-control/src/` assembled into a single payload sent over SSH.

---

## Build & Toolchain Requirements

- **JDK**: Java 17 (required by Gradle 9.5.1 and Kotlin compiler target `JVM_17`).
- **Android SDK**: `compileSdk = 37`, `minSdk = 24`, `targetSdk = 37`.
- **ABIs**: `arm64-v8a` and `armeabi-v7a` only (universal APK is disabled by split config in `app/build.gradle.kts`).
- **Core library desugaring**: Enabled via `desugar_jdk_libs:2.1.5` (necessary for `java.time` in SSH layer on API < 26).

---

## Crucial Build Tasks & Behaviors

### 1. `freeturn.aar` Engine Download (`fetchFreeturnAar`)
- Triggered automatically before build (`preBuild.dependsOn(fetchFreeturnAar)`).
- Downloads `freeturn.aar` and verifies SHA-256 against `checksums.txt` from GitHub releases.
- Configured in `gradle.properties` (`freeturnAar=4.0.1`) or via CLI/Env:
  - `-PfreeturnAar=<version>` or `export FREETURN_AAR_VERSION=<version>`
  - `-PfreeturnAar=local`: skips downloading and uses existing `app/libs/freeturn.aar` (fails if missing).
  - `-PfreeturnAarRepo=<owner/repo>`: defaults to `samosvalishe/free-turn-proxy`.
  - Env `GITHUB_TOKEN`: used if set to authenticate against GitHub API and avoid rate limits.
  - Cached in `~/.gradle/caches/freeturn-core/<version>`.
  - Stamped via `app/libs/.aar-version`.

### 2. Server Control Script Generation (`assembleControlScript`)
- Assembles all `.sh` fragments in `server-control/src/*.sh` in lexicographical order into a single `free-turn-control.sh`.
- Automatically registered as generated asset directory (`variant.sources.assets.addGeneratedSourceDirectory(...)`) for all variants.
- **Never edit generated script directly** — only edit source files in `server-control/src/`.

### 3. Signing & Keystore
- Release builds look for `keystore.properties` in project root:
  ```properties
  storeFile=release.jks
  storePassword=...
  keyAlias=...
  keyPassword=...
  ```
- If `keystore.properties` does not exist, release signing config is omitted (release builds will be unsigned). Debug builds use default debug signing.

---

## Developer Commands

### Android Builds & Verification
```bash
# Run unit tests
./gradlew testDebugUnitTest

# Run a specific unit test class
./gradlew testDebugUnitTest --tests "com.freeturn.app.domain.ssh.SshHostKeySecurityTest"
./gradlew testDebugUnitTest --tests "com.freeturn.app.data.share.FreeturnLinkTest"

# Assemble Debug APK (creates split APKs for arm64-v8a and armeabi-v7a)
./gradlew assembleDebug

# Assemble Release APK (requires keystore.properties or IDE signing)
./gradlew assembleRelease

# Lint check
./gradlew lintDebug

# Generate Compose compiler metrics & stability reports
./gradlew assembleRelease -PcomposeReports=true
```

### Server Control Script Tests (Bats)
Tests for the server orchestrator bash scripts live in `server-control/test/`:
```bash
# Run bats tests (Linux/WSL/macOS or Git Bash with bats-core installed)
bats server-control/test/
bats server-control/test/wg.bats
bats server-control/test/runtime.bats
```

---

## Architecture & Code Conventions

- **Engine JNI Layer**:
  - `com.freeturn.core.mobile.Mobile` is provided by `freeturn.aar`.
  - Single access point in Kotlin: `ProxyEngine` (`app/src/main/java/com/freeturn/app/domain/proxy/ProxyEngine.kt`), registered as a Koin singleton.
  - Session lifecycle is serialized with an internal mutex and monotonic session IDs (`issued` / `cancelled`) to prevent race conditions where a slow/stale start command from a dying service arrives after a new start.
  - Native socket protection: `VpnService.protect(fd)` is called by the Go engine via `SocketProtector` to prevent routing loops.
  - TUN interface descriptor: `tunHandle.dupFd()` creates a duplicated FD via `ParcelFileDescriptor.dup().detachFd()` for Go because the Go core takes ownership and closes the descriptor on shutdown.
- **Service Layer**:
  - `ProxyService` extends Android's `VpnService` with foreground service type `specialUse` (requires Android 14+ subtype declaration in manifest).
  - Handles screen on/off wake locks and deep-sleep gaps (`DEEP_SLEEP_KICK_MS = 60_000ms`), kicking the Go engine to recycle stale UDP/TURN allocations.
  - Trampoline: `ProxyShortcutActivity` is used to prompt for VPN permission if not granted before starting `ProxyService`.
- **SSH & Server Management**:
  - Uses JSch with BouncyCastle for key negotiation and known_hosts management (`SshHostKeySecurityTest.kt`).
  - Streams `free-turn-control.sh` via stdin (`bash -s -- <subcmd> <args>`) to remote host; parses JSON responses.
- **Versioning**:
  - `versionName` in `app/build.gradle.kts` is controlled via `release-please` (`// x-release-please-version`).
  - `versionCode` is computed automatically: `major * 10000 + minor * 100 + patch`.
