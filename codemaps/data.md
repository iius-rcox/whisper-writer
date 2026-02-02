# WhisperWriter Data Models

**Freshness:** 2026-02-02

## Configuration Schema

Source: `src/config_schema.yaml`

### model_options

```yaml
use_api: bool (false)        # Toggle API vs local
common:
  language: str|null         # ISO-639-1 code
  temperature: float (0.0)   # Output randomness
  initial_prompt: str|null   # Conditioning prompt
api:
  model: str (whisper-1)     # API model name
  base_url: str              # API endpoint
  api_key: str|null          # Stored in .env
local:
  model: str (base)          # tiny/base/small/medium/large
  device: str (auto)         # auto/cuda/cpu
  compute_type: str (default) # float32/float16/int8
  condition_on_previous_text: bool (true)
  vad_filter: bool (false)
  model_path: str|null       # Custom model location
```

### recording_options

```yaml
activation_key: str (ctrl+shift+space)  # Hotkey combo
input_backend: str (auto)    # auto/evdev/pynput
recording_mode: str (continuous)
  # continuous|voice_activity_detection|press_to_toggle|hold_to_record
sound_device: str|null       # Device index
sample_rate: int (16000)     # Hz
silence_duration: int (900)  # ms before stop
min_duration: int (100)      # ms minimum recording
```

### post_processing

```yaml
writing_key_press_delay: float (0.005)  # Seconds
remove_trailing_period: bool (false)
add_trailing_space: bool (true)
remove_capitalization: bool (false)
input_method: str (pynput)   # pynput/ydotool/dotool
```

### misc

```yaml
print_to_terminal: bool (true)
hide_status_window: bool (false)
noise_on_completion: bool (false)
```

## Runtime Data Structures

### Audio Data

```python
# Recording buffer
audio_data: np.ndarray[np.int16]  # Raw PCM samples
sample_rate: int                   # Typically 16000 Hz
frame_size: int                    # 30ms frames for VAD

# For API transcription
byte_io: io.BytesIO               # WAV-encoded audio
```

### Key Events

```python
class InputEvent(Enum):
    KEY_PRESS = auto()
    KEY_RELEASE = auto()
    MOUSE_PRESS = auto()
    MOUSE_RELEASE = auto()

class KeyCode(Enum):
    # 190+ key codes covering:
    # - Modifiers (CTRL_LEFT, SHIFT_RIGHT, etc.)
    # - Function keys (F1-F24)
    # - Letters (A-Z)
    # - Numbers (ZERO-NINE, NUMPAD_0-9)
    # - Special keys (SPACE, ENTER, ESC, etc.)
    # - Media keys (PLAY_PAUSE, VOLUME_*, etc.)
    # - Mouse buttons (MOUSE_LEFT, etc.)
```

### Key Chord State

```python
class KeyChord:
    keys: Set[KeyCode | frozenset[KeyCode]]  # Required keys
    pressed_keys: Set[KeyCode]                # Currently held
```

## File Locations

| File | Purpose | Format |
|------|---------|--------|
| `src/config_schema.yaml` | Default values + metadata | YAML |
| `src/config.yaml` | User overrides | YAML |
| `.env` | API key storage | dotenv |

## Signals & Events

### Qt Signals

| Class | Signal | Payload |
|-------|--------|---------|
| MainWindow | openSettings | - |
| MainWindow | startListening | - |
| MainWindow | closeApp | - |
| SettingsWindow | settings_closed | - |
| SettingsWindow | settings_saved | - |
| StatusWindow | statusSignal | str |
| StatusWindow | closeSignal | - |
| ResultThread | statusSignal | str ('recording'/'transcribing'/'idle'/'error') |
| ResultThread | resultSignal | str (transcription text) |

### KeyListener Callbacks

| Event | When |
|-------|------|
| on_activate | Key chord pressed |
| on_deactivate | Key chord released |
