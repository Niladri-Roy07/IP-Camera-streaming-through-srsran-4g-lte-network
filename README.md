# IP-Camera-streaming-through-srsran-4g-lte-network
Complete LTE (4G) network setup using srsRAN with UE attach, EPC integration, and IP data flow verification.
# Configuration Files

This directory contains all configuration files used in the srsRAN 4G LTE network implementation.

## Directory Structure
```
configs/
├── epc/
│   ├── epc.conf          # EPC core network configuration
│   ├── user_db.csv       # Subscriber database
│   └── README.md
├── enb/
│   ├── enb.conf          # eNodeB main configuration
│   ├── rr.conf           # Radio Resource configuration
│   ├── sib.conf          # System Information Block configuration
│   ├── rb.conf           # Radio Bearer configuration
│   └── README.md
└── ue/
    ├── ue.conf           # UE configuration
    └── README.md
```

## Configuration Overview

### EPC Configuration

**File**: `epc/epc.conf`

Key parameters:
- **MCC/MNC**: 001/01 (Mobile Country/Network Code)
- **TAC**: 7 (Tracking Area Code)
- **MME Address**: 127.0.1.100
- **APN**: srsapn
- **UE IP Pool**: 172.16.0.0/24

### eNodeB Configuration

**Files**: 
- `enb/enb.conf` - Main eNodeB configuration
- `enb/rr.conf` - Radio Resource configuration
- `enb/sib.conf` - System Information Blocks
- `enb/rb.conf` - Radio Bearer configuration

Key parameters:
- **Cell ID**: 0x01
- **PCI**: 1 (Physical Cell ID)
- **EARFCN**: 3350 (Band 7, 2680 MHz)
- **Bandwidth**: 10 MHz (50 PRBs)
- **TX Gain**: 80 dB
- **RX Gain**: 40 dB

### UE Configuration

**File**: `ue/ue.conf`

Key parameters:
- **IMSI**: 001010123456789
- **K**: 00112233445566778899aabbccddeeff
- **OPc**: 63BFA50EE6523365FF14C1F45F88737D
- **EARFCN**: 3350
- **TX/RX Gain**: 80/40 dB

## Usage

1. **Copy configurations to system directory**:
```bash
sudo cp -r configs/epc/* /etc/srsran/
sudo cp -r configs/enb/* /etc/srsran/
sudo cp -r configs/ue/* /etc/srsran/
```

2. **Or specify config file when running**:
```bash
sudo srsepc epc/epc.conf
sudo srsenb enb/enb.conf
sudo srsue ue/ue.conf
```

## Customization

### Adding Subscribers

Edit `epc/user_db.csv`:
```csv
name,mil,IMSI,K,opc,OPc,AMF,SQN,QCI,IP
ue4,mil,001010123456792,<key>,opc,<opc>,9001,000000000003,9,dynamic
```

### Changing RF Parameters

Edit `enb/enb.conf` and `
