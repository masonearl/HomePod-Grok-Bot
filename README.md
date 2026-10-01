# HomePod Grok Bot

Home Assistant + xAI Grok voice stack that uses HomePods as speakers.

## Scope

**This repo is** documentation and (later) packaging for a practical voice path: capture speech on a Home Assistant Assist satellite, run conversation / STT / TTS through Grok, and play the reply on a HomePod over AirPlay via Home Assistant’s Apple TV integration.

**This repo is not** a HomePod jailbreak, an on-device Grok runtime, or a replacement for Siri on the HomePod microphone. There is no realistic way to run Grok on a HomePod. The HomePod mic stays Siri-only; HomePod Grok Bot treats the speaker as an output device only.

This project is independent. It is not an official xAI or Apple product and claims no partnership with either.

## Architecture

```
Assist satellite mic  →  Home Assistant  →  Grok conversation / STT / TTS  →  media_player (HomePod)
```

1. A separate microphone (ESPHome / HA Assist satellite, or another Assist-capable client) hears the wake word and streams audio into Home Assistant.
2. Home Assistant runs the Assist pipeline: Grok STT transcribes, the Grok conversation agent answers (and can call HA tools), Grok TTS synthesizes speech.
3. Home Assistant plays the resulting audio on a HomePod `media_player` created by the Apple TV / AirPlay integration.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for a diagram and the TTS reliability note.

## Planned components / status

Scaffold only. No working custom integration ships in this first commit.

| Piece | Status | Notes |
| --- | --- | --- |
| Home Assistant config / packaging | TBD | HACS custom integration vs. a YAML config package is still open. This repo will pick one once there is something to install. |
| Grok conversation + TTS/STT | Reference existing work | Use the community HACS integration [braytonstafford/grok_conversation](https://github.com/braytonstafford/grok_conversation) (conversation agent, xAI Grok STT, xAI Grok TTS). Do not paste API keys into this repo. |
| HomePod as speaker | Reference core HA | [Apple TV integration](https://www.home-assistant.io/integrations/apple_tv/) exposes HomePods as `media_player` entities over AirPlay. |
| Microphone | Reference ESPHome / Assist | [ESPHome voice satellites](https://www.home-assistant.io/voice_control/s3_box_voice_assistant/) or other Assist satellites. The HomePod microphone is not in this path. |
| TTS → HomePod playback | Documented workaround | Direct `tts.speak` / TTS proxy URLs to HomePods are unreliable. Pre-generate the media, copy it to local media, then `media_player.play_media` a `media-source://` URI. See architecture doc. |

## Setup outline

High-level checklist. This is not a full install guide yet.

1. Run a working Home Assistant instance on the same network (or mDNS-reachable VLAN) as the HomePods.
2. Add each HomePod with the Apple TV integration so a `media_player` entity appears.
3. Add an Assist satellite (ESPHome or equivalent) for the microphone. Do not expect the HomePod mic to feed Assist.
4. Install [grok_conversation](https://github.com/braytonstafford/grok_conversation) via HACS, add the integration, and store the xAI API key in Home Assistant secrets — never in this git tree.
5. Point a Voice Assistant pipeline at Grok for conversation, STT, and TTS.
6. Confirm Grok replies in the Assist debug / “Start conversation” UI before involving speakers.
7. Play a short local MP3 to the HomePod `media_player` to prove AirPlay audio works.
8. For spoken replies, pre-generate TTS into `/media`, then `play_media` the local file. Skip sending the temporary `/api/tts_proxy/` URL straight to the HomePod.

## License

MIT. See [LICENSE](LICENSE).
