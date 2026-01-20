# 4G LTE IP Camera Streaming Project

A complete implementation of a 4G LTE network using srsRAN and USRP B210 SDRs, demonstrating real-time video streaming from an IP camera over a wireless cellular link.

## 🎯 Project Overview

This project successfully implements:
- Complete 4G LTE network (EPC + eNB + UE)
- Wireless video streaming from IP camera
- Real-time bidirectional communication
- Integration with USRP B210 Software Defined Radios

## 📋 Table of Contents

- [System Architecture](#system-architecture)
- [Hardware Requirements](#hardware-requirements)
- [Software Requirements](#software-requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Network Parameters](#network-parameters)
- [Streaming Commands](#streaming-commands)
- [Troubleshooting](#troubleshooting)
- [Performance Metrics](#performance-metrics)
- [Screenshots](#screenshots)
- [Contributing](#contributing)
- [License](#license)

## 🏗️ System Architecture
```
┌─────────────────┐
│   IP Camera     │
│  192.168.1.100  │
│  (Axis Camera)  │
└────────┬────────┘
         │
    [Switch]──────┬────────────────┬─────────────
         │        │                │
    ┌────┴────┐   │                │
    │  PC-3   │   │                │
    └─────────┘   │                │
         │  ┌─────┴──────────┐     │
         │  │     PC-1       │     │
         │  │ 192.168.1.11   │     │
         │  │                │     │
         │  │  EPC + eNB     │     │
         │  │  USRP B210 ────┼─────┤ ~~~~ LTE Air Interface ~~~~
         │  │  (TX/RX)       │     │
         │  └────────────────┘     │
         │                         │
         │                    ┌────┴──────────┐
         │                    │     PC-2      │
         │                    │ LTE IP:       │
         │                    │  172.16.0.2   │
         │                    │               │
         │                    │  UE + USRP    │
         └────────────────────│  B210 (TX/RX) │
                              │               │
                              │ Video Display │
                              └───────────────┘
```

## 🔧 Hardware Requirements

| Component | Specification | Quantity |
|-----------|--------------|----------|
| **PCs** | Ubuntu 22.04 LTS, Intel i5+ | 3 |
| **USRP B210** | Software Defined Radio | 2 |
| **Antennas** | 2.6 GHz, SMA connector | 2-4 |
| **Network Switch** | Gigabit Ethernet | 1 |
| **IP Camera** | RTSP capable (Axis) | 1 |
| **USB 3.0 Cables** | For USRP connection | 2 |
| **Ethernet Cables** | Cat 5e or better | 4 |

## 💻 Software Requirements

- **Operating System**: Ubuntu 22.04 LTS
- **srsRAN**: Version 4G (commit 5275f3336)
- **UHD**: 4.8.0 or later
- **FFmpeg**: 4.4.2 or later
- **Compiler**: GCC 11+ or Clang 12+
- **CMake**: 3.10+

## 📦 Installation

### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake libfftw3-dev libmbedtls-dev \
    libboost-program-options-dev libconfig++-dev libsctp-dev \
    libuhd-dev uhd-host git
```

### 2. Install srsRAN 4G
```bash
cd ~
git clone https://github.com/srsran/srsRAN_4G.git
cd srsRAN_4G
mkdir build
cd build
cmake ../
make -j$(nproc)
sudo make install
sudo ldconfig
```

### 3. Install FFmpeg (for video streaming)
```bash
sudo apt-get install -y ffmpeg vlc
```

### 4. Setup USRP
```bash
# Download USRP firmware
sudo uhd_images_downloader

# Test USRP connection
uhd_find_devices
uhd_usrp_probe
```

## ⚙️ Configuration

### PC-1 Configuration (EPC + eNB)

1. **Copy EPC configuration:**
```bash
mkdir -p ~/.config/srsran
cp configs/pc1-epc/epc.conf ~/.config/srsran/
cp configs/pc1-epc/user_db.csv ~/.config/srsran/
```

2. **Copy eNB configuration:**
```bash
cp configs/pc1-enb/enb.conf ~/.config/srsran/
sudo cp configs/pc1-enb/*.conf /etc/srsran/
```

3. **Configure network:**
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Add NAT rules
sudo iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o enp3s0 -j MASQUERADE
sudo iptables -P FORWARD ACCEPT
```

### PC-2 Configuration (UE)

1. **Copy UE configuration:**
```bash
mkdir -p ~/.config/srsran
cp configs/pc2-ue/ue.conf ~/.config/srsran/
```

## 🚀 Usage

### Starting the Network

#### On PC-1:

**Terminal 1 - Start EPC:**
```bash
cd ~/srsRAN_4G/build
sudo ./srsepc/src/srsepc ~/.config/srsran/epc.conf
```

Wait for: `SP-GW Initialized.`

**Terminal 2 - Start eNB (after EPC):**
```bash
cd ~/srsRAN_4G/build
sudo ./srsenb/src/srsenb ~/.config/srsran/enb.conf
```

Wait for: `==== eNodeB started ===`

#### On PC-2:

**Terminal 1 - Start UE:**
```bash
cd ~/srsRAN_4G/build
sudo ./srsue/src/srsue ~/.config/srsran/ue.conf
```

Expected: `Network attach successful. IP: 172.16.0.2`

## 📡 Network Parameters

| Parameter | Value |
|-----------|-------|
| **MCC** | 001 |
| **MNC** | 01 |
| **PLMN** | 00101 |
| **TAC** | 7 |
| **DL Frequency** | 2630 MHz |
| **UL Frequency** | 2510 MHz |
| **EARFCN** | 2850 |
| **Bandwidth** | 10 MHz (50 PRB) |
| **UE IP Pool** | 172.16.0.0/16 |
| **IMSI** | 001010123456789 |

## 📹 Streaming Commands

### Downlink (Camera → PC-1 → LTE → PC-2)

**On PC-1:**
```bash
ffmpeg -rtsp_transport tcp -i rtsp://Admin:Admin123@192.168.1.100:554/axis-media/media.amp \
  -c:v libx264 -preset veryfast -tune zerolatency \
  -b:v 400k -maxrate 400k -bufsize 800k \
  -vf "scale=640:480" \
  -c:a aac -b:a 32k -ar 22050 \
  -f mpegts "udp://172.16.0.2:5000?pkt_size=1316"
```

**On PC-2:**
```bash
ffplay -fflags nobuffer -flags low_delay -framedrop udp://0.0.0.0:5000
```

### Uplink (PC-2 → LTE → PC-1)

**On PC-2:**
```bash
ffmpeg -f lavfi -i testsrc=size=640x480:rate=15 \
  -f lavfi -i sine=frequency=1000:sample_rate=22050 \
  -c:v libx264 -preset veryfast -b:v 300k \
  -c:a aac -b:a 32k -t 60 \
  -f mpegts "udp://172.16.0.1:6000?pkt_size=1316"
```

**On PC-1:**
```bash
ffplay -fflags nobuffer udp://0.0.0.0:6000
```

## 🔧 Troubleshooting

### UE Not Connecting

1. **Check frequency match:**
```bash
# On eNB, verify output shows EARFCN 2850
grep dl_earfcn ~/.config/srsran/enb.conf
```

2. **Verify USRP connection:**
```bash
uhd_find_devices
```

3. **Check antenna placement:**
   - Ensure antennas on TX/RX ports (not RX2)
   - Distance: 1-2 meters apart
   - Both antennas vertical

### Port Binding Errors
```bash
# Kill existing processes
sudo killall srsepc srsenb srsue

# Check ports
sudo netstat -tulpn | grep -E "2152|36412"
```

### Low Throughput
```bash
# Reduce video bitrate
# Change -b:v 400k to -b:v 200k

# Adjust gains
tx_gain = 60  # in config files
rx_gain = 40
```

## 📊 Performance Metrics

| Metric | Value |
|--------|-------|
| **Latency** | 30-50 ms |
| **Downlink Throughput** | ~0.5 Mbps |
| **Uplink Throughput** | ~0.3 Mbps |
| **Video Bitrate** | 400 kbps |
| **Video Resolution** | 640x480 |
| **Packet Loss** | <5% |

## 📸 Screenshots

Place screenshots in `screenshots/` directory:
- `epc_running.png` - EPC terminal showing successful initialization
- `enb_started.png` - eNB started message
- `ue_connected.png` - UE showing IP assignment
- `video_streaming.png` - Video playback on PC-2
- `network_diagram.png` - Complete setup photo

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **srsRAN** - Open-source LTE implementation
- **Ettus Research** - USRP hardware
- **Open5GS** - 5G core network (for future work)

## 📧 Contact

For questions or issues, please open an issue on GitHub.

## 🔗 References

- [srsRAN Documentation](https://docs.srsran.com/)
- [USRP B210 Manual](https://www.ettus.com/all-products/ub210-kit/)
- [LTE Architecture Overview](https://www.3gpp.org/)

---

**Project Status:** ✅ Complete and Functional

**Last Updated:** January 2026
