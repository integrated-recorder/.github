# Integrated Recorder

[한국어](README.md) | **English**

Integrated Recorder is an open-source archiving system for preserving original live-stream media segments and broadcast metadata over time.

It preserves original HLS media segment bytes and lets Core own the meaning of the archive. The system is designed for headless Docker operation with browser management and VOD playback. The canonical recording path does not require FFmpeg or Streamlink; FFmpeg may be used for optional derivative features.

## Architecture at a glance

```text
Source Plugins
  source.hls           bundled in Core
  source.owncast       official Registry, first-party
  source.soop          development / experimental
  source.chzzk         development / experimental
  third-party plugins
        │
        │ Adapter Protocol v1
        ▼
Core
  Runtime Host · Recorder Engine · Control Plane · Web UI/API
  recording identity · metadata · ordinals · gaps · integrity
  canonical archive · lifecycle · generation · writer fencing
        │
        │ Storage Provider Protocol v1
        ▼
Storage Plugins
  storage.local        bundled reference provider in Core
  future providers     no repository in this organization today
```

Core alone decides what archived bytes mean: it owns Recording identity, metadata timelines, segment ordinals, gap and integrity semantics, canonical commit ordering, and ownership fencing. Source Plugins resolve platform-specific media; Storage Providers place logical objects physically.

## Repository map

### Core and operations

| Repository | Role |
| --- | --- |
| [core](https://github.com/integrated-recorder/core) | The Integrated Recorder application: Runtime Host, Recorder Engine, Control Plane, Web UI/API, canonical archives, and plugin lifecycle. It also owns the official installer, Docker deployment, and installation/upgrade operations documentation. |

There is no separate operations/deployment repository. Installation and upgrade procedures live in `core`. `source.hls` and `storage.local` are bundled implementations in Core, not separate repositories.

### Source plugins

| Repository | Role |
| --- | --- |
| [source.owncast](https://github.com/integrated-recorder/source.owncast) | Owncast integration. A first-party plugin not bundled in Core, distributed as v0.2.0 through the Registry. |
| [source.soop](https://github.com/integrated-recorder/source.soop) | SOOP integration development repository. High-quality stream selection is still being addressed, so stable Registry distribution is deferred. |
| [source.chzzk](https://github.com/integrated-recorder/source.chzzk) | CHZZK integration. Public first-party source plugin in development/experimental status; real historical DVR recording end-to-end remains unverified, and it is not listed in the Registry. |

### Plugin development and distribution

| Repository | Role |
| --- | --- |
| [adapter-sdk-go](https://github.com/integrated-recorder/adapter-sdk-go) | Go Source Plugin development SDK with protocol APIs and framing. Core's conformance runner is the final executable compatibility check. |
| [plugin-registry](https://github.com/integrated-recorder/plugin-registry) | Official distribution catalog for approved exact plugin artifacts. It does not build plugin source. |

## Plugin distribution and trust

Plugin admission paths are `bundled`, `registry`, and `operator`: whether an executable ships in a Core release, has an artifact approved through a Registry, or is supplied locally by an operator. Publisher affiliation is separate metadata: `first-party`, `third-party`, or `unknown`. First-party does not mean bundled.

The official Registry publishes publisher metadata in its v3 catalog and lists Owncast `0.2.0`. Registry approval is approval of distribution metadata and artifact identity, not a safety guarantee or sandbox. Every plugin is native executable code and runs without a sandbox. See the [Core Plugin Trust Model](https://github.com/integrated-recorder/core/blob/main/docs/PLUGIN_TRUST_MODEL.md) and the [Plugin Registry](https://github.com/integrated-recorder/plugin-registry).

## How to publish a plugin through the Registry

The plugin publisher builds and publishes an immutable executable release in the plugin repository, then submits a PR to the [Plugin Registry](https://github.com/integrated-recorder/plugin-registry/blob/main/CONTRIBUTING.md) with the exact source commit and per-platform URL, filename, size, and SHA-256. CI validates the catalog and artifacts; merging the PR publishes the static catalog. The Registry does not build plugin source.

## Current status

The project is in **active development**. Core has public prereleases, but no stable public release yet; see the [release list](https://github.com/integrated-recorder/core/releases) for the current version and status. Owncast is a first-party v0.2.0 plugin in the official Registry. SOOP is under development/experimental, with stable distribution deferred while its high-quality stream selection issue remains unresolved. CHZZK is a public first-party source plugin in development/experimental status; real historical DVR recording end-to-end validation and Registry listing are still pending.

## Where to start

- **Users:** Start with the Docker Compose quick start and setup wizard in [Core](https://github.com/integrated-recorder/core).
- **Source Plugin developers:** Start with the [Go Adapter SDK](https://github.com/integrated-recorder/adapter-sdk-go) and Core's `adapter-conformance`.
- **Official plugin catalog:** Browse the [Plugin Registry v3 catalog](https://integrated-recorder.github.io/plugin-registry/catalog-v3.json).
- **A specific integration:** See [source.owncast](https://github.com/integrated-recorder/source.owncast), [source.soop](https://github.com/integrated-recorder/source.soop), or [source.chzzk](https://github.com/integrated-recorder/source.chzzk).
