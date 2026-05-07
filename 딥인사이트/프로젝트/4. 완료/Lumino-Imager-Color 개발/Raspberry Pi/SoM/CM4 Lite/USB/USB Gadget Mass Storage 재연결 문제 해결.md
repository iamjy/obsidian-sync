# USB Gadget Mass Storage 재연결 문제 해결

## 📝 문제 상황

### 초기 증상
- **환경**: Raspberry Pi + dwc2 USB 컨트롤러
- **backing file**: `/piusb.bin` (exFAT 포맷)
- **문제**: 부팅 후 USB gadget mass storage가 한번만 인식됨
- **핵심 이슈**: eject 후 USB 케이블 재연결 시 정상적으로 mount되지 않음

### 에러 로그
```
[  137.167913] dwc2 fe980000.usb: new device is high-speed
[  137.231907] dwc2 fe980000.usb: new device is high-speed
[  137.284654] dwc2 fe980000.usb: new address 7
```

## 🔍 문제 분석 과정

### 1. 시스템 상태 확인
- **USB gadget 설정 방식**: legacy `g_mass_storage` 모듈 사용
- **configfs 상태**: 마운트되어 있지만 실제 gadget 구성 없음
- **주요 경로들**:
  - UDC: `/sys/class/udc/fe980000.usb`
  - backing file: `/sys/devices/platform/soc/fe980000.usb/gadget.0/lun0/file`
  - suspend 상태: `/sys/class/udc/fe980000.usb/device/gadget.0/suspended`

### 2. 핵심 발견사항

#### eject 동작 분석
- **eject 전**: backing file에 `/piusb.bin` 경로 정상 표시
- **eject 후**: backing file이 **완전히 비워짐** (empty)
- 이는 eject의 직접적인 결과를 감지하는 가장 확실한 방법

#### suspend 상태 변화
- **0**: 장치 활성 상태 (호스트와 정상 통신 중)
- **1**: 장치 suspend 상태 (호스트가 suspend 시키거나 연결 해제됨)
- **unknown**: 연결 해제 상태

#### USB 연결 상태별 패턴
| 상황 | Suspend | Backing File | 의미 |
|------|---------|-------------|------|
| 정상 활성 | 0 | correct | 정상 사용 중 |
| 정상 대기 | 1 | correct | PC 절전 등 |
| **eject 감지** | 0/1 | **empty** | eject 실행됨 |
| 재연결 | unknown→0 | any | 케이블 재연결 |

## 🛠️ 해결 방안 개발 과정

### 시도 1: configfs 기반 접근
- 문제: `/sys/kernel/config/usb_gadget/g1` 경로 존재하지 않음
- 원인: legacy 모듈 사용 중이어서 configfs 구성 없음

### 시도 2: I/O 활동 모니터링
- 시도: `/proc/diskstats`에서 USB gadget I/O 감지
- 문제: USB gadget은 호스트 관점의 블록 디바이스가 아니어서 I/O 통계 없음

### 시도 3: UDC 상태 모니터링
- 시도: `/sys/class/udc/fe980000.usb/state` 확인
- 문제: eject 후에도 `configured` 상태 유지되어 물리적 연결 해제 감지 불가

### 시도 4: dmesg 로그 모니터링
- 시도: USB 관련 커널 메시지 실시간 모니터링
- 한계: 모든 상황을 실시간으로 캐치하기 어려움

## ✅ 최종 해결책: Suspend 상태 기반 모니터링

### 핵심 아이디어
- **suspend 상태 변화**만을 감지하여 재연결/절전 복귀 등에 대응
- **eject는 로깅만** 하고 액션하지 않음 (backing file empty 감지)
- **물리적 재연결**은 suspend 상태 변화로 감지

### 구현된 스크립트 구조

#### 상태 감지 로직
```bash
# suspend 상태 확인
get_suspend_state() {
    cat /sys/class/udc/fe980000.usb/device/gadget.0/suspended 2>/dev/null || echo "unknown"
}

# 상태 변화 분석
analyze_suspend_transition() {
    case "${prev_suspend}→${curr_suspend}" in
        "unknown→0") echo "reconnection_active" ;;      # 재연결 (활성)
        "unknown→1") echo "reconnection_suspended" ;;   # 재연결 (대기)
        "1→0") echo "became_active" ;;                   # 절전 복귀
        "0→1") echo "became_suspended" ;;                # 절전 진입
        *) echo "no_change" ;;
    esac
}
```

#### 액션 결정 로직
| Suspend 변화 | 의미 | 액션 |
|-------------|------|------|
| `unknown → 0` | **재연결 (활성)** | `full_reset` |
| `unknown → 1` | **재연결 (suspend)** | `gentle_reset` |
| `1 → 0` | suspend → 활성 | backing 상태에 따라 `quick_reset` 또는 `verify_only` |
| `0 → 1` | 활성 → suspend | backing 이상시만 `gentle_reset` |
| backing file empty | eject 감지 | **액션 없음 (로깅만)** |

#### 복구 방법들
```bash
# 빠른 재설정 (0.5초)
quick_reset() {
    echo "$BACKING_FILE" > "$BACKING_FILE_PATH"
    sleep 0.5
}

# 부드러운 재설정 (1초, suspend 상태 고려)
gentle_reset() {
    echo "$BACKING_FILE" > "$BACKING_FILE_PATH"
    sleep 1
}

# 전체 재설정 (2초, 재연결 시)
full_reset() {
    echo "" > "$BACKING_FILE_PATH"
    sleep 1
    echo "$BACKING_FILE" > "$BACKING_FILE_PATH"
    sleep 1
}
```

## 🚀 설치 및 사용법

### 서비스 설치
```bash
# 기존 서비스 정리
sudo systemctl stop usb-*-monitor*.service
sudo systemctl disable usb-*-monitor*.service

# 새 스크립트 설치
sudo tee /usr/local/bin/usb_suspend_only_monitor.sh > /dev/null <<'EOF'
[스크립트 내용]
EOF

sudo chmod +x /usr/local/bin/usb_suspend_only_monitor.sh

# systemd 서비스 생성
sudo tee /etc/systemd/system/usb-suspend-only-monitor.service > /dev/null <<EOF
[Unit]
Description=USB Mass Storage Suspend-Only Monitor
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/local/bin/usb_suspend_only_monitor.sh
Restart=always
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
EOF

# 서비스 시작
sudo systemctl daemon-reload
sudo systemctl enable usb-suspend-only-monitor.service
sudo systemctl start usb-suspend-only-monitor.service
```

### 모니터링
```bash
# 실시간 로그 확인
sudo tail -f /var/log/usb-gadget.log

# 서비스 상태 확인
sudo systemctl status usb-suspend-only-monitor.service

# 현재 상태 확인
echo "Suspend: $(cat /sys/class/udc/fe980000.usb/device/gadget.0/suspended 2>/dev/null)"
echo "Backing: $(cat /sys/devices/platform/soc/fe980000.usb/gadget.0/lun0/file 2>/dev/null || echo 'empty')"
```

## 📊 성능 및 결과

### 감지 및 복구 시간
- **체크 간격**: 1초
- **재연결 감지**: 최대 1초
- **복구 시간**: 0.5~2초 (상황에 따라)
- **총 재연결 시간**: **1.5~3초**

### 주요 장점
1. **🎯 정확한 감지**: suspend 상태 변화로 재연결/절전 복귀 정확히 감지
2. **⚡ 빠른 복구**: 상황별 최적화된 복구 방법 사용
3. **🧠 지능적 판단**: eject는 로깅만, suspend 변화에만 반응
4. **🔄 자동화**: 사용자 개입 없이 모든 재연결 상황 자동 처리

### 해결된 시나리오
- ✅ **eject 후 케이블 재연결**: 1.5~3초 이내 자동 마운트
- ✅ **PC 절전 모드 복귀**: 상태 확인 및 필요시 재설정
- ✅ **케이블 분리 후 재연결**: 즉시 감지 및 완전 재설정
- ✅ **장시간 사용 안정성**: 지속적인 모니터링으로 안정성 보장

## 🔧 트러블슈팅

### 일반적인 문제들
1. **suspend 파일 없음**: dwc2 드라이버 로드 확인
2. **backing file 없음**: g_mass_storage 모듈 상태 확인
3. **서비스 실행 실패**: 로그 파일 권한 및 경로 확인

### 디버깅 명령어
```bash
# 경로들 존재 확인
ls -la /sys/class/udc/fe980000.usb/device/gadget.0/suspended
ls -la /sys/devices/platform/soc/fe980000.usb/gadget.0/lun0/file

# 모듈 상태 확인
lsmod | grep g_mass_storage

# UDC 상태 확인
cat /sys/class/udc/fe980000.usb/state
```

## 📝 배운 점

1. **USB Gadget의 복잡성**: 호스트-디바이스 간 상태 동기화의 어려움
2. **상태 감지의 중요성**: 올바른 상태 파일을 찾는 것이 핵심
3. **계층적 접근**: 여러 상태를 조합해서 정확한 판단 필요
4. **성능 최적화**: 상황별 최적화된 복구 방법의 중요성

---

**최종 결과**: eject 후 USB 케이블 재연결 시 **1.5~3초 이내 자동 마운트** 달성! 🎉
