# 🎹 7-Note Analog Optical Theremin (Synthesizer)

> **순수 아날로그 소자(Op-Amp, Comparator, BJT) 기반의 7음계 광학 테레민 시스템**  
> *Pure Analog 7-Note Optical Synthesizer Utilizing Wien-Bridge Oscillator & Class-B Push-Pull Amplifier*

[![Circuit](https://img.shields.io/badge/Circuit-Pure_Analog-E0234E.svg)]()
[![Simulation](https://img.shields.io/badge/Simulation-OrCAD_PSpice-007ACC.svg)]()
[![OpAmp](https://img.shields.io/badge/Op--Amp-TL084_JFET-orange.svg)]()
[![Comparator](https://img.shields.io/badge/Comparator-LM339-green.svg)]()
[![PowerAmp](https://img.shields.io/badge/Power_Amp-TIP41C_%2F_TIP42C-blue.svg)]()

---

## 1. 📖 Project Overview (개요)

디지털 마이크로컨트롤러(MCU)나 음원 합성 IC 없이, **순수 아날로그 전자회로 소자(Op-Amp, 비교기, BJT, 수동소자)만으로 7음계(C4~B4)를 생성하고 빛으로 연주하는 광학 테레민(Optical Theremin)** 프로젝트입니다.  
12V 단일 전원 환경에서 6V 가상 접지(Virtual Ground)를 구축하여 교류 신호의 스윙 범위를 확보하고, 빈 브리지 발진기(Wien-Bridge Oscillator), 듀얼 BJT 아날로그 오디오 스위치, 그리고 교차 왜곡(Crossover Distortion)을 보상한 B급 푸시풀 전력 증폭기를 유기적으로 통합하여 8Ω 스피커를 직접 구동합니다.

* **개발 기간:** 2026.03 ~ 2026.06 (2인 팀 프로젝트 / 전자회로실험 텀프로젝트)
* **담당 역할:** 
  * **스위칭·발진부 회로 설계 및 OrCAD PSpice 시뮬레이션 검증**
  * **듀얼 2N3904 BJT 기반 아날로그 오디오 게이트(Mute 스위치) 설계 및 바이어스 해석**
  * **TL084 기반 7채널 빈 브리지 발진기 주파수 수식 도출 및 파형 클리핑/감쇠 보정**
  * **단위 이득 버퍼(Unity-Gain Buffer)를 활용한 독립 가상 접지(Virtual Ground) 안정화**

---

## 2. 🏗️ System Architecture (시스템 구조)

```plaintext
┌─────────────────────────────────────────────────────────────┐
│             [ 1. 광학 감지 및 제어부 (Logic Stage) ]           │
│  - 7채널 CdS 조도 센서 (손 움직임에 따른 명암/저항 변화 감지)    │
│  - LM339 전압 비교기 (기준 전압 Vref 비교 후 12V High/Low 출력) │
└──────────────────────────────┬──────────────────────────────┘
                               │ 제어 신호 (Base Bias, IB = 0.83mA)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│        [ 2. 정현파 발진 및 스위칭부 (Oscillator & Gate) ]      │
│  - TL084 빈 브리지 발진기 (RC 피드백을 통한 C4~B4 순음 생성)    │
│  - Dual 2N3904 BJT 스위치 (Emitter Follower 통과 및 0V Mute) │
│  - AC 커플링 커패시터 & 믹싱 저항 (다중 채널 간섭 및 DC 충돌 차단)│
└──────────────────────────────┬──────────────────────────────┘
                               │ 오디오 신호 통합 (Master Bus)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             [ 3. 전력 증폭 및 출력부 (Power Stage) ]          │
│  - Negative Feedback 루프 결합형 TIP41C/42C B급 푸시풀 증폭기  │
│  - 교차 왜곡(Crossover Distortion) 보상 및 8Ω 스피커 구동      │
└─────────────────────────────────────────────────────────────┘
```

<div align="center">
  <img width="90%" alt="전체 회로도 (1개 음계 기준)" src="여기에_7페이지_전체회로도_이미지_URL_입력" />
</div>

### 🛠 Tech Stack & Hardware Components

| 분류 | 소자 / 장비 | 규격 | 주요 역할 |
| :--- | :--- | :--- | :--- |
| **Op-Amp** | **TL084** | Quad JFET Op-Amp (3EA) | 7음계 빈 브리지 정현파 발진기, 가상 접지 버퍼, 마스터 증폭 |
| **Comparator** | **LM339** | Quad Comparator (2EA) | 7채널 CdS 광센서 전압 판별 및 풀업 스위칭 제어 |
| **Transistor** | **2N3904** | NPN BJT (14EA) | 채널별 듀얼 아날로그 오디오 게이트 (채널당 2개, Mute 제어) |
| **Power BJT** | **TIP41C / TIP42C** | NPN / PNP Power BJT (6EA) | 8Ω 스피커 구동용 B급 푸시풀 전력 증폭단 |
| **Sensor** | **CdS (LDR)** | GL5528 (7EA) | 도(C4)~시(B4) 7음계 독립 광학 건반 센서 |
| **Passives** | **R / C / VR** | 10kΩ 가변저항(14EA) 외 | 주파수 정밀 튜닝, 12V 풀업, AC 커플링, 패시브 믹싱 |
| **Instruments** | **Test & Sim** | PSpice, DSO, DMM, PSU | 파형 계측, 주파수 캘리브레이션, 시뮬레이션 검증 |

---

## 3. ⚙️ Key Features & Circuit Design (핵심 회로 설계)

### 🔹 1) 단일 전원 극복을 위한 가상 접지(Virtual Ground) 및 단위 이득 버퍼
* **전압 분배 기준점 확보:** 단일 전원($V_{CC}=12\text{V}$, 시뮬레이션 $9\text{V}$) 환경에서 교류 정현파가 위아래로 대칭 스윙할 수 있도록 저항 분배($R_1 = R_2 = 10\text{k}\Omega$)를 통해 중간 기준 전압($V_{ref}$)을 형성했습니다.
  $$V_{ref} = V_{CC} \times \frac{R_2}{R_1 + R_2} = 12\text{V} \times \frac{10\text{k}\Omega}{10\text{k}\Omega + 10\text{k}\Omega} = 6\text{V}$$
* **단위 이득 버퍼(Unity-Gain Buffer) 적용:** 다중 채널 동작 시 가상 접지 전압이 흔들리는 현상을 방지하기 위해, 여유분의 TL084 비반전 단자에 기준 전압을 인가하여 출력 임피던스가 매우 낮은 고정 가상 전원을 각 발진단에 독립 공급했습니다.

### 🔹 2) TL084 기반 7채널 빈 브리지 발진기 (Wien-Bridge Oscillator)
* **RC 정궤환 주파수 결정:** 정궤환(Positive Feedback) 루프의 직·병렬 RC 네트워크($C=100\text{nF}$ 고정)와 가변저항 튜닝을 통해 C4(도)~B4(시) 7음계 정현파를 독립 생성했습니다.
  $$f = \frac{1}{2\pi R C} \quad \text{e.g., C4(도): } f = \frac{1}{2\pi \times 6.2\text{k}\Omega \times 100\text{nF}} \approx 256.7\text{Hz}$$
* **발진 유지 조건($A_v \ge 3$) 충족:** 부궤환(Negative Feedback) 루프의 비반전 증폭기 이득을 $A_v = 1 + R_4/R_3 \ge 3$으로 설계하고, 실제 소자 공차로 인한 파형 감쇠 및 클리핑을 잡아내기 위해 반고정 가변저항으로 정밀 이득을 튜닝했습니다.

| 음계 | 도 (C4) | 레 (D4) | 미 (E4) | 파 (F4) | 솔 (G4) | 라 (A4) | 시 (B4) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **표준 주파수** | 261.6 Hz | 293.7 Hz | 329.6 Hz | 349.2 Hz | 392.0 Hz | 440.0 Hz | 493.9 Hz |
| **커패시턴스($C$)** | 100 nF | 100 nF | 100 nF | 100 nF | 100 nF | 100 nF | 100 nF |
| **이론 저항($R$)** | ~6.08 kΩ | ~5.42 kΩ | ~4.83 kΩ | ~4.56 kΩ | ~4.06 kΩ | ~3.62 kΩ | ~3.22 kΩ |

### 🔹 3) 듀얼 2N3904 BJT 아날로그 오디오 게이트 (Mute Switch)
* **활성 영역 에미터 팔로워 동작:** 연주자가 CdS를 가리면 LM339 비교기가 High($12\text{V}$ 풀업)를 출력하며, 2N3904 베이스에 $I_B \approx \frac{12\text{V}-0.7\text{V}}{10\text{k}\Omega} \approx 0.83\text{mA}$의 전류를 공급해 오디오 AC 신호를 에미터로 왜곡 없이 통과시킵니다.
* **2-BJT 연동 완전 무음(Mute) 구현:** BJT 1개만 사용할 때 발생하는 오프 상태 신호 누설을 해결하기 위해 채널당 2개의 2N3904를 연동하여, 비연주 시에는 신호를 $0\text{V}(\text{GND})$로 완전히 흘려보내는(Bypass) 스위치 구조를 완성했습니다.

### 🔹 4) 교차 왜곡(Crossover Distortion) 보상 B급 푸시풀 전력 증폭기
* **고출력 스피커 구동:** 오피앰프 단일 출력으로 구동이 불가능한 $8\Omega$ 저임피던스 스피커를 울리기 위해 TIP41C(NPN) / TIP42C(PNP) 상보 대칭 푸시풀 전력 증폭단을 설계했습니다.
* **Negative Feedback 루프 내 출력단 결합:** 바이어스 다이오드 없이 TL084 연산증폭기의 부궤환 루프 내부에 푸시풀 트랜지스터의 에미터 출력 노드를 직접 포함시켜, BJT 문턱전압($\pm 0.7\text{V}$) 데드존으로 인한 교차 왜곡을 원천 제거했습니다.

---

## 4. 🛠️ Troubleshooting & Optimization (트러블슈팅)

### 📌 1. 다중 음계 동시 연주 시 채널 간섭 및 로딩 효과(Loading Effect) 해결
* **문제 상황:**  
  단일 음계는 깔끔하게 출력되나, 2개 이상의 음계를 동시에 연주할 때 볼륨이 급격히 줄어들고 지지직거리는 노이즈가 섞이거나 개별 소리가 죽어버리는 현상 발생
* **원인 분석:**  
  7개 채널의 출력이 마스터 버스에 직접 결선되면서 채널 간 직류(DC) 전압 충돌이 발생하고, 공통 가상 접지 라인이 흔들리며 다른 채널의 발진 주파수까지 간섭하는 로딩 효과 확인
* **해결 방안:**  
  1. **패시브 믹싱 저항 및 AC 커플링 도입:** 각 채널이 마스터 버스로 합류하는 길목에 직류 차단용 커패시터(AC Coupling)와 믹싱 저항을 직렬 배치하여 채널 간 전기적 분리 확보
  2. **단위 이득 가상 접지 버퍼 추가:** 미사용 TL084 앰프를 전압 팔로워로 활용하여 흔들리지 않는 고정 가상 접지 제공
* **결과:**  
  화음(다중 음계) 연주 시 직류 충돌 및 볼륨 저하 문제를 해결하고 맑은 정현파 믹싱 달성

---

### 📌 2. 아날로그 다단계 비교 한계 극복 및 IC 미사용 핀 플로팅(Floating) 디버깅
* **문제 상황:**  
  1. 초기 설계 시 CdS 3개의 조도(명암 단계)를 3V/6V 다중 기준전압과 디지털 논리 게이트로 조합해 7음계를 구현하려 했으나, 주변 광노이즈 민감도 및 전원 전압 불일치(12V 아날로그 회로 vs 5V TTL 게이트)로 인해 판별 불가 현상 발생
  2. 브레드보드 실장 후 발진 회로가 간헐적으로 먹통이 되어 파형이 출력되지 않는 현상 발생
* **해결 방안:**  
  1. **7채널 독립 광센서 및 물리적 차광 칸막이 적용:** 다단계 조도 판별 방식을 과감히 폐기하고 CdS 센서를 7개로 확장하여 1:1 매칭 구조로 개편, 센서 간 빛 간섭을 막는 칸막이 하우징 설치
  2. **미사용 IC 핀 종단(Termination) 처리:** Quad 패키지 IC(TL084, LM339) 내부의 사용하지 않는 핀들을 허공에 둔(Floating) 것이 내부 발진 및 정전기 노이즈의 원인임을 규명하고 전원/접지 처리 수행
* **결과:**  
  외부 조명 환경 변화에도 오작동 없는 즉각적인 건반 반응성 및 100% 발진 신뢰성 확보

---

## 5. 🎬 Simulation & Hardware Results (실험 및 구현 결과)

### 🔹 PSpice 시뮬레이션 검증 (비연주 vs 연주 상태 파형 비교)

| 비연주 상태 (CdS 빛 감지 - Mute) | 연주 상태 (CdS 빛 차단 - Sine Wave 출력) |
| :---: | :---: |
| <img width="851" height="1191" alt="image" src="https://github.com/user-attachments/assets/350105c1-64d5-4f9a-9d46-a3a047f66752" />
 | <img width="841" height="1243" alt="image" src="https://github.com/user-attachments/assets/d5763f2f-9c06-4739-bf2b-f9aaeac9932e" /> |
| LM339 출력 0V 유지, 스피커 출력단 신호 차단 | 2N3904 게이트 개방, 스피커 단 정현파 정상 출력 확인 |

---

### 🔹 발진부 회로 개선에 따른 오실로스코프 파형 변화 (1차 ➡️ 3차)

| 1차 발진 회로 (발진 불량 / 클리핑) | 2차 발진 회로 (고조파 왜곡 / 기계음) | 3차 최종 회로 (순수 정현파 출력) |
| :---: | :---: | :---: |
| <img width="844" height="295" alt="image" src="https://github.com/user-attachments/assets/484f8dd5-913e-4092-843f-0dda580be3ba" />
 | <img width="850" height="306" alt="image" src="https://github.com/user-attachments/assets/289f5887-4631-4c71-a898-8aef8d64232c" />
 | <img width="862" height="377" alt="image" src="https://github.com/user-attachments/assets/f418570f-1f56-45c9-89b7-7ee41d24c844" /> |

---

### 🔹 하드웨어 브레드보드 프로토타입 및 시연 영상

<div align="center">
  <img width="756" height="765" alt="image" src="https://github.com/user-attachments/assets/e68cdfb5-bffb-43d8-a0a5-0c252bfbca93" />

</div>

<div align="center">

<!-- GitHub 편집창에 시연 영상 파일을 드래그 앤 드롭하여 생성된 링크를 아래에 붙여넣으세요 -->
https://github.com/user-attachments/assets/여기에_시연_영상_고유_링크_입력

</div>

---

## 6. 📂 Repository Structure (저장소 구조 제안)

```plaintext
Analog_Optical_Theremin/
├── simulation/
│   ├── oscillator_wien_bridge.opj   # TL084 빈 브리지 발진기 PSpice 프로젝트
│   └── full_theremin_channel.opj    # LM339 + 2N3904 + Push-Pull 통합 시뮬레이션
├── docs/
│   ├── schematic_single_channel.pdf # 1개 음계 기준 전체 회로도
│   ├── BOM_component_list.md        # 소요 부품 리스트 (BOM)
│   └── final_report.pdf             # 전자회로실험 텀프로젝트 최종 보고서
├── media/
│   └── oscilloscope_waveforms/      # 오실로스코프 파형 캡처 및 하드웨어 사진
└── README.md
```
````[cite: 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 15, 16, 17]

---

이미지와 영상 파일을 GitHub 편집창에 드래그 앤 드롭하여 생성된 `[https://github.com/user-attachments/assets/](https://github.com/user-attachments/assets/)...` 주소만 각 자리표시자에 넣어주시면 완성됩니다. 추가로 수정하거나 강조하고 싶은 부분이 있다면 편하게 말씀해 주세요!
