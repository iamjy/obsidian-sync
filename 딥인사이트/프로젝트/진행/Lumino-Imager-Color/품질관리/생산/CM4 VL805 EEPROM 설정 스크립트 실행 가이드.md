---
title: CM4 VL805 EEPROM 설정 스크립트 실행 가이드
source:
author:
  - "[[@_louis]]"
published:
created: 2025-09-23
description: VL805 USB 3.0 컨트롤러를 활성화하기 위한 EEPROM 설정 절차입니다. 이 과정을 통해 USB 3.0 포트의 고속 데이터 전송 기능을 사용할 수 있습니다.
tags:
  - cm4-usb-vl805
---
# CM4 VL805 EEPROM 설정 스크립트 실행 가이드

## 📋 개요
이 문서는 Compute Module 4 (CM4)에서 VL805 USB 컨트롤러의 EEPROM 설정을 위한 스크립트 실행 절차를 설명합니다.

## ⚠️ 주의사항
- **반드시 순서대로 실행**해야 합니다
- 각 단계 후 **재부팅이 필수**입니다
- 스크립트 실행 전 시스템 백업을 권장합니다

## 🔧 사전 준비
1. `cm4_vl805_manual.sh` 스크립트가 작업 디렉토리에 있는지 확인
2. 스크립트에 실행 권한 부여

```bash
chmod +x cm4_vl805_manual.sh
```

## 📝 실행 절차

### 1단계: EEPROM 설정 활성화
```bash
./cm4_vl805_manual.sh step1
```
- 기존 [cm4] 섹션의 EEPROM 설정을 활성화합니다
- **실행 완료 후 반드시 재부팅**

```bash
sudo reboot
```

### 2단계: VL805 EEPROM 설정 적용
```bash
./cm4_vl805_manual.sh step2
```
- VL805=1 EEPROM 설정을 적용합니다
- **실행 완료 후 반드시 재부팅**

```bash
sudo reboot
```

### 3단계: 설정 원복
```bash
./cm4_vl805_manual.sh step3
```
- EEPROM 설정을 다시 주석처리로 원복합니다
- **실행 완료 후 반드시 재부팅**

```bash
sudo reboot
```

## ✅ 전체 실행 순서 요약
```bash
# 스크립트 실행 권한 부여
chmod +x cm4_vl805_manual.sh

# 1단계: 기존 EEPROM 설정 활성화
./cm4_vl805_manual.sh step1
sudo reboot

# 2단계: VL805 EEPROM 설정 적용  
./cm4_vl805_manual.sh step2
sudo reboot

# 3단계: 설정 원복 (EEPROM 설정 다시 주석처리)
./cm4_vl805_manual.sh step3
sudo reboot
```

## 🚨 문제해결
- 스크립트 실행 중 오류 발생 시 시스템을 재부팅하고 해당 단계부터 다시 시작
- 각 단계가 성공적으로 완료되었는지 확인 후 다음 단계 진행
- 문제 발생 시 기술팀에 연락

---
**배포대상**: 생산팀

Lumino-Imager-Color 제품의 USB 포트에서 고속 데이터 전송(USB 3.0)을 사용하기 위해 필요한 설정 작업입니다. 

### 🔍 작업 내용 
	- CM4 모듈의 VL805 USB 컨트롤러 펌웨어 설정 
	- USB 3.0 고속 전송 기능 활성화
	
### ✨ 작업 완료 후 효과 
	- USB 포트 데이터 전송 속도 향상 (USB 2.0 → USB 3.0)
	- 고객의 대용량 파일 처리 성능 개선
