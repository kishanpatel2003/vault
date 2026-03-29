---
title: "Local-First Two-Way Voice Conversation on macOS Apple Silicon"
type: research_brief
date: 2026-03-28
requested_by: kishan
tags: [voice, stt, tts, whisper, kokoro, openclaw, apple-silicon, local-first, telegram]
mission: null
sources_count: 12
summary: "Local STT and TTS on Apple Silicon are production-ready for conversational use; whisper.cpp (Metal) handles inbound transcription in ~1s, Kokoro-82M handles outbound speech in ~90–200ms, and both integrate with OpenClaw's existing config primitives."
---

## background

This brief covers the practical options for running two-way voice conversation through OpenClaw on a Mac mini (Apple Silicon), with emphasis on local/on-device processing for privacy and cost. The scope covers: (1) speech-to-text for inbound voice messages, (2) text-to-speech for outbound spoken replies, (3) integration with OpenClaw's Telegram channel, and (4) tradeoffs between local and hosted options.

The target UX is: Kishan sends a voice note in Telegram → OpenClaw transcribes it locally → the agent responds → the reply is spoken back as a voice note. Optionally, a live-mic desktop loop could be added later.

## key findings

### speech-to-text (inbound)

- **whisper.cpp** is the most mature local STT option on Apple Silicon. It runs fully on GPU via Metal, supports Core ML for ANE acceleration (claimed 3x speedup over CPU-only), and ships a C CLI binary (`whisper-cli`). On an M4 with the `large-v3-turbo` model (quantized q5\_0 + Core ML), benchmark time is ~1.2s for a short utterance. The `base.en` model is faster (~0.3s) with slightly lower accuracy. v1.8.1 is the current stable release. Source: [whisper.cpp GitHub](https://github.com/ggml-org/whisper.cpp), [mac-whisper-speedtest benchmarks](https://github.com/anvanvan/mac-whisper-speedtest).

- **mlx-whisper** (Apple's MLX framework) benchmarks at ~1.0s on the `large-v3-turbo` model on M4, marginally faster than whisper.cpp for large models. It runs as a Python package (`pip install mlx-whisper`). Advantage: native unified-memory optimization on Apple Silicon. Disadvantage: Python dependency, less battle-tested as a CLI tool for integration. Source: [mac-whisper-speedtest benchmarks](https://github.com/anvanvan/mac-whisper-speedtest).

- **WhisperKit** is a native Swift framework from Argmax, optimized for Apple Neural Engine. Published benchmarks show 0.46s mean latency for streaming ASR, competitive with cloud services. However, it's a Swift package — not a drop-in CLI binary. Best suited for native macOS/iOS apps, less suitable for OpenClaw's CLI-based integration pattern. Source: [WhisperKit GitHub](https://github.com/argmaxinc/WhisperKit), [ICML 2025 paper](https://arxiv.org/html/2507.10860v1).

- **faster-whisper** benchmarks poorly on Apple Silicon (~7.0s for large model on M4) because it uses CTranslate2 which is CPU-only on macOS (no Metal/MPS support). Not recommended for this setup. Source: [mac-whisper-speedtest benchmarks](https://github.com/anvanvan/mac-whisper-speedtest).

- **OpenClaw's native STT integration** supports a `type: "cli"` model entry in `tools.media.audio.models`. This means any CLI binary that accepts a file path and outputs text can be used. whisper.cpp's `whisper-cli` fits this pattern directly. A community skill ([local-whisper-openclaw-skill](https://github.com/kesslerio/local-whisper-openclaw-skill)) and a guide on [termo.ai](https://termo.ai/skills/whisper-cpp) document this exact setup. Source: [OpenClaw config reference](https://docs.openclaw.ai/gateway/configuration-reference), [OpenClaw GitHub issues #7573, #14374, #18424](https://github.com/openclaw/openclaw/issues/18424).

### text-to-speech (outbound)

- **Kokoro-82M** is a lightweight neural TTS model (82M parameters) that runs locally on Apple Silicon. Reports cite ~90ms latency on Mac, with quality rated A-tier for the best English voices (e.g., `af_heart`, `bf_alice`). Multiple deployment options exist: (a) **Kokoro-FastAPI** — an OpenAI-compatible `/v1/audio/speech` endpoint via Docker or direct `uv` run on port 8880, (b) **Kokori** — a macOS menubar app with built-in REST API on port 5002, (c) **kokoro-onnx** — a Python package for direct ONNX inference. Source: [Kokoro-FastAPI GitHub](https://github.com/remsky/Kokoro-FastAPI), [Kokori app](https://kokori.app/), [Reddit r/TextToSpeech](https://www.reddit.com/r/TextToSpeech/comments/1ijmbbs/kokoro82m_is_very_impressive_and_is_super_fast_on/).

- **Kokoro-FastAPI** is the integration-critical option because it exposes an **OpenAI-compatible endpoint**. OpenClaw's TTS config accepts a custom `providers.openai.baseUrl`. Pointing this at `http://localhost:8880/v1` would route all TTS through local Kokoro inference with zero code changes to OpenClaw. The CPU Docker image supports arm64/Apple Silicon. MPS (Metal GPU) acceleration is not yet available in the Docker build — inference runs on CPU via PyTorch, which is still fast for an 82M model. Source: [Kokoro-FastAPI GitHub](https://github.com/remsky/Kokoro-FastAPI).

- **Piper TTS** (from the Open Home Foundation) uses ONNX/VITS models. It now installs on macOS arm64 via `pip install piper-tts` (wheel published Feb 2026). Quality is described as better than macOS `say` but below Kokoro's neural quality. Historically had install friction on macOS (piper-phonemize dependency issues), now largely resolved. Source: [ThoughtAsylum blog](https://www.thoughtasylum.com/2025/08/25/text-to-speech-on-macos-with-piper/), [PyPI piper-tts](https://pypi.org/project/piper-tts/).

- **macOS built-in `say` command** is instant and zero-setup but sounds robotic. Not suitable for conversational replies that would be sent as Telegram voice notes. Source: general knowledge, confirmed by ThoughtAsylum blog comparison.

- **Microsoft Edge TTS** (via `node-edge-tts`) is already bundled in OpenClaw as a provider. It requires no API key and produces decent neural voices. However, it's a cloud service (Microsoft endpoints), not local. It's a good zero-cost fallback. Source: [OpenClaw TTS docs](https://docs.openclaw.ai/tools/tts).

- **OpenClaw's TTS config** lives at `messages.tts` in `openclaw.json`. It supports `auto: "inbound"` (speak back only when user sent voice), custom provider base URLs, voice selection, and timeout settings. The `/tts` slash command provides per-session control. Source: [OpenClaw TTS docs](https://docs.openclaw.ai/tools/tts), [LumaDock voice tutorial](https://lumadock.com/tutorials/openclaw-voice-tts-stt-talk-mode).

### integration architecture with OpenClaw

- **Inbound (voice → text):** OpenClaw's `tools.media.audio.models` accepts a `type: "cli"` entry. When a Telegram voice note arrives, OpenClaw downloads the audio, passes the file path to the CLI command, and uses the stdout as transcript text. Config:
  ```json5
  {
    tools: {
      media: {
        audio: {
          enabled: true,
          models: [
            {
              type: "cli",
              command: "/path/to/whisper-cli",
              args: ["-m", "/path/to/ggml-base.en.bin", "-f", "{{MediaPath}}"],
              timeoutSeconds: 30
            }
          ]
        }
      }
    }
  }
  ```

- **Outbound (text → voice):** OpenClaw's `messages.tts` supports custom OpenAI-compatible endpoints via `providers.openai.baseUrl`. Pointing at Kokoro-FastAPI (`http://localhost:8880/v1`) uses local TTS transparently. Config:
  ```json5
  {
    messages: {
      tts: {
        auto: "inbound",
        provider: "openai",
        providers: {
          openai: {
            baseUrl: "http://localhost:8880/v1",
            apiKey: "not-needed",
            model: "kokoro",
            voice: "bf_alice"
          }
        }
      }
    }
  }
  ```

- **Live mic / desktop loop:** OpenClaw's docs describe this as a "node feature" — a paired device (macOS, iOS, Android) handles the microphone, gateway handles model calls. For a Mac mini acting as both gateway and node, this could work but is a separate integration step beyond the Telegram voice-note MVP. Source: [LumaDock voice tutorial](https://lumadock.com/tutorials/openclaw-voice-tts-stt-talk-mode).

### latency budget for conversational loop

For a Telegram voice-note round trip (voice in → transcribe → LLM → TTS → voice out):

| Step | Estimated latency | Notes |
|------|------------------|-------|
| Telegram audio download | ~0.5s | Network-dependent |
| whisper.cpp (base.en, Metal) | ~0.3–0.5s | Short voice notes |
| LLM inference (API) | ~1–3s | Model-dependent |
| Kokoro TTS (local, CPU) | ~0.2–0.5s | Short reply text |
| Telegram upload | ~0.5s | Network-dependent |
| **Total** | **~2.5–5s** | Acceptable for async voice note UX |

For live-mic streaming (future), the LLM step dominates and would require streaming TTS to feel conversational.

## source tensions

- **whisper.cpp vs mlx-whisper speed:** The mac-whisper-speedtest benchmarks show mlx-whisper slightly faster than whisper.cpp on large models (~1.0s vs ~1.2s on M4), but whisper.cpp has a simpler integration path (standalone C binary, no Python runtime). For the CLI integration pattern OpenClaw uses, whisper.cpp is more practical despite the marginal speed difference.

- **Kokoro-FastAPI GPU acceleration on Apple Silicon:** The Kokoro-FastAPI README explicitly states CUDA is required for GPU mode and MPS support is "planned but not yet available." CPU inference on the 82M model is fast enough (~200ms for short text), but this is a gap worth monitoring. A Reddit thread from March 2026 mentions a project called "Murmur" running Kokoro natively via MLX on Apple Silicon, which would be faster, but this is not yet integrated into the FastAPI wrapper. Single-sourced claim.

- **OpenClaw CLI STT reliability:** GitHub issues #7573 and #14374 document cases where configured `tools.media.audio.models` entries were ignored in favor of the active chat model. Issue #7573 is from Feb 2026. The OpenClaw 2026.3.2 release notes mention `api.runtime.stt.transcribeAudioFile(...)` as a new runtime helper, suggesting active development in this area. The CLI path may require testing against the current OpenClaw version to confirm it works as documented.

- **Kokori app vs Kokoro-FastAPI:** Kokori is a polished macOS app with a simpler install (DMG download), but it's a paid product (license via LemonSqueezy) and its API runs on port 5002. Kokoro-FastAPI is free, open-source, and uses the OpenAI-compatible endpoint format that OpenClaw already supports. For OpenClaw integration, Kokoro-FastAPI is the more direct fit.

## recommended stack (for kishan's mac mini)

### MVP (can be set up today)

**STT:** whisper.cpp compiled with Metal support, using `base.en` model for speed or `small.en` for accuracy.

Install:
```bash
git clone https://github.com/ggml-org/whisper.cpp.git
cd whisper.cpp
sh ./models/download-ggml-model.sh base.en
cmake -B build
cmake --build build -j --config Release
# Binary at ./build/bin/whisper-cli
# Model at ./models/ggml-base.en.bin
```

**TTS:** Kokoro-FastAPI running as a local service on port 8880.

Install:
```bash
# Option A: Docker (simplest)
docker run -p 8880:8880 ghcr.io/remsky/kokoro-fastapi-cpu:v0.2.1

# Option B: Direct (faster startup)
git clone https://github.com/remsky/Kokoro-FastAPI.git
cd Kokoro-FastAPI
# Requires: uv (pip install uv), espeak-ng (brew install espeak-ng)
./start-cpu.sh
```

**OpenClaw config:** Combine both in `~/.openclaw/openclaw.json` using the config blocks shown in the integration section above.

**Fallback:** Add `microsoft` as a TTS fallback provider (no API key needed, already bundled in OpenClaw) and `openai/gpt-4o-mini-transcribe` as an STT fallback if an OpenAI key is available.

### Long-term architecture

- Upgrade to whisper.cpp with Core ML for ANE acceleration (3x speedup, requires generating Core ML model via Python script).
- Monitor Kokoro MLX native support (via Murmur or direct MLX port) for GPU-accelerated TTS on Apple Silicon.
- For live-mic conversational loop: explore WhisperKit (streaming ASR) + OpenClaw node pairing on the same Mac mini, or a companion iOS/macOS app.
- Consider whether a glue script is needed to manage the Kokoro-FastAPI service lifecycle (launchd plist for auto-start on boot).

### whether selena should build glue code

A launchd plist to auto-start Kokoro-FastAPI on boot would be useful (5-minute task). Beyond that, no custom glue code is needed for the MVP — OpenClaw's existing `tools.media.audio` CLI integration and `messages.tts` OpenAI-compatible provider config handle the wiring. If the CLI STT path has bugs in the current OpenClaw version (per the GitHub issues noted), a thin shell wrapper that normalizes whisper-cli output format might be needed.

## sources

1. whisper.cpp GitHub repository — https://github.com/ggml-org/whisper.cpp
2. mac-whisper-speedtest benchmarks (M4 24GB) — https://github.com/anvanvan/mac-whisper-speedtest
3. WhisperKit GitHub + ICML 2025 paper — https://github.com/argmaxinc/WhisperKit, https://arxiv.org/html/2507.10860v1
4. Kokoro-FastAPI GitHub — https://github.com/remsky/Kokoro-FastAPI
5. Kokori macOS app — https://kokori.app/
6. Piper TTS blog walkthrough — https://www.thoughtasylum.com/2025/08/25/text-to-speech-on-macos-with-piper/
7. OpenClaw TTS documentation — https://docs.openclaw.ai/tools/tts
8. OpenClaw configuration reference — https://docs.openclaw.ai/gateway/configuration-reference
9. LumaDock OpenClaw voice tutorial — https://lumadock.com/tutorials/openclaw-voice-tts-stt-talk-mode
10. OpenClaw GitHub issue #18424 (local Whisper STT feature request) — https://github.com/openclaw/openclaw/issues/18424
11. OpenClaw GitHub issue #7573 (audio model config ignored) — https://github.com/openclaw/openclaw/issues/7573
12. Reddit r/TextToSpeech Kokoro-82M thread — https://www.reddit.com/r/TextToSpeech/comments/1ijmbbs/kokoro82m_is_very_impressive_and_is_super_fast_on/

## related vault nodes

- `missions/general/research/openai-vs-anthropic-agentic-remix-pipeline.md` — overlaps on OpenAI API pricing and provider comparison context; the audio DSP pipeline research covers related API cost considerations.
