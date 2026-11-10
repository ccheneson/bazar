# Android (Kotlin / Gradle)

Detected by: `build.gradle*` or `settings.gradle*` at the root.

- test_cmd: `./gradlew :app:testDebugUnitTest`
- lint_cmd: `./gradlew :app:lint`
- build_cmd: `./gradlew :app:assembleDebug`
- ui_test_cmd: `./gradlew :app:connectedDebugAndroidTest` (optional, needs an emulator or device)
- run_check: launch the app with the `run` skill and check the screen against the plan.
  Skip if no emulator or device is available and say so in the report to the Reviewer.
- dep_file: `gradle/libs.versions.toml` (version catalog, never inline coordinates)
- allowlist: `./gradlew *`, `adb *`

Notes:
- Module name `:app` is the default. Read `settings.gradle*` if the project differs.
- Follow the plugin and version conventions in `CLAUDE.md` before touching Gradle files.
