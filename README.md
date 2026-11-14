# TV-runtime

Mirrored runtime modules and build-time dependencies for
[ciweihaha/TV](https://github.com/ciweihaha/TV) (a fork of FongMi TV).

## Why this repo exists

The TV app used to fetch these files from upstream `FongMi/Release` at build
time and at runtime (in-app runtime-module updates). Upstream deleted a
published asset once already (`runtime-mpv-1.1.0`, which broke a build), so
everything the app needs is now mirrored here, under the user's own account.
Nothing in `ciweihaha/TV` fetches from `FongMi/Release` anymore.

## Contents

- `catalog.json` — runtime module catalog (schema 1) consumed by the TV app.
  The app reads it from
  `https://raw.githubusercontent.com/ciweihaha/TV-runtime/main/catalog.json`
  (see `app/build.gradle` → `runtimeCatalog` in ciweihaha/TV). All 6 modules
  (python, nodejs, quickjs, thunder, bdj, mpv), both ABIs, with sizes and
  sha256 digests copied from the upstream catalog.
- `module/` — the 12 runtime module APKs referenced by `catalog.json`.
- `build/` — the two official 5.8.7 mobile APKs (`mobile-arm64_v8a.apk`,
  `mobile-armeabi_v7a.apk`). Only used by the TV CI workflow to extract the
  unified `libffmpeg.so` / `libffmpegDoviJNI.so`; never installed or shipped.

All files are byte-identical to the upstream originals (sha256 verified
against the upstream catalog at mirror time, 2026-11-14).

## Updating a runtime module

Upstream publishes new module versions on the `FongMi/Release` `module` tag.
To mirror an update:

1. Download the new `runtime-<module>-<version>-<abi>.apk` pair from upstream.
2. In the `ciweihaha/TV` repo, run the catalog tool with the new base URL and
   the current catalog as base (it validates APK contents, versions and
   digests, and refuses downgrades):
   `python3 scripts/runtime_catalog.py <dir-with-new-apks> --output <dir> --url-base https://raw.githubusercontent.com/ciweihaha/TV-runtime/main/module --base-catalog catalog.json`
3. Upload the new APKs to `module/` here and replace `catalog.json` with the
   regenerated one. The app picks the new versions up on its next catalog
   check — no app rebuild needed for a module-only update.

The Media3 source fork used by CI is mirrored separately as
[ciweihaha/media](https://github.com/ciweihaha/media) (fork of `FongMi/media`,
pinned commits in `.github/workflows/build-apk.yml`).
