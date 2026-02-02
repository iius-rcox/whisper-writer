# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WhisperWriter is a Python desktop speech-to-text application that records audio via hotkey, transcribes using local Whisper models or OpenAI API, and types the result into the active window.

## Commands

```bash
# Setup
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Linux/macOS
pip install -r requirements.txt

# Run
python run.py

# Find audio devices
python -m sounddevice
```

No test suite exists. No linter is configured.

## Architecture

### Entry Flow
`run.py` → loads .env → launches `src/main.py` → `WhisperWriterApp`

### Core Components

**WhisperWriterApp** (`src/main.py`) - Central orchestrator (QObject). Manages lifecycle, coordinates UI/key listener/transcription thread, handles system tray.

**KeyListener** (`src/key_listener.py`) - Multi-backend keyboard detection with:
- `EvdevBackend` - Linux (uses evdev library)
- `PynputBackend` - Cross-platform (uses pynput)
- `KeyChord` - Tracks multi-key combinations
- Callbacks: `on_activate`, `on_deactivate`

**ResultThread** (`src/result_thread.py`) - QThread for async recording/transcription:
- Uses sounddevice for audio capture
- webrtcvad for voice activity detection
- Emits `statusSignal` and `resultSignal`

**Transcription** (`src/transcription.py`) - Dual-mode transcription:
- `transcribe_local()` - faster-whisper library
- `transcribe_api()` - OpenAI API
- `post_process_transcription()` - text cleanup

**InputSimulator** (`src/input_simulation.py`) - Types text via pynput/ydotool/dotool

**ConfigManager** (`src/utils.py`) - Singleton config manager. Schema in `src/config_schema.yaml`, user config in `src/config.yaml`.

### UI Layer (PyQt5)
All windows inherit from `BaseWindow` (frameless, draggable, rounded corners):
- `MainWindow` - Start/Settings buttons
- `SettingsWindow` - Auto-generated from schema, saves to config.yaml
- `StatusWindow` - Recording/transcribing indicator (always on top)

### Data Flow
1. KeyListener detects activation chord → `on_activation()`
2. ResultThread records audio with VAD silence detection
3. Transcription (local or API) → post-processing
4. InputSimulator types result into active window

### Recording Modes
- `continuous` - Auto-restart after silence until key pressed again
- `voice_activity_detection` - Stop on silence
- `press_to_toggle` - Manual start/stop with same key
- `hold_to_record` - Record while key held

## Configuration

Schema-driven (`src/config_schema.yaml`). API key stored in `.env` as `OPENAI_API_KEY`, not in config.yaml.

Key sections:
- `model_options` - API vs local, model selection, compute settings
- `recording_options` - activation_key, recording_mode, silence_duration
- `post_processing` - typing delay, text transformations
- `misc` - terminal output, status window visibility

## Dependencies

- PyQt5 - UI framework
- faster-whisper - Local transcription
- openai - API transcription
- pynput - Input handling (keyboard/mouse)
- sounddevice - Audio capture
- webrtcvad-wheels - Voice activity detection

GPU support requires CUDA 12 + cuBLAS + cuDNN 8.
