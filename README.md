#  Assignment: Refactor & Extend SmartApp IoT System

##  Objective

Refactor the existing SmartApp IoT project to follow good software design principles. You'll improve maintainability, flexibility, and scalability by:

1. **Refactoring to a controller class** (separating logic from UI)
2. **Implementing design patterns**: Decorator and Facade
3. **Migrating from Tkinter GUI to a web-based interface** using **FastAPI** with **MVT (Model-View-Template)** pattern.

---

##  Part 1: Refactor Using a Controller Class

### Goal

Move all device logic and state management out of the `SmartApp` Tkinter class and into a **controller class**, leaving only UI event wiring in `SmartApp`.

###  Tasks

- Create a new module: `controller/app_controller.py`
- Move the following logic to the controller:
  - Device registration
  - Toggling the speaker
  - Getting status updates
  - Connection and messaging
- Update `SmartApp` to use this controller for interactions

###  Starter Code

#### `controller/app_controller.py`

```python
class AppController:
    def __init__(self, service):
        self.service = service
        self.speaker_on = False
        self.speaker_id = self.service.register_device(SmartSpeakerDevice())

    def toggle_speaker(self) -> bool:
        self.speaker_on = not self.speaker_on
        speaker = self.service.get_device(self.speaker_id)
        ip, port = speaker.connection_info()
        conn = Connection(ip, port)
        message = Msg("SERVER", self.speaker_id, "switch_on" if self.speaker_on else "switch_off")

        conn.connect()
        conn.send(message.b64)
        conn.disconnect()

        return self.speaker_on

    def get_status(self) -> str:
        status = ""
        for device_id, device in self.service.devices().items():
            status += f"{device_id}: {device.status_update()}\n"
        return status.strip()
````

#### Updated usage in `SmartApp`

```python
self.controller = AppController(self.service)

def toggle(self) -> None:
    speaker_status = self.controller.toggle_speaker()
    self.toggle_button.config(text="Speaker ON" if speaker_status else "Speaker OFF")
```

---

##  Part 2: Implement Design Patterns

###  Goal

Improve the system design by applying:

* The **Decorator pattern** to extend device behaviors
* The **Facade pattern** to simplify complex service/device interactions

---

###  2.1: Decorator Pattern

Wrap a device with a decorator to dynamically add behavior like logging, monitoring, or authentication.

####  Example

```python
class DeviceDecorator(Device):
    def __init__(self, device: Device):
        self._device = device

    def connect(self):
        print("[Decorator] Logging device connection.")
        self._device.connect()

    def disconnect(self):
        self._device.disconnect()

    def connection_info(self):
        return self._device.connection_info()

    def status_update(self):
        return self._device.status_update()
```

##### Usage:

```python
decorated_speaker = DeviceDecorator(SmartSpeakerDevice())
```

---

### 2.2: Facade Pattern

Create a simplified interface to register devices, toggle states, and get statuses, hiding complex logic.

#### Example

```python
class IOTFacade:
    def __init__(self, service: IOTService):
        self.service = service
        self.devices = {}

    def register(self, device: Device) -> str:
        device_id = self.service.register_device(device)
        self.devices[device_id] = device
        return device_id

    def toggle_device(self, device_id: str, turn_on: bool):
        device = self.service.get_device(device_id)
        ip, port = device.connection_info()
        conn = Connection(ip, port)
        message = Msg("SERVER", device_id, "switch_on" if turn_on else "switch_off")

        conn.connect()
        conn.send(message.b64)
        conn.disconnect()
```

---

## Part 3: Migrate to Web UI (MVT with FastAPI)

### Goal

Replace the Tkinter-based desktop UI with a **web-based UI** using:

* **FastAPI** for backend API (View)
* **Jinja2** for HTML templates (Template)
* **Device/service classes** as your Model

---

### Tasks

* Replace `SmartApp` with a web app (FastAPI server)
* Create HTML template for frontend
* Create FastAPI endpoints for:

  * Toggling speaker
  * Getting device statuses
* Structure the app in MVT format

---

### Recommended Directory Structure

```
/iot
/controller
/web
  ├── templates/
  │     └── index.html
  └── app.py
```

---

### `web/app.py` (FastAPI server)

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates

from iot.service import IOTService
from iot.devices import SmartSpeakerDevice
from controller.app_controller import AppController

app = FastAPI()
templates = Jinja2Templates(directory="web/templates")

controller = AppController(IOTService())

@app.get("/", response_class=HTMLResponse)
def read_root(request: Request):
    status = controller.get_status()
    return templates.TemplateResponse("index.html", {"request": request, "status": status})

@app.post("/toggle")
def toggle():
    state = controller.toggle_speaker()
    return {"state": state}
```

---

### `web/templates/index.html` (UI Template)

```html
<!DOCTYPE html>
<html>
<head>
    <title>SmartApp</title>
</head>
<body>
    <h1>Smart Speaker</h1>
    <form action="/toggle" method="post">
        <button type="submit">Toggle Speaker</button>
    </form>
    <h2>Status</h2>
    <pre>{{ status }}</pre>
</body>
</html>
```

---

## Submission Guidelines

* Submit your full codebase:

  * Refactored controller class
  * Decorator and Facade patterns
  * FastAPI-based web interface
* Include a short `README.md`:

  * How to run your FastAPI app
  * Explain which files contain your design patterns
* Ensure code is clean, organized, and documented

---

## Bonus (Optional)

* Add user authentication (basic session login)
* Register/unregister multiple devices from UI
* Use a database to store device states
* Add support for additional device types (e.g., Curtains, Hue Light)


