- [사용자 입력 내용을 간단하게 정리]  
    라즈베리파이 **CM5**와 **RK3588 기반 SoM**의 **단가(대략)**와 **주요 사양**을 표로 간단 비교해 달라는 요청입니다.
    
- GPT의 응답 요약 : CM5는 공식 “from $45” + 옵션별 소매가가 명확하고, RK3588 SoM은 **제조사/옵션별 편차가 커서 대표 제품(예: Firefly Core-3588 계열) 기준으로 범위**를 잡아 비교합니다.
    
- 응답 :
    

|항목|Raspberry Pi **Compute Module 5 (CM5)**|**RK3588 SoM** (대표 예: Firefly/Geniatech 등)|
|---|---|---|
|CPU|Quad **Cortex-A76** (Raspberry Pi 5급, 최대 2.4GHz 표기) ([Raspberry Pi](https://www.raspberrypi.com/products/compute-module-5/?utm_source=chatgpt.com "Buy a Raspberry Pi Compute Module 5"))|Octa **4×A76 + 4×A55** (A76 최대 2.4GHz 표기) ([rock-chips.com](https://www.rock-chips.com/uploads/pdf/2022.8.26/192/RK3588%20Brief%20Datasheet.pdf?utm_source=chatgpt.com "RK3588 Brief Datasheet.pdf"))|
|GPU|(CM5는 Pi 5 계열 GPU/비디오 인터페이스 제공: 상세 GPU명은 문서/버전별 확인 필요) ([Raspberry Pi](https://pip.raspberrypi.com/documents/RP-008181-DS-compute-module-5-product-brief.pdf?utm_source=chatgpt.com "Raspberry Pi Compute Module 5"))|**Mali-G610 MP4** (스펙 페이지에 450 GFLOPS 등 표기) ([en.t-firefly.com](https://en.t-firefly.com/product/industry/rocrk3588pc?utm_source=chatgpt.com "ROC-RK3588-PC"))|
|NPU|없음/미탑재(일반적으로 Pi 계열은 별도 NPU 모듈 사용) _(확실하지 않음: 특정 서드파티 CM5 변형 제외)_|**최대 6 TOPS NPU** ([en.t-firefly.com](https://en.t-firefly.com/product/industry/rocrk3588pc?utm_source=chatgpt.com "ROC-RK3588-PC"))|
|RAM 옵션|**2/4/8/16GB LPDDR4x** ([Raspberry Pi](https://www.raspberrypi.com/products/compute-module-5/?utm_source=chatgpt.com "Buy a Raspberry Pi Compute Module 5"))|**4~32GB** 옵션(벤더별 상이) ([geniatech.com](https://www.geniatech.com/product/som3588/?utm_source=chatgpt.com "RK3588 System-on-module \| Octa-core AI SoM"))|
|eMMC 옵션|**Lite(0GB), 16/32/64GB** ([Raspberry Pi](https://pip.raspberrypi.com/documents/RP-008180-DS-cm5-datasheet.pdf?utm_source=chatgpt.com "Raspberry Pi Compute Module 5"))|**8~128GB** 등 옵션(벤더별 상이) ([geniatech.com](https://www.geniatech.com/product/som3588/?utm_source=chatgpt.com "RK3588 System-on-module \| Octa-core AI SoM"))|
|주요 I/O 성격|**PCIe 인터페이스 제공**, 임베디드용 모듈 폼팩터 ([Raspberry Pi](https://www.raspberrypi.com/products/compute-module-5/?utm_source=chatgpt.com "Buy a Raspberry Pi Compute Module 5"))|**PCIe 3.0**, (레퍼런스 스펙에) HDMI/eDP, MIPI CSI/DSI, SATA 등 폭넓게 표기 ([rock-chips.com](https://www.rock-chips.com/uploads/pdf/2022.8.26/192/RK3588%20Brief%20Datasheet.pdf?utm_source=chatgpt.com "RK3588 Brief Datasheet.pdf"))|
|영상/디스플레이|(Product brief에) **듀얼 4Kp60 HDMI 출력** 등 표기 ([Raspberry Pi](https://pip.raspberrypi.com/documents/RP-008181-DS-compute-module-5-product-brief.pdf?utm_source=chatgpt.com "Raspberry Pi Compute Module 5"))|**8K 디코드/인코드** 등(벤더/보드 구성 따라 차이) ([geniatech.com](https://www.geniatech.com/product/som3588/?utm_source=chatgpt.com "RK3588 System-on-module \| Octa-core AI SoM"))|
|단가(대략, USD)|**공식 “from $45”** ([Raspberry Pi](https://pip.raspberrypi.com/documents/RP-008181-DS-compute-module-5-product-brief.pdf?utm_source=chatgpt.com "Raspberry Pi Compute Module 5")) \n예시 소매가: **$50(2GB/16GB) ~ $95(8GB/32GB)** 등 ([pishop.us](https://www.pishop.us/product-category/raspberry-pi/raspberry-pi-boards/compute-modules/compute-module-5/?srsltid=AfmBOopHj2IndQsSAt8D6Vf4cOYty0xotHhQTq-mYGD4r3GqdsRWUFsE&utm_source=chatgpt.com "Raspberry Pi Compute Module 5 Variants")) \n예시: **$155(16GB/64GB)** ([Seeed Studio](https://www.seeedstudio.com/Raspberry-Pi-Compute-Module-CM5116064-p-6299.html?srsltid=AfmBOopU8rNlrsP_SnI6zA1MN3o7Vibm8Lze9x48zI2vlup1MD22U0jh&utm_source=chatgpt.com "Raspberry Pi Compute Module 5 - 16GB RAM, 64GB ..."))|(대표 소매 예) Firefly RK3588 SoM: **$169~$199** 범위 표기 ([Firefly Store](https://www.firefly.store/collections/commercial-modules?srsltid=AfmBOoqYxKICDXcs2Ab1De2Dywvxu7yg8h2XJF9Rdzl1iK-qgruL3JWq&utm_source=chatgpt.com "Commercial Modules \| Firefly SoM")) \n대량/산업용/온도등급/램·eMMC 옵션에 따라 편차 큼 _(확실하지 않음)_|

**주의(단가 비교에서 중요한 점)**

- **CM5는 공식 가격 하한($45)과 옵션별 SKU 소매가가 비교적 명확**합니다. ([Raspberry Pi](https://pip.raspberrypi.com/documents/RP-008181-DS-compute-module-5-product-brief.pdf?utm_source=chatgpt.com "Raspberry Pi Compute Module 5"))
    
- **“RK3588 SoM”은 표준 제품이 1개가 아니라 제조사(예: Firefly, Geniatech, Forlinx 등)·보드 폼팩터·RAM/eMMC·산업용 온도(-40~85℃)·인증/기술지원 포함 여부**에 따라 가격이 크게 흔들립니다. ([geniatech.com](https://www.geniatech.com/product/som3588/?utm_source=chatgpt.com "RK3588 System-on-module | Octa-core AI SoM"))
    

원하시면, 비교 기준을 **(1) 목표 RAM/eMMC (예: 8GB/32GB)**로 맞춘 뒤, **(2) ‘양산가(100/1k/10k)’ 기준**으로 재정리해서 “견적 요청용 질문 리스트”까지 같이 뽑아드릴게요.