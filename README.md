# Assignment: Refactor & Extend SmartApp IoT System

## 📌 Objective

Refactor the existing SmartApp IoT project to follow sound software design principles. You'll improve maintainability, flexibility, and scalability by:

1. **Refactoring to a controller class** (separating logic from UI)
2. **Implementing design patterns**: Decorator and Facade
3. **Migrating from a Tkinter GUI to a web-based interface** using **FastAPI** and the **MVT (Model-View-Template)** pattern
4. **Extending device implementations** to support real attributes and respond to actions
5. **Refactoring** the SmartApp IoT project to implement a true **microservices** architecture where each device runs as an independent HTTP server with real network endpoints.

---

##  Part 1: SmartApp System
├── Main Web Application (FastAPI)
├── Device Microservices
│   ├── Smart Speaker (Port 8001)
│   ├── Smart Light (Port 8002) 
│   └── Smart Curtains (Port 8003)
└── Controller & Facade Layer

##   Project Structure

smartapp-iot/
├── main.py                 # FastAPI web application
├── controller/
│   ├── app_controller.py   # Main application controller
│   └── iot_facade.py       # Facade pattern implementation
├── devices/
│   ├── base_device.py      # Base device class & decorators
│   ├── smart_speaker.py    # Speaker device microservice
│   ├── smart_light.py      # Light device microservice
│   └── smart_curtains.py   # Curtains device microservice
├── web/
│   ├── templates/
│   │   └── index.html
│   └── static/
│       └── style.css
└── requirements.txt

## Part 2: Implementation Tasks

1.1 Base Device Interface
Create an abstract base class that all devices must implement.

'''python
class Device(ABC):
    @abstractmethod
    def __init__(self, device_id: str, host: str, port: int):
        self.device_id = device_id
        self.host = host
        self.port = port
        self.base_url = f"http://{host}:{port}"
    
    @abstractmethod
    def get_status(self) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def perform_action(self, action: str, **kwargs) -> bool:
        pass
'''
