# WhisperWriter Backend

**Freshness:** 2026-02-02

## Module Structure

```
src/
├── main.py           # App orchestration
├── key_listener.py   # Input detection
├── result_thread.py  # Recording & transcription
├── transcription.py  # Speech-to-text engines
├── input_simulation.py # Keyboard output
└── utils.py          # Configuration
```

## Key Classes

### WhisperWriterApp (`main.py:20`)
Main application controller.

| Method | Purpose |
|--------|---------|
| `initialize_components()` | Setup all subsystems |
| `on_activation()` | Handle hotkey press |
| `on_deactivation()` | Handle hotkey release |
| `start_result_thread()` | Begin recording |
| `on_transcription_complete()` | Type result |

### KeyListener (`key_listener.py:274`)
Hotkey detection with pluggable backends.

| Backend | Platform | Priority |
|---------|----------|----------|
| EvdevBackend | Linux | 1st |
| PynputBackend | All | 2nd |

Key classes:
- `KeyCode` (enum) - Normalized key identifiers
- `InputEvent` (enum) - KEY_PRESS, KEY_RELEASE, MOUSE_*
- `KeyChord` - Multi-key combination tracker
- `InputBackend` (ABC) - Backend interface

### ResultThread (`result_thread.py:16`)
QThread for async audio processing.

| Signal | Payload | When |
|--------|---------|------|
| `statusSignal` | str | State changes |
| `resultSignal` | str | Transcription done |

Recording modes:
- `continuous` - Auto-restart after silence
- `voice_activity_detection` - Stop on silence
- `press_to_toggle` - Manual start/stop
- `hold_to_record` - Record while pressed

### Transcription Module (`transcription.py`)

| Function | Engine |
|----------|--------|
| `create_local_model()` | faster-whisper |
| `transcribe_local()` | faster-whisper |
| `transcribe_api()` | OpenAI API |
| `transcribe()` | Router (config-based) |
| `post_process_transcription()` | Text cleanup |

Local model options:
- Models: tiny, base, small, medium, large variants
- Devices: auto, cuda, cpu
- Compute: default, float32, float16, int8
- VAD filter support

### InputSimulator (`input_simulation.py:22`)
Text output via system input simulation.

| Method | Backend |
|--------|---------|
| `_typewrite_pynput()` | pynput.keyboard |
| `_typewrite_ydotool()` | ydotool CLI |
| `_typewrite_dotool()` | dotool daemon |

### ConfigManager (`utils.py:4`)
Singleton configuration manager.

| Method | Purpose |
|--------|---------|
| `initialize()` | Load schema + user config |
| `get_config_value(*keys)` | Nested key lookup |
| `set_config_value(val, *keys)` | Nested key update |
| `save_config()` | Write to config.yaml |
| `console_print()` | Conditional logging |

## Threading Model

```
Main Thread (Qt Event Loop)
├── UI Windows
├── System Tray
└── Signal/Slot dispatch

ResultThread (QThread)
├── Audio recording (sounddevice callback)
├── VAD processing (webrtcvad)
└── Transcription (local/API)

KeyListener Backend Thread
├── EvdevBackend: select() loop
└── PynputBackend: pynput.Listener threads
```

## External Integrations

| Service | Module | Auth |
|---------|--------|------|
| OpenAI API | transcription.py | OPENAI_API_KEY env |
| Local Whisper | transcription.py | N/A |
| HuggingFace Hub | faster-whisper | Auto-download |
