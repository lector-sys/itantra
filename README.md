# iTantra — Offline Voice-over-Text Communicator

SIH 2026 **SIH26173** (ISRO, Dept. of Space): *Indian Multilingual TTS & STT Aided Neural Transceiver Radio Access for Low-Bitrate Links*.

**Concept:** never transmit audio over the low-bitrate link. The sender runs offline speech-to-text on-device, transmits **text** (tens of bytes per sentence instead of ~100 KB/s audio) over Wi-Fi/Bluetooth, and the receiver speaks it back with offline text-to-speech. Voice communication that fits low-bitrate links and works for everyone — literate or not.

**Status: Phases 0–7 complete.** Fully offline Android app, 10 Indian languages, walkie-talkie + phone modes, non-interruptible alerts, live latency metrics, CSV benchmark export, ESP32 reference receiver.

## Quick start (Android Studio)

1. Open this folder in Android Studio (or use the Gradle wrapper + any JDK 21).
2. Run the `app` configuration on a device/emulator.
3. Sideload language packs (below). The Silero VAD ships inside the APK; STT/TTS models are sideloaded, keeping the APK ~24–34 MB.

## Language packs

**In-app download (easiest):** open the app → **Packs** tab → *Download packs* → Install.
Downloads from HuggingFace with progress + SHA-256 verification, then the app is fully offline.
(Handles HF rate limits with retry/backoff.)

**Via USB instead** (no in-app download needed / offline provisioning):

```
tools/.venv/Scripts/python tools/packs/build_pack.py --spec tools/packs/packs.json --all --out dist/packs
tools/android/install_packs.sh <phone-serial> multi-stt-omnilingual-300m-int8 en-tts-piper-lessac-medium hi-tts-mms
```

| Pack | Model | Licence |
|---|---|---|
| STT, all 10 languages | Omnilingual 300M CTC INT8 | Apache-2.0 |
| TTS en | Piper lessac | MIT + research dataset voice |
| TTS hi/gu/mr/kn/ml/ta/te/or/bn | MMS VITS per-language | **CC-BY-NC-4.0** (competition use; see LICENSES.md) |

## Verify everything (as CI would)

```
./gradlew testDebugUnitTest              # 17 unit tests: protocol, clock-sync, normaliser
tools/android/run_verification.sh        # instrumented model tests on emulator/device
```

Headless end-to-end on a running app (debug builds):

```
adb shell am broadcast -n isro.itantra/.DebugHookReceiver -a isro.itantra.DEBUG_HOST
adb shell am broadcast -n isro.itantra/.DebugHookReceiver -a isro.itantra.DEBUG_JOIN --es host <ip> --ei port <port>
adb shell am broadcast -n isro.itantra/.DebugHookReceiver -a isro.itantra.DEBUG_SEND --es lang hi --es text "…" --ez alert true
adb shell am broadcast -n isro.itantra/.DebugHookReceiver -a isro.itantra.DEBUG_STATE
adb shell am broadcast -n isro.itantra/.DebugHookReceiver -a isro.itantra.DEBUG_EXPORT
```

## Layout

| Path | Contents |
|---|---|
| `app/` | Compose UI + engines (STT/TTS/VAD), protocol, transports (TCP/BT), CommService, metrics |
| `docs/` | `MASTER_BUILD_PROMPT.md` (spec) · `MODELS.md` · `BENCHMARKS.md` · `LICENSES.md` · `PROTOCOL.md` · `DEMO_SCRIPT.md` · screenshots |
| `tools/eval/` | desktop benchmarks + FLEURS WER scripts and results |
| `tools/packs/` | pack builder (manifest.json + SHA-256 zips) |
| `firmware/esp32/` | open-source reference receiver (SPP, CRC, ACK, buzzer) |
| `dist/packs/` | ready-to-sideload language packs |

## Measured (emulator + desktop; phone numbers pending hardware)

- STT: RTF 0.06, Hindi/Odia/Tamil/Bengali native-script output (FLEURS CER 9–18%)
- TTS: first clause 48 ms (en) / 226–677 ms (Indic), RTF 0.04–0.21
- Link: RTT 2 ms on emulator NAT; screen-off receiving verified; alerts non-interruptible
- Release APK: 23.9 MB (armv7) / 33.5 MB (arm64) per-ABI + sideloaded packs

Honest gaps: on-phone benchmark tables (needs real devices), Room persistence deferred (in-memory history), BLE transport coded but not exercised on hardware, Indic TTS is CC-BY-NC (team sign-off recorded in LICENSES.md).
