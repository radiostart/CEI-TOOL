# BakeTrack Hardware (CEI-TOOL) — KiCad 회로 설계

## 프로젝트 개요

BakeTrack IoT 환경 모니터링 기기의 하드웨어 회로 설계 프로젝트.
제과제빵 환경의 온도/습도를 측정하여 e-Paper 디스플레이에 표시하고 BLE로 모바일 앱과 통신하는 배터리 구동 IoT 센서 기기.

- **도구**: KiCad 9.0
- **PCB 사이즈**: 71.51mm x 33mm (2-layer, 1.6mm, FR4)
- **조립**: 단면 SMD (Top layer only)
- **제작**: JLCPCB (Gerber + BOM + Pick&Place 출력 완료)

---

## 핵심 IC 및 주요 부품

| Ref | Part Number | 제조사 | 역할 |
|-----|-------------|--------|------|
| U1 | ESP32-C3-MINI-1 | Espressif | MCU/SoC (Wi-Fi + BLE, RISC-V) |
| U2 | XC6220B331PR-G | Torex | 3.3V LDO 레귤레이터 (1A, SOT-89-5) |
| U3 | BQ24075RGTT | Texas Instruments | 1-cell Li-ion 충전 IC (Power Path, VQFN-16) |
| U4 | USBLC6-2SC6 | STMicroelectronics | USB ESD 보호 (SOT-23-6) |
| D2 | PESD5V0S2BT | NXP | I2C ESD 보호 (SOT-23) |
| D1 | SMBJ5.0A | — | VBUS TVS 다이오드 (DO-214AA) |
| D4 | SMF5.0A | — | 전원 TVS 다이오드 (SOD-123FL) |
| F1 | 0603L100SLYR | Littelfuse | PTC 퓨즈 (6V, 1A hold, 0603) |
| D3 | LTST-S270KRKT | Lite-On | 충전 상태 LED (Red 631nm, 0603) |

---

## GPIO 핀 맵 (ESP32-C3-MINI-1, U1)

| GPIO | KiCad Pin | Net Label | 기능 | 회로 상세 |
|------|-----------|-----------|------|-----------|
| GPIO0 | 12 | I2C_SDA | SHT4x 센서 데이터 | 2.2k pull-up (R10), JST SH 4핀 (J4) |
| GPIO1 | 13 | I2C_SCL | SHT4x 센서 클럭 | 2.2k pull-up (R8), JST SH 4핀 (J4) |
| GPIO2 | 5 | MCU-(GPIO2) | 배터리 전압 ADC (ADC1_CH2) | 분압: R2(47k)+R1(47k), R15(1k) 직렬, C11(0.47uF) 필터 |
| GPIO3 | 6 | MAIN_BTN | 메인 버튼 (Active LOW) | RTC GPIO, 딥슬립 웨이크업 지원 |
| GPIO4 | 18 | SPI_RST | E-Paper 리셋 | 풀업 (R17, 47k → 3.3V) |
| GPIO5 | 19 | USB_PGOOD | BQ24075 전원 양호 신호 | Open-drain, LOW=USB 연결 |
| GPIO6 | 20 | SPI_CLK | E-Paper SPI 클럭 | |
| GPIO7 | 21 | SPI_MOSI(DIN) | E-Paper SPI 데이터 | |
| GPIO8 | 22 | (no label) | Boot strap | 10k pull-up (R22) → SPI 부트 |
| GPIO9 | 23 | BOOT_BTN | 다운로드 모드 버튼 (S3) | Active LOW |
| GPIO10 | 16 | SPI_CS | E-Paper 칩 셀렉트 | 풀업 (R16, 47k) |
| GPIO18 | 26 | USB_D- | USB Type-C 데이터 | 22ohm 직렬 (R12,R14), ESD 보호 (U4, D2) |
| GPIO19 | 27 | USB_D+ | USB Type-C 데이터 | 22ohm 직렬 (R18,R19), ESD 보호 (U4, D2) |
| GPIO20 | 30 | SPI_BUSY | E-Paper BUSY 신호 | HIGH = Busy |
| GPIO21 | 31 | SPI_DC | E-Paper 데이터/명령 선택 | |

> GPIO11~17은 모듈 내부 SPI 플래시 전용 (외부 사용 불가)

---

## 커넥터

| Ref | Part Number | 제조사 | 타입 | 핀 수 | 용도 |
|-----|-------------|--------|------|-------|------|
| J1 | TYPE-C-31-M-12 | HRO | USB Type-C Receptacle | 16 | USB 전원 + 데이터 |
| J2 | 532610871 | Molex PicoBlade | SMD RA Header | 8 | E-Paper 디스플레이 (1.25mm pitch) |
| J3 | S2B-PH-SM4-TB | JST PH | SMD Header | 2 | 배터리 (2.0mm pitch) |
| J4 | SM04B-SRSS-TB | JST SH | SMD Header | 4 | I2C 센서 SHT-45 (1.0mm pitch) |
| J5 | 532610271 | Molex PicoBlade | SMD RA Header | 2 | 버튼 (1.25mm pitch) |
| J6 | SM02B-SRSS-TB | JST SH | SMD Header | 2 | 보조 커넥터 (1.0mm pitch) |

### J4 센서 커넥터 핀아웃 (SM04B-SRSS-TB)
1. +3.3V (C13 1uF 바이패스)
2. GND
3. I2C_SDA (GPIO0)
4. I2C_SCL (GPIO1)

---

## 전원 아키텍처

```
USB Type-C (VBUS 5V)
    │
    ├── F1 (PTC 1A) → D1 (TVS SMBJ5.0A) → VBUS rail
    │                                         │
    │                                    U3 (BQ24075)
    │                                    ├── USB_PGOOD → GPIO5
    │                                    ├── CHG_LED → D3 (Red) + R21 (2.2k)
    │                                    └── BAT ←→ J3 (Li-ion 3.7V)
    │                                         │
    └─────────────────────────────────────────┤
                                              │
                                         U2 (XC6220B)
                                              │
                                         +3.3V rail
                                              │
                                    ESP32-C3, SHT-45, E-Paper
```

### 충전 설정 (BQ24075)
- 충전 전류: R5 (1.8k) → ~500mA
- Power Path: USB와 배터리 간 동적 전환
- PGOOD: Open-drain, USB 연결 시 LOW

### 배터리 전압 측정 (GPIO2)
- 분압비: VBAT × 47k / (47k + 47k) = VBAT / 2
- R15 (1k): ADC 입력 보호 직렬 저항
- C11 (0.47uF): RC 로우패스 노이즈 필터
- 최대 배터리 4.2V → ADC 핀 2.1V (11dB 감쇠 범위 내)

### USB 보호
- PTC 퓨즈 F1 (1A hold)
- TVS D1 (SMBJ5.0A, VBUS)
- TVS D4 (SMF5.0A, 추가 보호)
- ESD U4 (USBLC6-2SC6, 데이터)
- ESD D2 (PESD5V0S2BT, 데이터)
- CC 저항 R4, R6 (5.1k): USB UFP/Sink 식별

---

## 스위치

| Ref | Part Number | 기능 |
|-----|-------------|------|
| S1 | KMR211NGLFS | 리셋 버튼 (EN/CHIP_PU → LOW) |
| S3 | KMR211NGLFS | 부트 모드 버튼 (GPIO9 → LOW) |

- 메인 버튼(MAIN_BTN)은 J5 커넥터를 통해 외부 연결 (GPIO3)
- 부트 모드 진입: S3 누른 채 USB 연결 (또는 S1으로 리셋)

---

## PCB 사양

| 항목 | 값 |
|------|-----|
| 보드 크기 | 71.51 x 33mm |
| 레이어 | 2-layer (F.Cu + B.Cu) |
| 두께 | 1.6mm (FR4, Er=4.5) |
| 동박 | 1oz (0.035mm) 양면 |
| 마운팅 홀 | 4개 (H1-H4) |
| 최소 트랙 폭 | 0.15mm (Default), 0.4mm (3V3), 0.8mm (VBUS) |
| 최소 비아 | 0.4mm dia / 0.2mm drill |
| USB 차동 페어 | 0.2mm width / 0.25mm gap |

### Net Class

| Class | 트랙 폭 | 비아 (dia/drill) | 클리어런스 | 용도 |
|-------|---------|-----------------|-----------|------|
| Default | 0.15mm | 0.4/0.2mm | 0.15mm | 신호선 |
| 3V3 | 0.4mm | 0.6/0.3mm | 0.2mm | 3.3V 전원 |
| VBUS | 0.8mm | 0.7/0.35mm | 0.25mm | USB 5V 전원 |
| USB_HS | 0.2mm | 0.45/0.2mm | 0.15mm | USB 차동 페어 |

---

## 파일 구조

```
CEI-TOOL/
├── CEI-TOOL.kicad_sch          # 회로도 (메인)
├── CEI-TOOL.kicad_pcb          # PCB 레이아웃
├── CEI-TOOL.kicad_pro          # 프로젝트 설정
├── CEI-TOOL.kicad_prl          # 로컬 환경 설정
├── CEI-TOOL.net                # 넷리스트
├── CEI-TOOL.csv                # BOM (회로도)
├── CEI-TOOL.step               # 3D 모델
├── Library.kicad_sym           # 로컬 심볼 라이브러리
├── sym-lib-table               # 심볼 라이브러리 경로 (26개)
├── fp-lib-table                # 풋프린트 라이브러리 경로 (27개)
├── jlcpcb_rules.json           # JLCPCB DRC 규칙
├── Gerber/                     # 거버 파일 (제작용)
├── production/                 # JLCPCB 제작 파일 (BOM, positions)
│   ├── bom.csv                 # BOM (29 line items)
│   ├── positions.csv           # Pick & Place (50 placements)
│   └── CEI-TOOL.zip            # 제작 패키지
├── lib/                        # 커스텀 부품 라이브러리 (21개)
└── CEI-TOOL-backups/           # 자동 백업 (25개)
```

---

## 연관 프로젝트

### BakeTrack 펌웨어 (CEI-SENSOR)
- **경로**: `../CEI-SENSOR`
- **프레임워크**: ESP-IDF v5.4.1 (PlatformIO)
- **역할**: 이 PCB 위에서 동작하는 펌웨어. GPIO 핀 맵, 전원 관리, 센서/디스플레이 드라이버 구현
- **claude.md**: `../CEI-SENSOR/claude.md`

### BakeTrack 모바일 앱 (CEI-APP)
- **경로**: `../CEI-APP`
- **프레임워크**: Flutter (iOS + Android)
- **역할**: BLE GATT Client. 이 기기와 BLE 통신하여 실시간 모니터링, 데이터 동기화, 공정 설정
- **claude.md**: `../CEI-APP/claude.md`

---

## 설계 참고사항

- ESP32-C3 RTC GPIO: GPIO0~5만 해당. 딥슬립 웨이크업은 GPIO3(MAIN_BTN)만 가능. GPIO5(USB_PGOOD)는 RTC GPIO이나 웨이크업 미사용
- 사용 가능한 GPIO를 전부 사용 중 — 여유 핀 없음 (GPIO0~10, 18~21)
- I2C 풀업 2.2k: 외부 프로브 케이블 최대 1.5m 대응 (50kHz 클럭), PESD5V0S2BT ESD 보호
- SPI_RST(GPIO4)는 R17(47k)로 풀업, SPI_CS(GPIO10)는 R16(47k)로 풀업
- USB_PGOOD는 GPIO5로 변경 (기존 GPIO10에서 스왑)
- USB Type-C는 CC1/CC2에 5.1k 풀다운으로 Sink/UFP 모드 고정
- 충전 전류 ~500mA는 R5(1.8k)로 설정 (BQ24075 ISET 핀)
- 보드 4개 마운팅 홀로 케이스 고정
- 배터리: BMS 내장 배터리 팩 사용 (외부 보호 회로 별도 미설치)
- LDO U2(XC6220B): 최대 1A 출력으로 ESP32-C3 WiFi TX 피크(~350mA)에 충분한 여유. VIN 디커플링 C4(10uF) + C14(22uF) 병렬
- E-Paper 디스플레이: 2.13인치 BW 250×122px (SSD1680 컨트롤러), J2 Molex 8핀 연결
