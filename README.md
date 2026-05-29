# KrishiKausal

The **KrishiKausal** is a comprehensive software suite designed to monitor agricultural metrics and remotely control a 6-wheel or 4-wheel rover. The system consists of a web-based **Dashboard (Frontend)** and a **Raspberry Pi Onboard Agent (Backend)**.

It provides a dashboard for tracking agricultural metrics (soil moisture, temperature, pH, NPK, etc.), accompanied by a backend API that interfaces directly with hardware sensors and a physical rover vehicle's motor controller.

---

## 🎯 Architecture Overview

1. **Backend (Rover API)**: Runs on a Raspberry Pi mounted on the rover. It exposes a FastAPI server to receive control commands, run autonomous navigation procedures, read sensors, and drive the DC motors and servos via GPIO.
2. **Frontend (Dashboard)**: A React/Vite web application that provides real-time monitoring of soil metrics (moisture, temperature, pH, NPK), AI suggestions, and teleoperation capabilities (Rover Control).

---

## 🚀 1. Hardware & Wiring Guide

The rover uses a differential drive system (sides are driven as units) and optional steering servos. The pinout is defined in `backend/config.py`.

**Motor Controller Pins (BCM)**
- **Left Side Motors**:
  - `IN1`: BCM 5
  - `IN2`: BCM 6
  - `PWM`: BCM 12
- **Right Side Motors**:
  - `IN1`: BCM 13
  - `IN2`: BCM 19
  - `PWM`: BCM 18

**Servo Pins (BCM)** (For suspension/steering if applicable)
- Front-Left: BCM 23
- Front-Right: BCM 24
- Rear-Left: BCM 25
- Rear-Right: BCM 8

---

## 💻 2. Setup & Development Environment

### Frontend Setup (Local Laptop/PC)
The dashboard is built with React, Vite, and Tailwind CSS. You usually run this on your laptop or deploy it statically.

```bash
# 1. Navigate to the project directory
cd <project-directory>

# 2. Install Node.js dependencies
npm install

# 3. Start the Vite development server
npm run dev
```
The dashboard will be available at `http://localhost:5173`. When configuring the API endpoint, check your `.env` or network settings to point it to the Raspberry Pi's IP Address on the local network.

### Backend Setup (Raspberry Pi)
The FastAPI backend controls the hardware and reads real-time sensors. It should be run on the Raspberry Pi mounted to the rover.

```bash
# 1. SSH into your Raspberry Pi
ssh pi@<pi-ip-address>

# 2. Navigate to where you clone this repository
cd ~/<project-directory>

# 3. Create a Python virtual environment and activate it
python3 -m venv venv
source venv/bin/activate

# 4. Install dependencies
# Assuming you need common FastAPI packages and RPi.GPIO
pip install fastapi uvicorn pydantic pydantic-settings

# 5. Run the server natively
python backend/main.py

# OR run with uvicorn for auto-reloading during development
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload
```
The API is now running at `http://<pi-ip-address>:8000`.

---

## 🛠️ 3. Continuous Operation (Systemd Service)

To ensure the rover's backend starts automatically every time you power on the Raspberry Pi in the field, you can configure it as a Linux systemd service.

1. **Verify the Service File**:
   Open `tools/krishikausal.service`. Update the `User`, `Group`, `WorkingDirectory`, and `ExecStart` variables to match your deployment layout.
   *(E.g., mapping exactly to the python executable in your `venv`).*

2. **Copy and Enable the Service**:
   Run these commands on the Raspberry Pi:
   ```bash
   # Copy the service file to the systemd directory
   sudo cp tools/krishikausal.service /etc/systemd/system/

   # Reload the systemd daemon
   sudo systemctl daemon-reload

   # Enable the service to start automatically on boot
   sudo systemctl enable krishikausal.service

   # Start the service immediately
   sudo systemctl start krishikausal.service

   # Check the status of the service
   sudo systemctl status krishikausal.service
   ```

3. **Viewing System Logs**:
   To view live console logs (e.g., debug information, connection events, or Python errors) of the background service:
   ```bash
   journalctl -u krishikausal.service -f
   ```

---

## 🗂️ 4. Project Structure

```text
krishikausal/
├── backend/            # FastAPI python server running on the Pi
│   ├── api/            # API endpoints (`control`, `sensors`, etc.)
│   ├── config.py       # Hardware GPIO pin configuration
│   └── main.py         # Entry point for the API
├── src/                # React web application source code
│   ├── components/     # UI elements (SensorCard, RoverControl, etc.)
│   └── pages/          # The main Dashboard page (Index.tsx)
├── tools/              # Utility scripts and Linux service config (.service files)
├── pi_agent/           # Rover specific Python scripts and helpers
└── package.json        # Frontend config and dependencies
```

---

## 🚜 Field Workflow Example

1. Power on the Rover in the field (the built-in Raspberry Pi boots up and launches the backend `krishikausal.service` automatically).
2. Connect your laptop, tablet, or phone to the same Wi-Fi network as the Pi (or the Pi's own Hotspot).
3. Open the Frontend Dashboard URL (either hosted locally via `npm run dev` on your laptop, or statically deployed alongside the backend).
4. Go to the **Control** tab to manually drive the rover to the desired location in the field.
5. Go back to the **Soil Insights** tab to read the real-time sensor analytics from that location and see crop-specific AI Suggestions.
