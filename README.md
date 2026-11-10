# IE Stopwatch Pro — Phase 1 (Dashboard + Professional Stopwatch)

Status: source project only. **No APK has been built or verified yet** (see BUILD).

## Architecture
MVVM, single Activity, Compose UI, offline-first.

    com.iestopwatchpro
    ├─ MainActivity.kt            AppCompatActivity (needed for in-app language switching)
    ├─ stopwatch/
    │   ├─ SwState.kt             immutable state + all timing/cycle/stat logic (pure Kotlin)
    │   ├─ StopwatchViewModel.kt  takes the monotonic timestamp at tap time, persists after each action
    │   └─ Feedback.kt            beep / vibration
    ├─ data/
    │   ├─ SwCodec.kt             state (de)serialisation + reboot recovery (pure Kotlin)
    │   └─ SwStore.kt             DataStore: stopwatch snapshot + settings
    └─ ui/                        Theme, Dashboard, Stopwatch, Settings

Planned for later phases (not in Phase 1): Room entities (Study, Operation, Operator, Cycle, Element,
Rating, Allowance, Production, Line, WorkstationAllocation), migrations, backup/restore, exports.

## Timing design
- Every displayed time = accumulated + (SystemClock.elapsedRealtimeNanos() − segmentStart). No tick counter.
- Screen lock / navigation: nothing is "counting"; time is recomputed from the monotonic clock on redraw.
- App restart: state is saved to DataStore on every action and restored. If the device rebooted while
  running (detected via BOOT_COUNT), elapsed time is rebuilt from wall-clock time (less precise, by design).
- Not yet implemented: a foreground service. If Android kills the process, the saved state restores correctly,
  but there is no ongoing notification.

## Phase 1 behaviours / assumptions (change if you disagree)
- Cycle time = time between consecutive Cycle/Lap taps (continuous mode); the clock never stops at a tap.
- "Next cycle timer" = average of included cycles minus current cycle time (shown red with "+" once overrun).
- Marking a cycle Abnormal also excludes it from statistics; it is never deleted and can be re-included.
- Deleting a cycle keeps the other cycle numbers unchanged. Editing changes the cycle duration only.
- Display is millisecond-precision (truncated). Digits stay Latin in Bengali mode.

## Formulas (Phase 1)
- Cycle duration_n = T_n − T_(n−1)   (T = total elapsed at the tap)
- Average / Min / Max over included cycles only.

## BUILD (debug APK)
1. Install Android Studio (Koala or newer) with JDK 17 and Android SDK 34.
2. File ▸ Open this folder; let Gradle sync (needs internet once for dependencies).
3. Run tests: `./gradlew testDebugUnitTest` (generate the wrapper first with `gradle wrapper --gradle-version 8.7` if `gradlew` is missing, or run tests from Studio).
4. Debug APK: Build ▸ Build APK(s), or `./gradlew assembleDebug` → app/build/outputs/apk/debug/app-debug.apk
5. Release APK: create a keystore (Build ▸ Generate Signed Bundle / APK ▸ APK), choose "release".

## Manual test checklist (Phase 1)
- Start, lock screen 60 s, unlock: total time continues correctly.
- Start, press Home, return after 30 s: correct.
- Start, force-stop app from recents, reopen: still running with the right time.
- Cycle ×5: previous/current/avg/min/max correct; edit, mark abnormal, exclude, delete work.
- Settings ▸ বাংলা: all labels switch; stopwatch data remains.
