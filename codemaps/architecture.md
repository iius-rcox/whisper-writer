# WhisperWriter Architecture

**Freshness:** 2026-02-02

## Overview

WhisperWriter is a Python desktop application for real-time speech-to-text transcription. It captures audio via hotkey activation, transcribes using local Whisper models or OpenAI API, and types the result into the active application.

## Technology Stack

| Layer | Technology |
|-------|------------|
| UI Framework | PyQt5 |
| Audio Capture | sounddevice, webrtcvad |
| Transcription | faster-whisper (local), OpenAI API |
| Input Simulation | pynput, ydotool, dotool |
| Configuration | YAML (PyYAML) |

## High-Level Architecture

```
+------------------+     +------------------+     +------------------+
|   UI Layer       |     |  Core Logic      |     |  Services        |
|------------------|     |------------------|     |------------------|
| MainWindow       |<--->| WhisperWriterApp |<--->| Transcription    |
| SettingsWindow   |     | KeyListener      |     | InputSimulator   |
| StatusWindow     |     | ResultThread     |     | ConfigManager    |
+------------------+     +------------------+     +------------------+
```

## Entry Points

| File | Purpose |
|------|---------|
| `run.py` | Bootstrap script - loads .env, launches main.py |
| `src/main.py` | Application entry - initializes WhisperWriterApp |

## Core Components

### WhisperWriterApp (`src/main.py`)
- Central orchestrator (QObject)
- Manages lifecycle: init -> start -> record -> transcribe -> type
- Coordinates UI windows, key listener, transcription thread
- Handles system tray integration

### KeyListener (`src/key_listener.py`)
- Multi-backend keyboard/mouse event handling
- Backends: EvdevBackend (Linux), PynputBackend (cross-platform)
- KeyChord system for configurable activation sequences
- Event callback system (on_activate, on_deactivate)

### ResultThread (`src/result_thread.py`)
- QThread for async audio recording and transcription
- Voice Activity Detection (VAD) for automatic stop
- Multiple recording modes: continuous, VAD, toggle, hold
- Signals: statusSignal, resultSignal

### Transcription (`src/transcription.py`)
- Dual-mode: local (faster-whisper) or API (OpenAI)
- Post-processing: trim, capitalization, spacing
- Auto-fallback to CPU on GPU errors

### InputSimulator (`src/input_simulation.py`)
- Multi-backend text typing: pynput, ydotool, dotool
- Configurable keystroke delay

### ConfigManager (`src/utils.py`)
- Singleton pattern for global config access
- Schema-driven validation (config_schema.yaml)
- User config merges with defaults

## Data Flow

```
1. User presses activation key
   └─> KeyListener detects chord
       └─> WhisperWriterApp.on_activation()

2. Recording starts
   └─> ResultThread._record_audio()
       └─> sounddevice InputStream
       └─> webrtcvad silence detection

3. Recording stops (VAD/key release/toggle)
   └─> ResultThread emits statusSignal('transcribing')

4. Transcription
   └─> transcribe_local() or transcribe_api()
       └─> post_process_transcription()

5. Result typed
   └─> ResultThread emits resultSignal(text)
       └─> InputSimulator.typewrite()
```

## Configuration

Schema-defined in `src/config_schema.yaml`:
- `model_options`: API vs local, model selection, compute settings
- `recording_options`: hotkey, mode, audio device, VAD settings
- `post_processing`: typing delay, text transformations
- `misc`: terminal output, status window, completion sound

## Dependencies

Core dependencies from requirements.txt:
- `PyQt5==5.15.10` - UI framework
- `faster-whisper==1.0.2` - Local transcription
- `openai==1.28.0` - API transcription
- `pynput==1.7.6` - Input handling
- `sounddevice==0.4.6` - Audio capture
- `webrtcvad-wheels==2.0.11` - Voice activity detection
- `python-dotenv==1.0.0` - Environment variables
