![image](https://zenodo.org/records/17547596/files/SPARROW-banner.png)
 
  </a>
</p>

<div align="center"> 
<font size="6"> Accelerating Research and Conservation with Edge AI.</font>
<br>
<hr>
<!-- Badges Section -->
<p align="center">
  <!-- License -->  
    <img src="https://pypi-camo.freetls.fastly.net/cd0913ed24368b790668a699719b5562b033448d/68747470733a2f2f696d672e736869656c64732e696f2f707970692f6c2f5079746f72636857696c646c696665">
  <!-- Docker -->
    <img src="https://img.shields.io/badge/docker-ready-blue?logo=docker">
  <!-- Contributions Welcome -->
  <img src="https://img.shields.io/badge/contributions-welcome-brsvg">
<br><br>
</div>

# 👋 Welcome to SPARROW

**SPARROW**, developed by **Microsoft's AI for Good Lab**, is an **AI-powered edge computing solution** designed to monitor and protect wildlife in the most remote regions of the world.  
Solar-powered and equipped with advanced sensors, it collects biodiversity data—from camera traps, acoustic monitors, and other environmental detectors—that are processed using our most advanced PyTorch-based wildlife AI models on power efficient edge GPUs. The resulting critical information is then transmitted via low-Earth orbit satellites directly to the cloud or on-premise infrastructure, allowing researchers to access fresh, actionable insights in real time, no matter where they are.

# ✨ Key Features

1. **🔋 Autonomous operation**  
   Intelligent power management with solar charging, battery health monitoring, and dynamic component scheduling enables continuous off-grid operation.

2. **📷 Sensing**  
   Camera traps, acoustic monitoring, and environmental sensors - SPARROW integrates multi-modal sensing to capture images, sounds, and enviromental metrics for comprehensive biodiversity monitoring.

3. **🧠 On-device AI**  
   Runs optimized PyTorch Wildlife models on power efficient edge GPUs (e.g., Jetson Orin Nano) for real-time image and acoustic detection, species classification, and event recognition.

4. **🌐 Global connectivity**  
   Even in the most remote ecosystems, SPARROW maintains a link to the cloud or on-premise infrastructure through low-Earth-orbit satellites, ensuring that vital conservation data reaches researchers in near real time.

5. **🛡️ Resilience**  
   Designed for extreme field conditions - SPARROW safely records data when offline, automatically synchronizing once connectivity is restored to ensure no loss of information.

---

# This repository contains the **SPARROW client**:  
Data collection, on-device inference, power management, telemetry, and secure transmission.  

All services run in **Docker** and are orchestrated with **Docker Compose**. 🐳

---

# 🚀 Getting Started

## 🛠️ 1. Hardware Assembly


### 📋 Prerequisites

🛒 Before you begin, ensure you have all the necessary hardware listed in the Bill of Materials (BOM):  
[https://link-to-bom/](https://link-to-bom/)

🏗️ Full build instructions can be found here:  
[https://link-to-build/](https://link-to-build/)

---

## ⚡ 2. One-click Jetson Setup (Recommended)

The repo contains a Jetson configuration script `sparrow_setup.sh` that installs prerequisites, prepares folders, downloads default Triton models, seeds the DS3231 RTC, configures Wi-Fi hotspot, and launches the SPARROW services.

To send data to the SPARROW dashboard you will need to pair it with your account.  
To create an account and obtain an access key visit:  
[https://dashboard.sparrow-earth.com/](https://dashboard.sparrow-earth.com/)

**Script:** Download the SPARROW setup script from this repo once the hardware assembly and Jetson flash is complete (detailed instructions can be found in the build instructions).  
The setup script should be run from `~/Desktop`.

### ▶️ Usage

```bash
cd ~/Desktop
sudo chmod +x sparrow_setup.sh
sudo ./sparrow_setup.sh

```

## What the Script Does

### 1️⃣ Prereqs & Tooling
Installs:
```
docker, docker-compose, git, curl, wget, uuidgen, smbus2
```

### 2️⃣ Device Identity
Generates `/etc/unique_id` if missing (single-line UUID).

### 3️⃣ Folder Layout (Host)
Creates `~/Desktop/system` with:
```
/system
├── docker-compose.yml
├── sparrow_setup.sh
├── Models/
│   └── tritonserver/
│       └── model_repository/
│           ├── megadetectorv6/
│           │   ├── 1/
│           │   │   └── model.onnx
│           │   └── config.pbtxt
│           ├── AI4GAmazonClassification/
│           │   ├── 1/
│           │   │   └── model.onnx
│           │   └── config.pbtxt
│           └── megadetector_birds_v1/
│               ├── 1/
│               │   └── model.onnx
│               └── config.pbtxt
├── sparrow/
│   ├── Dockerfile
│   ├── config/
│   │   └── access_key.txt
│   ├── images/
│   ├── recordings/
│   ├── logs/
│   └── static/
│       ├── data/
│       └── gallery/
└── starlink/
    ├── Dockerfile.starlink
    ├── config/
    │   └── access_key.txt
    └── logs/

```

### 4️⃣ Models + Configs
Downloads three default ONNX models from Zenodo and writes minimal `config.pbtxt` for each Triton repo.

### 5️⃣ Access Key
Prompts for the server access key (obtained from the SPARROW dashboard) and writes it to:
```
sparrow/config/access_key.txt
starlink/config/access_key.txt
```

### 6️⃣ RTC Seeding (DS3231 over I2C bus 7)
Gets UTC from WorldClock API (fallback: NTP or system UTC) and writes it to the RTC.

### 7️⃣ Wi-Fi Hotspot
Configures a persistent hotspot via NetworkManager:  
**SSID:** `CameraTraps`  
**Password:** `User prompted`

### 8️⃣ Docker Build & Launch
Builds images with BuildKit (no cache), runs `docker-compose up -d`, and tails logs.

---

# 🧩 Software Dependencies

All Python dependencies are inside the containers (no host Python required):

1. `PyTorch`, `torchaudio`, `tritonclient`, `aiosmtpd`, `psutil`, `smbus2`, `pyserial`, etc.  
2. **NVIDIA Triton Inference Server** (explicit model control mode)  
3. **Hardware:** I2C, ALSA audio, and USB serial (compose is configured privileged)

---

# 🔗 API References

The client calls these endpoints on `SERVER_BASE_URL`:

| Endpoint | Description |
|-----------|--------------|
| `/uploads` | Image + detection metadata |
| `/audio_uploads` | WAV audio files |
| `/system_metrics` | System + sensor metrics JSON |
| `/get_schedule` | Starlink sleep window |
| `/get_scheduleaudio` | Audio capture settings |
| `/model_settings` | Classification model + labels |
| `/model_update` | Triton model manifest |

Each request includes `auth_key` and a `unique_id` derived from `/etc/unique_id`.

---

# 🤝 Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [Contributor License Agreements](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

---

# :fountain_pen: Cite us!
We have recently published a [technical paper on SPARROW](https://arxiv.org/abs/XXXXXXX). Please feel free to cite us!

```
@misc{lavista2025listening,
      title={Listening to the Earth in Real Time: SPARROW and the Future of Conservation Technology}, 
      author={Juan M. Lavista Ferres*, Carl Chalmers*, Bruno Demuro Segundo*, Zhongqi Miao*, Andres Hernandez Celis, Isai Daniel Chacon Silva, Allen Kim, Luana Marotti, Amy Michaels, Daniela Ruiz Lopez, Rahul Dodhia, Inbal Becker-Reshef, Pablo Andrés Arbelaez Escalante, Federico Alves Torres, Meygha Machado, Anthony Cintron Roman},
      year={2025},
      eprint={xxxxxxx},
      archivePrefix={arXiv},
}
```

# 🏷️ Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.

---
