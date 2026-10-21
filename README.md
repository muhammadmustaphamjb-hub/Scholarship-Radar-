# Scholarship Radar

Android app for discovering verified scholarships and research funding.

Status: Phase 0. A minimal Compose app that shows the app name in English and Hausa.
It has NOT yet been compiled or tested by the author. The first build on GitHub Actions
is the first real compile; if it fails, copy the error log and bring it back for fixing.

## Build the APK (from a phone browser)

1. Push this project to the `main` branch of your GitHub repository.
2. Open the repository, tap the Actions tab, and open the latest "Build debug APK" run.
3. When it shows a green check, scroll down to Artifacts and download `ScholarshipRadar-debug-apk`.
4. Unzip the download, tap the .apk file, and allow "install unknown apps" for your browser or file manager when asked.

You can also start a build by hand: Actions, Build debug APK, Run workflow.

## Folders

- `app/` the Android app
- `docs/` architecture, data schema, and Firestore security rules
