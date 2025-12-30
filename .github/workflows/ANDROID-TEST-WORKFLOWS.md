# Android Test Build Workflows

## Overview

Two new optimized GitHub Actions workflows have been created for building test Android APKs following 2025 best practices. These workflows generate debug builds for personal testing without requiring production keystore configuration.

## Workflows

### 1. Android Changes Detector (`android-changes-detector.yml`)

**Purpose:** Automatically detect Android-related changes and trigger builds only when necessary.

**Features:**
- ✅ Monitors changes in Android-related files and directories
- ✅ Uses `dorny/paths-filter@v3` for efficient change detection
- ✅ Concurrency control to cancel duplicate builds
- ✅ Fast execution (5-minute timeout)
- ✅ Provides clear change detection results

**Monitored Paths:**
- `android/**` - Android native code
- `src/**` - Frontend source code
- `capacitor.config.ts` - Capacitor configuration
- `package.json` - Dependencies
- `package-lock.json` - Dependency lockfile
- Workflow files themselves

**Triggers:**
- `pull_request` on branches: `main`, `master`
- `push` on branches: `main`, `master`

### 2. Build Android Test APK (`build-android-test.yml`)

**Purpose:** Build debug Android APKs with comprehensive caching and monitoring.

**Features:**

#### 🚀 2025 Best Practices Implemented:

1. **Multi-Layer Cache Optimization:**
   - `setup-node@v6` with automatic npm cache
   - `setup-java@v5` with automatic Gradle cache
   - `gradle/actions/setup-gradle@v5` with granular cache control
   - Android SDK components cache
   - Frontend build output cache for incremental builds
   - Additional npm directory cache

2. **Gradle Optimizations:**
   - `GRADLE_OPTS` configured for CI environments
   - Parallel builds enabled
   - Build caching enabled
   - Configuration on demand
   - Build Scan integration for advanced debugging
   - Dependency graph submission for security analysis

3. **Performance:**
   - Concurrency control (cancels old builds)
   - Timeout protection (45 minutes)
   - `npm ci --prefer-offline --no-audit` for faster installs
   - Incremental frontend builds
   - Zero compression for APK artifacts (already compressed)

4. **Debug & Monitoring:**
   - Automatic Build Scan generation (`--scan` flag)
   - Gradle logs uploaded on failures
   - Cache hit statistics in job summary
   - APK validation with SHA256 hash
   - Comprehensive build information

5. **Security:**
   - Dependency graph submission (automatic vulnerability detection)
   - SHA256 hash in releases
   - APK integrity validation

**Triggers:**
- `workflow_call` - Called by the changes detector
- `workflow_dispatch` - Manual execution with optional release name

**Build Output:**
- Debug APK (fdroid flavor)
- No production keystore required
- Not optimized (faster build, larger size)
- Suitable for personal testing only

## Key Differences vs Production Build

| Aspect | Production (`build-android.yml`) | Test (new workflows) |
|--------|----------------------------------|----------------------|
| **Build Type** | `release` (assembleRelease) | `debug` (assembleDebug) |
| **Flavor** | `play` | `fdroid` |
| **Command** | `npm run dist:android:prod` | `npm run dist:android` |
| **Keystore** | Required (production) | Not required (debug auto-sign) |
| **Optimization** | Minified, ProGuard | Not optimized (debug) |
| **Google Play** | ✅ Upload automático | ❌ Não publica |
| **Destino** | Google Play Store | Teste pessoal |
| **APK Path** | `play/release/*.apk` | `fdroid/debug/*.apk` |
| **Cache Strategy** | Basic | Multi-layer optimized 2025 |
| **Build Scan** | No | ✅ Enabled |
| **Dependency Graph** | No | ✅ Enabled (security) |

## Usage

### Automatic (via Changes Detector)

1. Create a PR or push to `main`/`master` with Android changes
2. Detector workflow runs automatically
3. If changes detected, build workflow is triggered
4. APK is built and uploaded to artifacts
5. For push/merge events, APK is published to Releases

### Manual Execution

1. Go to **Actions** → **Build Android APK - Test (Personal Use)**
2. Click **Run workflow**
3. (Optional) Enter a custom release name
4. Click **Run workflow**
5. Wait ~6-8 minutes for first build (2-3 minutes with cache)
6. APK available in Releases with `-test` suffix

## APK Installation

1. **Download** the APK from GitHub Releases or Artifacts
2. On your **Android device**:
   - Go to Settings → Security
   - Enable "Install from unknown sources"
3. **Open** the downloaded APK
4. Tap **Install**
5. If you have an existing version, you may need to uninstall first

## Build Outputs

### Artifacts
- **Name:** `super-productivity-android-test-{build_number}`
- **Retention:** 30 days
- **Location:** Actions run artifacts

### Releases
- **Tag Format:** `v{version}-test-{date}-build{number}`
- **Marked as:** Prerelease
- **Contains:** APK file with SHA256 hash
- **Only created on:** Push/merge to main/master (not on PRs)

### PR Comments
For pull requests, the build workflow adds a comment with:
- ✅ Build success confirmation
- 📦 APK information (name, size, SHA256)
- 📥 Link to download artifacts
- ⚠️ Note that Release is only created after merge

## Cache Performance

Expected cache hit rates:
- **npm cache:** 90-95% (changes with package.json updates)
- **Frontend build:** 70-80% (changes with src/ updates)
- **Gradle cache:** 85-90% (managed automatically)
- **Android SDK:** 95-99% (rarely changes)

First build: ~6-8 minutes
Cached builds: ~2-3 minutes (60-70% faster)

## Security Features

1. **Dependency Graph Submission:**
   - Automatically submits dependency information
   - Enables Dependabot security alerts
   - Identifies vulnerable dependencies

2. **SHA256 Hash:**
   - Every APK includes SHA256 checksum
   - Verifiable integrity
   - Included in releases and comments

3. **Cache Read-Only for PRs:**
   - Prevents cache corruption from PRs
   - Maintains cache integrity
   - Write access only on main/master

## Troubleshooting

### Build Fails
1. Check the **Build Scan** link in the workflow output
2. Review **Gradle logs** (uploaded on failure)
3. Check cache hit statistics for anomalies

### APK Not Generated
1. Verify the `dist:android` command in package.json
2. Check that Android SDK is properly installed
3. Review the "List and validate generated APK files" step

### Cache Issues
1. Cache automatically cleans up old entries
2. Manual cache clear: Re-run workflow after 7 days
3. Check cache hit statistics in job summary

## Maintenance

### Updating Actions Versions
All actions are pinned to major versions (v5, v6) for stability:
- `actions/checkout@v6`
- `actions/setup-node@v6`
- `actions/setup-java@v5`
- `gradle/actions/setup-gradle@v5`
- `android-actions/setup-android@v3`
- `actions/cache@v4`
- `actions/upload-artifact@v5`
- `softprops/action-gh-release@v2`
- `actions/github-script@v7`
- `dorny/paths-filter@v3`

### Cache Cleanup
Automatic cache cleanup is enabled. Old cache entries are removed after 7 days of inactivity.

### Monitoring
Check the job summary after each run for:
- ✅ Cache hit statistics
- 📱 APK information
- 🔍 Build Scan links
- 📊 Dependency graph status

## Best Practices

1. **Always merge with Android changes** to trigger automatic builds
2. **Review Build Scan** for performance insights
3. **Verify SHA256** before installing APKs
4. **Monitor cache hits** for optimization opportunities
5. **Keep actions updated** to latest major versions
6. **Use workflow_dispatch** for custom release names

## Future Improvements

Possible enhancements for future iterations:
- [ ] Add build time comparisons
- [ ] Implement build artifact size tracking
- [ ] Add automated APK testing
- [ ] Create build performance dashboard
- [ ] Add build failure notifications
- [ ] Implement build status badges

## Support

For issues or questions:
1. Check the **Actions** tab for detailed logs
2. Review the **Build Scan** for Gradle-specific issues
3. Check **Artifacts** for build outputs
4. Review **Dependency Graph** for security issues

---

**Created:** 2025-12-25
**Version:** 1.0
**Compatible with:** Android SDK 21+, Node.js 20, Java 21
