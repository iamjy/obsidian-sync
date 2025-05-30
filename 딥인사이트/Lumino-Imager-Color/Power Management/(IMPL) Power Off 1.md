---
title: 
source: 
author: 
published: 
created: 
description: 
tags:
---
---

# Request
PIN_POWER_BUTTON에 edge 가 detected 되는 즉시 후처리가 이루어지도록 수정해줘.

# Solution
파워 버튼(PIN_POWER_BUTTON)이 눌려질 때 즉시 처리되도록 수정하겠습니다:

```python:lumino-medison/ICG-Color/apps/apps-camera/core/gpio_handler.py
    def _get_pin_config(self) -> Dict[int, Dict[str, Any]]:
        """
        핀 설정 정보 가져오기
        
        Returns:
            Dict: 핀 번호를 키로, 설정 정보를 값으로 하는 딕셔너리
        """
        return {
            # 입력 핀 (버튼)
            self.PIN_RECORD_BUTTON: {
                'direction': GPIO.IN,
                'pull_up_down': GPIO.PUD_UP,  # 풀업 저항 사용 (버튼 누를 때 LOW)
                'edge': GPIO.BOTH,            # 양쪽 엣지 모두 감지
                'callback': self._on_record_button_event,
                'name': 'RECORD_BUTTON',
                'long_press_time': 3.0        # 3초 이상 누르면 녹화 시작
            },
            self.PIN_POWER_BUTTON: {
                'direction': GPIO.IN,
                'pull_up_down': GPIO.PUD_UP,
                'edge': GPIO.FALLING,
                'callback': self._on_power_button_pressed,
                'name': 'POWER_BUTTON',
                'long_press_time': 0.0  # 즉시 처리 (롱 프레스 시간 0초)
            },
            
            # 출력 핀 (LED)
            self.PIN_STATUS_LED: {
                'direction': GPIO.OUT,
                'initial': GPIO.LOW,
                'name': 'STATUS_LED'
            },
            self.PIN_ERROR_LED: {
                'direction': GPIO.OUT,
                'initial': GPIO.LOW,
                'name': 'ERROR_LED'
            },
            self.PIN_RECORDING_LED: {
                'direction': GPIO.OUT,
                'initial': GPIO.LOW,
                'name': 'RECORDING_LED'
            }
        }
    
    def _on_power_button_pressed(self, pin: int) -> None:
        """전원 버튼 누름 처리 - 즉시 종료 프로세스 시작"""
        try:
            pin_config = self.pin_config.get(pin, {})
            self.logger.warning(f"{pin_config.get('name', f'Pin {pin}')} 감지됨 - 즉시 종료 프로세스 시작")
            
            # 모든 LED를 깜빡여 사용자에게 종료 알림
            for led_pin in [self.PIN_STATUS_LED, self.PIN_ERROR_LED, self.PIN_RECORDING_LED]:
                self.blink_led(led_pin, 0.1, 3.0)
            
            # 녹화 중인 경우 녹화 중지
            if self.is_recording:
                self.logger.info("종료 전 녹화 중지 중...")
                self._stop_recording()
                # 녹화 파일이 안전하게 저장되도록 잠시 대기
                time.sleep(1.5)
            
            # 시스템 종료 이벤트 발행
            self.event_bus.publish(Event(
                EventType.SHUTDOWN_REQUESTED,
                "GPIOHandler",
                {
                    "reason": "power_button_press",
                    "delay_seconds": 3
                }
            ))
            
        except Exception as e:
            self.logger.error(f"Error handling power button press on pin {pin}: {e}")
```
파워 버튼(PIN_POWER_BUTTON)이 눌려지는 즉시 처리되도록 성공적으로 변경했습니다. 이제 파워 버튼을 누르면 다음과 같이 동작합니다:

### 변경된 기능:

1. **즉시 종료 프로세스 시작**:
   - 파워 버튼이 눌리는 즉시(edge 감지) 종료 프로세스가 시작됩니다
   - 이전처럼 5초간 길게 누를 필요가 없어졌습니다

2. **시각적 피드백 제공**:
   - 모든 LED(상태, 오류, 녹화)가 동시에 깜빡여 사용자에게 종료 중임을 알립니다
   - LED 깜빡임은 3초간 지속됩니다 (0.1초 간격으로 빠르게 깜빡임)

3. **안전한 종료 프로세스**:
   - 녹화 중인 경우 자동으로 녹화를 중지합니다
   - 녹화 파일이 안전하게 저장될 수 있도록 1.5초간 대기합니다
   - `SHUTDOWN_REQUESTED` 이벤트를 발행하여 시스템 종료를 요청합니다
   - 3초의 지연 시간을 두어 모든 리소스가 안전하게 정리될 수 있도록 합니다

이 변경으로 파워 버튼을 누르는 즉시 시스템 종료 프로세스가 시작되어 사용자 경험이 개선되고, 긴급 상황에서 빠르게 시스템을 종료할 수 있게 되었습니다. 또한 녹화 중인 파일을 안전하게 저장하는 기능도 유지됩니다.
