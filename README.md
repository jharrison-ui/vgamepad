# Virtual Gamepad

Virtual XBox360 and DualShock4 gamepads in Python.

---

Virtual Gamepad (`vgamepad`) is a small Python library that emulates XBox360 and DualShock4 gamepads on your system. It enables controlling e.g. a video game that requires analog input directly from your Python script.

- **Windows:** `vgamepad` uses the [Virtual Gamepad Emulation (ViGEmBus)](https://github.com/nefarius/ViGEmBus) C++ framework, providing Python bindings and a user-friendly interface. The ViGEmBus driver (`v1.17.333`) is bundled with the library and installs automatically.
- **Linux:** `vgamepad` creates a `uinput` device and requires the corresponding system permissions.

---

## Quick Links

- [Installation](#installation)
  - [Windows](#windows)
  - [Linux](#linux)
- [Getting Started](#getting-started)
  - [XBox360 Gamepad](#xbox360-gamepad)
  - [DualShock4 Gamepad](#dualshock4-gamepad)
  - [Rumble and LEDs](#rumble-and-leds)
- [Advanced Users](#advanced-users)
- [Testing](#testing)
- [Contribute](#contribute)
- [Authors](#authors)

---

## Installation

### Windows

Open your preferred terminal (e.g. Anaconda Prompt) and run:

```bash
pip install vgamepad
```

This automatically launches the ViGEmBus driver installer. Accept the license agreement, click **Install**, allow the installer to modify your PC, wait for completion, and click **Finish**.

`vgamepad` is now installed in your active Python environment.

### Linux

#### Prerequisites

On Linux, `vgamepad` requires access to `/dev/uinput`.

To grant permission for the current session only:

```bash
sudo chmod +0666 /dev/uinput
```

To configure permissions permanently across reboots, create a `udev` rule:

```bash
sudo nano /etc/udev/rules.d/50-uinput.rules
```

Add the following line:

```udev
KERNEL=="uinput", TAG+="uaccess"
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### Installing vgamepad

Run:

```bash
pip install vgamepad
```

`vgamepad` is now installed in your active Python environment.

---

## Getting Started

`vgamepad` exposes two primary classes:
- `VX360Gamepad` — emulates an XBox360 controller.
- `VDS4Gamepad` — emulates a DualShock4 controller.

Both virtual devices are recognized by your operating system identically to physical gamepads and can be fully controlled programmatically.

The state of a virtual gamepad (buttons, triggers, joysticks) is represented as a **report**. Modify the report using API methods, then push updates to the operating system using `gamepad.update()`.

---

### XBox360 Gamepad

#### Initialization

```python
import vgamepad as vg

gamepad = vg.VX360Gamepad()
```

The virtual gamepad connects immediately upon instantiation and remains connected until the object is destroyed.

#### Buttons

Buttons can be toggled using `press_button` and `release_button`:

```python
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)          # Press A
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_DPAD_LEFT)  # Press D-Pad Left

gamepad.update()  # Send state to system

# (...) A and D-Pad Left are currently pressed

gamepad.release_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)        # Release A

gamepad.update()  # Send state to system
```

Available button definitions in `vg.XUSB_BUTTON`:

```python
class XUSB_BUTTON(IntFlag):
    XUSB_GAMEPAD_DPAD_UP        = 0x0001
    XUSB_GAMEPAD_DPAD_DOWN      = 0x0002
    XUSB_GAMEPAD_DPAD_LEFT      = 0x0004
    XUSB_GAMEPAD_DPAD_RIGHT     = 0x0008
    XUSB_GAMEPAD_START          = 0x0010
    XUSB_GAMEPAD_BACK           = 0x0020
    XUSB_GAMEPAD_LEFT_THUMB     = 0x0040
    XUSB_GAMEPAD_RIGHT_THUMB    = 0x0080
    XUSB_GAMEPAD_LEFT_SHOULDER  = 0x0100
    XUSB_GAMEPAD_RIGHT_SHOULDER = 0x0200
    XUSB_GAMEPAD_GUIDE          = 0x0400
    XUSB_GAMEPAD_A              = 0x1000
    XUSB_GAMEPAD_B              = 0x2000
    XUSB_GAMEPAD_X              = 0x4000
    XUSB_GAMEPAD_Y              = 0x8000
```

#### Triggers and Joysticks

Values can be supplied as raw integers:

```python
gamepad.left_trigger(value=100)                        # 0 to 255
gamepad.right_trigger(value=255)                       # 0 to 255
gamepad.left_joystick(x_value=-10000, y_value=0)       # -32768 to 32767
gamepad.right_joystick(x_value=-32768, y_value=15000)  # -32768 to 32767

gamepad.update()
```

Or normalized floats:

```python
gamepad.left_trigger_float(value_float=0.5)                        # 0.0 to 1.0
gamepad.right_trigger_float(value_float=1.0)                       # 0.0 to 1.0
gamepad.left_joystick_float(x_value_float=-0.5, y_value_float=0.0) # -1.0 to 1.0
gamepad.right_joystick_float(x_value_float=-1.0, y_value_float=0.8)# -1.0 to 1.0

gamepad.update()
```

#### Resetting State

```python
gamepad.reset()
gamepad.update()
```

#### Full XBox360 Example

```python
import time
import vgamepad as vg

gamepad = vg.VX360Gamepad()

# Wake up device
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)
gamepad.update()
time.sleep(0.5)
gamepad.release_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)
gamepad.update()
time.sleep(0.5)

# Apply inputs
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_LEFT_SHOULDER)
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_DPAD_DOWN)
gamepad.press_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_DPAD_LEFT)
gamepad.left_trigger_float(value_float=0.5)
gamepad.right_trigger_float(value_float=0.5)
gamepad.left_joystick_float(x_value_float=0.0, y_value_float=0.2)
gamepad.right_joystick_float(x_value_float=-1.0, y_value_float=1.0)
gamepad.update()

time.sleep(1.0)

# Release inputs
gamepad.release_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_A)
gamepad.release_button(button=vg.XUSB_BUTTON.XUSB_GAMEPAD_DPAD_LEFT)
gamepad.right_trigger_float(value_float=0.0)
gamepad.right_joystick_float(x_value_float=0.0, y_value_float=0.0)
gamepad.update()

time.sleep(1.0)

# Reset back to default
gamepad.reset()
gamepad.update()
```

---

### DualShock4 Gamepad

The `VDS4Gamepad` class mirrors the XBox360 interface while offering DualShock4-exclusive capabilities:

```python
import vgamepad as vg

gamepad = vg.VDS4Gamepad()
```

#### Standard Digital Buttons

```python
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.update()

# (...)

gamepad.release_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.update()
```

Available buttons in `vg.DS4_BUTTONS`:

```python
class DS4_BUTTONS(IntFlag):
    DS4_BUTTON_THUMB_RIGHT    = 1 << 15
    DS4_BUTTON_THUMB_LEFT     = 1 << 14
    DS4_BUTTON_OPTIONS        = 1 << 13
    DS4_BUTTON_SHARE          = 1 << 12
    DS4_BUTTON_TRIGGER_RIGHT  = 1 << 11
    DS4_BUTTON_TRIGGER_LEFT   = 1 << 10
    DS4_BUTTON_SHOULDER_RIGHT = 1 << 9
    DS4_BUTTON_SHOULDER_LEFT  = 1 << 8
    DS4_BUTTON_TRIANGLE       = 1 << 7
    DS4_BUTTON_CIRCLE         = 1 << 6
    DS4_BUTTON_CROSS          = 1 << 5
    DS4_BUTTON_SQUARE         = 1 << 4
```

#### Special Buttons (PS and Touchpad Click)

```python
gamepad.press_special_button(special_button=vg.DS4_SPECIAL_BUTTONS.DS4_SPECIAL_BUTTON_PS)
gamepad.update()

# (...)

gamepad.release_special_button(special_button=vg.DS4_SPECIAL_BUTTONS.DS4_SPECIAL_BUTTON_PS)
gamepad.update()
```

Available special buttons in `vg.DS4_SPECIAL_BUTTONS`:

```python
class DS4_SPECIAL_BUTTONS(IntFlag):
    DS4_SPECIAL_BUTTON_PS       = 1 << 0
    DS4_SPECIAL_BUTTON_TOUCHPAD = 1 << 1
```

#### Triggers and Joysticks

Raw integer inputs:

```python
gamepad.left_trigger(value=100)                # 0 to 255
gamepad.right_trigger(value=255)               # 0 to 255
gamepad.left_joystick(x_value=0, y_value=128)  # 0 to 255
gamepad.right_joystick(x_value=0, y_value=255) # 0 to 255

gamepad.update()
```

Normalized float inputs:

```python
gamepad.left_trigger_float(value_float=0.5)                         # 0.0 to 1.0
gamepad.right_trigger_float(value_float=1.0)                        # 0.0 to 1.0
gamepad.left_joystick_float(x_value_float=-0.5, y_value_float=0.0)  # -1.0 to 1.0
gamepad.right_joystick_float(x_value_float=-1.0, y_value_float=0.8) # -1.0 to 1.0

gamepad.update()
```

> **Note:** Since version `0.1.0`, the DS4 Y-axis on joysticks is inverted relative to the X360 API (matching native ViGEmBus behavior).

#### Directional Pad (Hat)

```python
gamepad.directional_pad(direction=vg.DS4_DPAD_DIRECTIONS.DS4_BUTTON_DPAD_NORTHWEST)
gamepad.update()
```

Available values in `vg.DS4_DPAD_DIRECTIONS`:

```python
class DS4_DPAD_DIRECTIONS(IntEnum):
    DS4_BUTTON_DPAD_NONE      = 0x8
    DS4_BUTTON_DPAD_NORTHWEST = 0x7
    DS4_BUTTON_DPAD_WEST      = 0x6
    DS4_BUTTON_DPAD_SOUTHWEST = 0x5
    DS4_BUTTON_DPAD_SOUTH     = 0x4
    DS4_BUTTON_DPAD_SOUTHEAST = 0x3
    DS4_BUTTON_DPAD_EAST      = 0x2
    DS4_BUTTON_DPAD_NORTHEAST = 0x1
    DS4_BUTTON_DPAD_NORTH     = 0x0
```

#### Motion Sensors (Accelerometer & Gyroscope)

```python
gamepad.accelerometer(x_value=3000, y_value=1000, z_value=0)     # -32768 to 32767
gamepad.gyroscope(x_value=0, y_value=10000, z_value=-5000)       # -32768 to 32767

gamepad.update()
```

#### Touchpad Surface

```python
# (0, 0) is top-left, (1919, 941) is bottom-right
gamepad.touchpad(finger1=(1800, 800), finger2=None)

gamepad.update()
```

#### Full DualShock4 Example

```python
import time
import vgamepad as vg

gamepad = vg.VDS4Gamepad()

# Wake up device
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.update()
time.sleep(0.5)
gamepad.release_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.update()
time.sleep(0.5)

# Apply inputs
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_CIRCLE)
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_THUMB_RIGHT)
gamepad.press_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIGGER_LEFT)
gamepad.press_special_button(special_button=vg.DS4_SPECIAL_BUTTONS.DS4_SPECIAL_BUTTON_TOUCHPAD)
gamepad.left_trigger_float(value_float=0.5)
gamepad.right_trigger_float(value_float=0.5)
gamepad.left_joystick_float(x_value_float=0.0, y_value_float=0.2)
gamepad.right_joystick_float(x_value_float=-1.0, y_value_float=1.0)
gamepad.accelerometer(x_value=3000, y_value=1000, z_value=0)
gamepad.gyroscope(x_value=0, y_value=10000, z_value=-5000)
gamepad.touchpad(finger1=(1800, 800), finger2=None)
gamepad.update()

time.sleep(1.0)

# Release inputs
gamepad.release_button(button=vg.DS4_BUTTONS.DS4_BUTTON_TRIANGLE)
gamepad.right_trigger_float(value_float=0.0)
gamepad.right_joystick_float(x_value_float=0.0, y_value_float=0.0)
gamepad.accelerometer(x_value=0, y_value=0, z_value=0)
gamepad.gyroscope(x_value=0, y_value=0, z_value=0)
gamepad.touchpad(finger1=None, finger2=None)
gamepad.update()

time.sleep(1.0)

# Reset back to default
gamepad.reset()
gamepad.update()
```

---

### Rumble and LEDs

`vgamepad` allows registering callback functions to capture updates for rumble motors and LED indicators sent from the host/game.

The callback function requires exactly 6 parameters:

```python
def my_callback(client, target, large_motor, small_motor, led, user_data):
    """
    Callback function triggered at each received state change.

    :param client: ViGEm bus ID (Windows) or None (Linux)
    :param target: ViGEm device ID (Windows) or None (Linux)
    :param large_motor: integer in [0, 255] representing large motor intensity
    :param small_motor: integer in [0, 255] representing small motor intensity
    :param led: integer in [0, 255] (X360) or a vg.DS4_LIGHTBAR_COLOR structure (DS4),
                with .Red, .Green, and .Blue fields in [0, 255]
    :param user_data: placeholder, do not use
    """
    print(f"Received notification for client {client}, target {target}")
    print(f"large motor: {large_motor}, small motor: {small_motor}")
    print(f"led: {led}")
```

Register the callback:

```python
gamepad.register_notification(callback_function=my_callback)
```

Each time the state of the gamepad is changed (for example, by a video game dispatching rumble commands), the callback function will be executed.

**Notes:**
- The callback must have exactly this 6-parameter signature; otherwise, `register_notification` raises a `TypeError`.
- Only one callback can be registered per gamepad (a second call raises `RuntimeError`); invoke `unregister_notification()` first.
- On Linux, `client` and `target` are always `None`, and the callback must not call methods of the gamepad.

Example output on Windows written to `stdout`:

```terminal
Received notification for client 2876897124288, target 2876931874736
large motor: 255, small motor: 255
led: 0
Received notification for client 2876897124288, target 2876931874736
large motor: 0, small motor: 0
led: 0
```

To stop receiving notifications:

```python
gamepad.unregister_notification()
```

---

## Advanced Users

More API functions are available for advanced use cases. Refer to [api.py](https://github.com/yannbouteiller/vgamepad/blob/main/vgamepad/api.py).

To skip the automated installation of the `ViGEmBus` driver during `vgamepad` installation on Windows, set the environment variable `VGAMEPAD_SKIP_VIGEMBUS_INSTALL=true` before running `pip install vgamepad`.

> **Note:** Only `ViGEmBus 1.17.333.0` is officially tested and supported on Windows.

---

## Testing

- **Basic X360 & DS4 input testing:** [hardwaretester.com/gamepad](https://hardwaretester.com/gamepad)
- **Advanced DS4 features (Windows only):** [dualshock-tools.github.io](https://dualshock-tools.github.io)

---

## Contribute

Contributions are welcome! Please submit a pull request including your name and a brief summary of your contribution in the Contributors list.

---

## Authors

### Maintainer
- **Yann Bouteiller**

### Contributors
- **JumpyzZ** (rumble and LEDs)
- **willRicard** (Linux support)
- **Palamabron** (Linux support, advanced DS4 support)
