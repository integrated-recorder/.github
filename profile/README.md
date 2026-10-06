# Integrated Recorder

**한국어** | [English](README.en.md)

Integrated Recorder는 라이브 스트림의 원본 미디어 세그먼트와 방송 메타데이터를 장기 보존하기 위한 오픈소스 아카이빙 시스템입니다.

원본 HLS media segment를 바이트 그대로 보존하고 Core가 archive 의미를 관리합니다. 운영은 headless Docker와 브라우저 관리·VOD를 중심으로 합니다. canonical recording 경로는 FFmpeg나 Streamlink를 필요로 하지 않으며, FFmpeg는 선택적 derivative 기능에 사용할 수 있습니다.

## 구조 한눈에 보기

```text
Source Plugins
  source.hls           Core bundled
  source.owncast       official Registry, first-party
  source.soop          development / experimental
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
  storage.local        Core bundled reference provider
  future providers     not currently present in this organization
```

Core alone decides what archived bytes mean: it owns Recording identity, metadata timeline, segment ordinal, gap and integrity semantics, canonical commit ordering, and ownership fencing. Source Plugins resolve platform-specific media; Storage Providers place logical objects physically.

## 저장소 지도

| 저장소 | 역할 |
| --- | --- |
| [core](https://github.com/integrated-recorder/core) | Integrated Recorder 실행 본체. Runtime Host, Recorder Engine, Control Plane, Web UI/API, canonical archive와 plugin lifecycle을 소유합니다. |
| [adapter-sdk-go](https://github.com/integrated-recorder/adapter-sdk-go) | Go Source Plugin authoring SDK. Protocol API와 framing을 제공하며 Core conformance runner가 최종 executable 호환성 기준입니다. |
| [plugin-registry](https://github.com/integrated-recorder/plugin-registry) | 승인된 exact plugin artifact의 official index입니다. source를 build하지 않습니다. |
| [source.owncast](https://github.com/integrated-recorder/source.owncast) | Owncast integration. first-party plugin이며 Core에 bundled되지 않고 Registry에서 v0.2.0으로 배포됩니다. |
| [source.soop](https://github.com/integrated-recorder/source.soop) | SOOP integration 개발 저장소. 고화질 stream 선택이 해결되지 않아 stable Registry 배포 대상이 아닙니다. |

현재 존재하지 않는 storage provider 전용 저장소는 없습니다. `source.hls`와 `storage.local`은 각각 Core에 포함된 bundled implementation입니다.

## Plugin 배포와 신뢰

Plugin admission 경로는 `bundled`, `registry`, `operator`로 구분합니다. 이는 executable이 Core release에 포함됐는지, Registry에서 artifact가 승인됐는지, 운영자가 직접 제공했는지를 나타냅니다. Publisher affiliation은 별도 metadata인 `first-party`, `third-party`, `unknown`으로 표현합니다. First-party라고 해서 bundled인 것은 아닙니다.

공식 Registry는 v3 catalog에서 publisher 정보를 제공하며 Owncast `0.2.0`을 포함합니다. Registry approval은 배포 metadata와 artifact identity 승인이지, 안전성 보증이나 sandbox가 아닙니다. 모든 plugin executable은 native code이며 sandbox 없이 실행됩니다. 자세한 내용은 [Core Plugin Trust Model](https://github.com/integrated-recorder/core/blob/main/docs/PLUGIN_TRUST_MODEL.md)과 [Plugin Registry](https://github.com/integrated-recorder/plugin-registry)를 참고하세요.

## 공식 Registry 등록 흐름

Plugin publisher는 자신의 저장소에서 immutable executable release를 build·게시한 뒤, 정확한 source commit과 platform별 URL·filename·size·SHA-256 metadata를 [Plugin Registry에 PR로 제출](https://github.com/integrated-recorder/plugin-registry/blob/main/CONTRIBUTING.md)합니다. CI가 catalog와 artifact를 검증하고 PR이 merge되면 정적 catalog에 배포됩니다. Registry는 plugin source를 build하지 않습니다.

## 현재 상태

프로젝트는 **active development** 상태이며 Core의 안정적인 public release는 아직 없습니다. Owncast는 official Registry의 first-party v0.2.0 plugin입니다. SOOP은 개발/실험 중이고 고화질 stream 선택 문제 때문에 stable 배포를 보류하고 있습니다.

## 어디서 시작할까요?

- **사용자:** [Core](https://github.com/integrated-recorder/core)의 Docker Compose quick start와 setup wizard를 시작하세요.
- **Source Plugin 개발자:** [Go Adapter SDK](https://github.com/integrated-recorder/adapter-sdk-go)와 Core의 `adapter-conformance`를 확인하세요.
- **공식 plugin 목록:** [Plugin Registry v3 catalog](https://integrated-recorder.github.io/plugin-registry/catalog-v3.json)를 확인하세요.
- **특정 integration:** [source.owncast](https://github.com/integrated-recorder/source.owncast) 또는 [source.soop](https://github.com/integrated-recorder/source.soop)를 살펴보세요.
