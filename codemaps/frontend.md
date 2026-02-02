# WhisperWriter Frontend (UI)

**Freshness:** 2026-02-02

## UI Framework

PyQt5-based desktop application with custom frameless windows.

## Window Hierarchy

```
QMainWindow (BaseWindow)
├── MainWindow      - Primary interface (Start/Settings)
├── SettingsWindow  - Configuration editor with tabs
└── StatusWindow    - Recording/transcription indicator
```

## Components

### BaseWindow (`src/ui/base_window.py`)
Abstract base for all windows.

| Feature | Implementation |
|---------|----------------|
| Style | Frameless, translucent, rounded corners (20px) |
| Dragging | Custom mouse event handling |
| Title bar | Custom with close button |
| Size | Fixed per subclass |

Signals: None (handled by subclasses)

### MainWindow (`src/ui/main_window.py`)
Primary user interface. Size: 320x180

| Signal | Trigger |
|--------|---------|
| `openSettings` | Settings button clicked |
| `startListening` | Start button clicked |
| `closeApp` | Window close event |

Behavior: Hides on Start (minimizes to tray)

### SettingsWindow (`src/ui/settings_window.py`)
Dynamic configuration editor. Size: 700x700

| Signal | Trigger |
|--------|---------|
| `settings_closed` | Window closed without save |
| `settings_saved` | Settings saved (triggers app restart) |

Features:
- Auto-generated from config_schema.yaml
- Tabbed interface per category
- Widget types: QCheckBox, QComboBox, QLineEdit
- Help tooltips per setting
- API key stored in .env (not config.yaml)
- Conditional visibility (API vs local options)

### StatusWindow (`src/ui/status_window.py`)
Floating status indicator. Size: 320x120

| Signal | Trigger |
|--------|---------|
| `statusSignal` | Status update from ResultThread |
| `closeSignal` | Window closed by user |

States:
- `recording` - Shows microphone icon
- `transcribing` - Shows pencil icon
- `idle/error/cancel` - Closes window

Features:
- Always on top (WindowStaysOnTopHint)
- Bottom-center screen positioning
- Tool window (no taskbar entry)

## System Tray

Managed by WhisperWriterApp:
- Icon: `assets/ww-logo.png`
- Menu: Show Main Menu, Open Settings, Exit

## Assets

| File | Usage |
|------|-------|
| `ww-logo.png` | Window icon, tray icon |
| `ww-logo.ico` | Windows taskbar icon |
| `microphone.png` | Recording status |
| `pencil.png` | Transcribing status |
| `beep.wav` | Completion sound |

## Styling

Inline CSS via setStyleSheet():
- Background: rgba(255, 255, 255, 220)
- Font: Segoe UI
- Text color: #404040
- Buttons: Transparent with hover effects
