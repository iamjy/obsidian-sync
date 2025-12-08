# OMS (Occupant Monitoring System) Architecture Document

  

**Version:** 1.0.0

**Date:** 2025-12-09

**Project Type:** Embedded ML Inference System

**Language:** C++17

  

---

  

## 1. System Overview

  

OMS (Occupant Monitoring System)는 차량 내 탑승자 모니터링을 위한 실시간 ML 추론 오케스트레이션 시스템입니다. TensorRT 기반 GPU 가속과 우선순위 기반 작업 스케줄링을 통해 Face/Head/Hand 감지 및 탑승자 상태 분석을 수행합니다.

  

### 1.1 핵심 기능

  

| 기능 | 설명 |

|------|------|

| **FHH Detection** | YOLOv5 기반 Face/Head/Hand 객체 감지 (320x320) |

| **Occupant Tracking** | IoU/거리 기반 탑승자 추적 |

| **OOP Detection** | Out-of-Position (위험 자세) 감지 |

| **Seatbelt Status** | 안전벨트 착용 상태 분류 (OFF/ON/MISUSED) |

| **CRS Detection** | 유아용 카시트 감지 |

| **Body Size Classification** | 체형 분류 (AF05/AM50/AM95) |

  

### 1.2 시스템 특성

  

- **Real-Time Processing**: 50ms 타이머 주기, 500ms Work 주기

- **Priority Scheduling**: 3단계 우선순위 (LOWEST/MIDDLE/HIGHEST)

- **Hybrid Execution**: CPU ThreadPool + GPU TensorRT

- **Multi-Camera Support**: 최대 3개 카메라 (MAIN/LEFT/RIGHT)

  

---

  

## 2. High-Level Architecture

  

```

┌─────────────────────────────────────────────────────────────────────────┐

│ Application Layer │

│ ┌─────────────────────────────────────────────────────────────────┐ │

│ │ ICMU State Machine │ │

│ │ INIT → RUN → STOP → DEINIT → EXIT │ │

│ └─────────────────────────────────────────────────────────────────┘ │

└─────────────────────────────────────────────────────────────────────────┘

│

▼

┌─────────────────────────────────────────────────────────────────────────┐

│ Scheduling Layer │

│ ┌────────────────┐ ┌────────────────┐ ┌────────────────────┐ │

│ │ Timer │───▶│ WorkScheduler │───▶│ Priority Queue │ │

│ │ (POSIX RT Sig) │ │ (3 Levels) │ │ HIGH > MID > LOW │ │

│ └────────────────┘ └────────────────┘ └────────────────────┘ │

└─────────────────────────────────────────────────────────────────────────┘

│

▼

┌─────────────────────────────────────────────────────────────────────────┐

│ Work Layer │

│ ┌──────────────────────────────────────────────────────────────────┐ │

│ │ Work Container │ │

│ │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │ │

│ │ │FunctionInst 1│─▶│FunctionInst 2│─▶│FunctionInst N│ │ │

│ │ │(PreProcess) │ │(Inference) │ │(PostProcess) │ │ │

│ │ └──────────────┘ └──────────────┘ └──────────────┘ │ │

│ └──────────────────────────────────────────────────────────────────┘ │

└─────────────────────────────────────────────────────────────────────────┘

│

┌───────────────┼───────────────┐

▼ ▼

┌──────────────────────────────┐ ┌──────────────────────────────┐

│ CPU Execution │ │ GPU Execution │

│ ┌────────────────────────┐ │ │ ┌────────────────────────┐ │

│ │ ThreadPool │ │ │ │ TensorRT Engine │ │

│ │ (4 Workers) │ │ │ │ + CUDA Callbacks │ │

│ └────────────────────────┘ │ │ └────────────────────────┘ │

└──────────────────────────────┘ └──────────────────────────────┘

│ │

└───────────────┬───────────────┘

▼

┌─────────────────────────────────────────────────────────────────────────┐

│ I/O Layer │

│ ┌────────────────────────┐ ┌────────────────────────────┐ │

│ │ InputManager │ │ OutputManager │ │

│ │ • File/Camera/Network │ │ • File (JSON) │ │

│ │ • Ring Buffer (10) │ │ • Network (TCP) │ │

│ │ • RW Lock │ │ • Callback │ │

│ └────────────────────────┘ └────────────────────────────┘ │

└─────────────────────────────────────────────────────────────────────────┘

```

  

---

  

## 3. Component Details

  

### 3.1 Application Controller - ICMU

  

**파일**: `include/application_controller/dis_icmu.hpp`

  

ICMU는 애플리케이션 생명주기를 관리하는 상태 머신입니다.

  

```cpp

typedef enum {

ICMU_NONE = 0,

ICMU_INIT, // 초기화 단계

ICMU_STOP, // 정지 상태

ICMU_RUN, // 실행 상태

ICMU_DEINIT, // 종료 준비

OTHER_MODE,

ICMU_EXIT // 완전 종료

} e_IcmuState;

```

  

**상태 전이**:

```

INIT ──(성공)──▶ RUN ──(SIGINT/SIGTERM)──▶ STOP ──▶ DEINIT ──▶ EXIT

│ │

└──(실패)──▶ DEINIT ◀───────────────────────┘

```

  

### 3.2 Work Scheduler

  

**파일**: `include/work_controller/dis_work_scheduler.hpp`

  

우선순위 기반 작업 스케줄러로 3개의 우선순위 큐를 관리합니다.

  

**구성 요소**:

  

| 컴포넌트 | 역할 |

|----------|------|

| `st_PriorityQueue[3]` | LOW/MID/HIGH 우선순위 큐 |

| `Timer` | POSIX 실시간 신호 기반 타이머 (50ms) |

| `ThreadPool` | CPU 작업 실행 (4 workers) |

| `scheduleList[]` | 등록된 Work 목록 |

  

**스케줄링 알고리즘**:

```cpp

// 높은 우선순위부터 순차 처리 (선점 없음)

for(int i = PRIORITY_COUNT-1; i >= 0; i--) {

while(!isEmptyPriorityQueue(priority[i])) {

Work* work = dequeueSchedule(priority[i]);

runWork(work); // CPU 또는 GPU 실행

}

}

```

  

### 3.3 Work / FunctionInstance / Function

  

**파일**: `include/work_controller/dis_work.hpp`

  

계층적 작업 구조:

  

```

Work (Period: 500ms, Priority: HIGH)

├── FunctionInstance[0]: PreProcess (CPU)

│ └── Function: { Initializer, Runner, Deinitializer }

├── FunctionInstance[1]: ModelProcess (GPU)

│ └── Function: { Initializer, Runner, Deinitializer }

└── FunctionInstance[2]: PostProcess (CPU)

└── Function: { Initializer, Runner, Deinitializer }

```

  

**Work 상태**:

```cpp

typedef enum {

STATE_NONE = 0,

STATE_STOP, // 대기 상태

STATE_RUN // 실행 중

} e_WorkState;

```

  

### 3.4 TensorRT Engine

  

**파일**: `include/excute_controller/tensorrt_engine.h`

  

GPU 추론을 위한 TensorRT 래퍼 클래스입니다.

  

**주요 메서드**:

  

| 메서드 | 설명 |

|--------|------|

| `loadEngine(path)` | .engine 파일 로드 및 역직렬화 |

| `infer(inputs, outputs)` | 동기 추론 |

| `inferAsync(inputs, outputs)` | 비동기 추론 |

| `inferAsyncWithCallback(param, callback)` | 콜백 기반 비동기 추론 |

  

**메모리 관리**:

```

┌─────────────────┐ ┌─────────────────┐

│ Host Memory │ │ Device Memory │

│ (Page-Locked) │◀─────▶│ (GPU) │

├─────────────────┤ ├─────────────────┤

│ host_input_buf │──H2D─▶│ bindings[] │

│ host_output_buf │◀─D2H──│ │

└─────────────────┘ └─────────────────┘

```

  

### 3.5 FHH Detector (Face/Head/Hand)

  

**파일**: `include/models/dis_model_fhh.hpp`

  

YOLOv5 기반 객체 감지 모델입니다.

  

**모델 사양**:

  

| 파라미터 | 값 |

|----------|-----|

| Input Size | 320 × 320 × 3 (RGB) |

| Confidence Threshold | 0.25 |

| IOU Threshold (NMS) | 0.45 |

| Classes | Face(1), Head(2), Hand(3) |

| Anchors | 9개 (3 스케일 × 3 앵커) |

  

**전처리 파이프라인**:

```

Input Image (BGR)

↓

Letterbox Resize (aspect ratio 유지, pad=114)

↓

BGR → RGB 변환

↓

Normalize (÷ 255)

↓

NCHW Layout 변환

↓

GPU Device Memory

```

  

**후처리 파이프라인**:

```

GPU Output (50,400 floats)

↓

Decode (3-level FPN: 40×40, 20×20, 10×10)

↓

Sigmoid Activation (tx, ty, tw, th, obj, cls)

↓

Anchor Box Scaling

↓

Coordinate Transform (Letterbox 역변환)

↓

Per-Class NMS

↓

Detection Results

```

  

### 3.6 Algorithm Module

  

**파일**: `include/algorithm/dis_algorithm.hpp`

  

탑승자 상태 분석을 위한 알고리즘 모음입니다.

  

**주요 함수**:

  

| 함수 | 입력 | 출력 | 설명 |

|------|------|------|------|

| `CheckOccupant` | Detections, Config | OccupantState | IoU/거리 기반 탑승자 매칭 |

| `CheckOop` | Pose3D, Config | oopStatus[3] | 위험 자세 감지 |

| `CheckBelt` | Classification[3] | seatbeltStatus | 안전벨트 상태 분류 |

| `CheckBaby` | CRS Detections | bool | 카시트 유무 |

| `CheckBodySize` | Pose3D | bodySize | 체형 분류 |

  

**OccupantState 구조체**:

```cpp

struct OccupantState {

bool isActive; // 추적 활성화

BoundingBox bbox; // 현재 바운딩 박스

Pose2D trackedPose2d; // 2D 포즈 (11 keypoints)

Pose3D trackedPose3d; // 3D 포즈 (10 keypoints)

int bodySize; // 0=AF05, 1=AM50, 2=AM95

float upperbodyHeight; // 상체 높이 (px)

float upperbodyRecline; // 기울기 각도 (°)

int seatbeltStatus; // 0=OFF, 1=ON, 2=MISUSED

bool oopStatus[3]; // [close, far, leg]

};

```

  

### 3.7 Input Manager

  

**파일**: `include/io_controller/dis_input_manager.hpp`

  

다중 카메라 입력 버퍼 관리자입니다.

  

**입력 소스**:

```cpp

typedef enum {

INPUT_FILE = 0, // 이미지 파일 시퀀스

INPUT_CAMERA, // V4L2 카메라

INPUT_NETWORK // TCP 소켓 (Port 5000)

} e_InputType;

```

  

**버퍼 구조**:

```

InputManager

├── st_ImageHandler[CAMERA_MAIN]

│ ├── imageBuffer[10] (Ring Buffer)

│ │ └── st_Image { rwlock, image*, timestamp }

│ └── BufferIndexLock (mutex)

├── st_ImageHandler[CAMERA_LEFT]

└── st_ImageHandler[CAMERA_RIGHT]

```

  

**동시성 제어**:

- `pthread_rwlock_t`: 다중 reader, 단일 writer

- `pthread_mutex_t`: 버퍼 인덱스 보호

  

### 3.8 Output Manager

  

**파일**: `include/io_controller/dis_output_manager.hpp`

  

결과 수집 및 출력 관리자입니다.

  

**출력 타입**:

```cpp

typedef enum {

OUTPUT_FILE = 1<<0, // JSON 파일

OUTPUT_NETWORK = 1<<1, // TCP 전송

OUTPUT_CALLBACK = 1<<2 // 함수 콜백

} e_OutputType;

```

  

**출력 주기**: 1000ms (설정 가능)

  

---

  

## 4. Data Flow

  

### 4.1 추론 파이프라인

  

```

1. Timer Signal (50ms)

↓

2. WorkScheduler: 만료된 Work 검사

↓

3. Work 상태 → STATE_RUN, 우선순위 큐에 추가

↓

4. Dequeue & Execute:

┌──────────────────────────────────────────────────────┐

│ FunctionInstance[0]: PreProcess (CPU) │

│ • InputManager.GetImage() → 이미지 획득 │

│ • Letterbox resize, normalize │

│ • Host buffer에 전처리 결과 저장 │

└──────────────────────────────────────────────────────┘

↓

┌──────────────────────────────────────────────────────┐

│ FunctionInstance[1]: ModelProcess (GPU) │

│ • H2D: cudaMemcpyAsync (Host → Device) │

│ • Inference: context->enqueueV2() │

│ • D2H: cudaMemcpyAsync (Device → Host) │

│ • cudaStreamAddCallback() 등록 │

└──────────────────────────────────────────────────────┘

↓

┌──────────────────────────────────────────────────────┐

│ CUDA Callback: WorkScheduler::cudaCallback() │

│ • 결과를 output buffer에 복사 │

│ • Work step 증가 │

│ • 다음 FunctionInstance 스케줄링 │

└──────────────────────────────────────────────────────┘

↓

┌──────────────────────────────────────────────────────┐

│ FunctionInstance[2]: PostProcess (CPU) │

│ • NMS 적용 │

│ • Detection decode │

│ • Algorithm::CheckOccupant() 등 호출 │

│ • OutputManager에 결과 전달 │

└──────────────────────────────────────────────────────┘

```

  

### 4.2 타이밍 다이어그램

  

```

Time(ms) 0 50 100 150 200 250 300 350 400 450 500 550

│ │ │ │ │ │ │ │ │ │ │ │

Timer ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤

│ │ │ │ │ │ │ │ │ │ │ │

Work A ├────────────────────────────────────────────────────────┤

(500ms) │◀──────────────── Period ──────────────────▶│◀──Next──

│ │

├─PreProc─┼──GPU Infer──┼─PostProc─┤ │

│ │ │ │ │

Output │ │ │ │ ├──Export──▶

(1000ms) │◀────────────────── Period ─────────────────▶│

```

  

---

  

## 5. Threading Model

  

### 5.1 스레드 구성

  

| 스레드 | 역할 | 동기화 |

|--------|------|--------|

| Main (ICMU) | 상태 머신 관리 | - |

| Timer | POSIX RT 신호 대기 | sigwaitinfo() |

| ThreadPool[0-3] | CPU 작업 실행 | mutex + cond |

| OutputManager | 결과 출력 처리 | mutex + cond |

| SocketDataGenerator | 네트워크 수신 (선택) | std::mutex |

  

### 5.2 동기화 프리미티브

  

```cpp

// WorkScheduler

pthread_mutex_t schedulerMutex;

pthread_cond_t schedulerCondition;

st_PriorityQueue::mutex (per queue)

  

// InputManager

pthread_mutex_t BufferIndexLock;

pthread_rwlock_t st_Image::lock;

  

// OutputManager

pthread_mutex_t wakeupMutex;

pthread_cond_t wakeupCondition;

pthread_mutex_t st_OutputBuffer::bufferMutex;

  

// ThreadPool

pthread_mutex_t queueMutex;

pthread_cond_t taskCondition;

```

  

### 5.3 알려진 동시성 이슈

  

| 이슈 | 위치 | 심각도 | 설명 |

|------|------|--------|------|

| Busy-wait Loop | InputManager::takeWriteLock() | HIGH | CPU 100% 가능 |

| Race Condition | WorkScheduler::isRunning | HIGH | TOCTOU 문제 |

| Mixed Threading | SocketDataGenerator | MEDIUM | pthread + std::thread 혼용 |

| Callback Context | CUDA Callback | MEDIUM | 비동기 상태 수정 |

  

---

  

## 6. Configuration

  

### 6.1 빌드 설정

  

```cmake

# CMakeLists.txt 주요 옵션

CMAKE_CXX_STANDARD=17

CMAKE_BUILD_TYPE=Release|Debug

  

# 선택적 GPU 지원

-DWITH_CUDA # CUDA 찾으면 자동 정의

-DWITH_TENSORRT # TensorRT 찾으면 자동 정의

-DTensorRT_ROOT=/usr/local/tensorrt

```

  

### 6.2 런타임 설정

  

INI 형식 설정 파일:

```ini

[paths]

models = /path/to/models

  

[models]

object_detector = yolov5s_fhh_320x320.engine

  

[algorithm]

face_tracking_iou_threshold = 0.3

face_retrack_timeout = 2.0

  

[output]

print_engine_info = true

```

  

### 6.3 알고리즘 파라미터

  

| 파라미터 | 기본값 | 설명 |

|----------|--------|------|

| bboxIouThreshold | 0.3 | 추적 IoU 임계값 |

| locationRadius | 300.0 px | 초기 매칭 반경 |

| smoothingFactorPose2d | 0.5 | 2D 포즈 스무딩 |

| am50Threshold | 42.0 px | AF05/AM50 경계 |

| am95Threshold | 50.0 px | AM50/AM95 경계 |

| oopCloseThreshold | 30.0° | Close OOP 임계값 |

| oopFarThreshold | 83.0° | Far OOP 임계값 |

  

---

  

## 7. Dependencies

  

### 7.1 필수 의존성

  

| 패키지 | 버전 | 용도 |

|--------|------|------|

| CMake | 3.16+ | 빌드 시스템 |

| GCC/Clang | C++17 지원 | 컴파일러 |

| OpenCV | 4.x | 이미지 처리 |

| pthreads | POSIX | 멀티스레딩 |

  

### 7.2 선택적 의존성

  

| 패키지 | 버전 | 용도 | Fallback |

|--------|------|------|----------|

| CUDA Toolkit | - | GPU 연산 | CPU 전용 빌드 |

| TensorRT | 8.x | ML 추론 | GPU 기능 비활성화 |

| cuBLAS | - | 선형대수 | - |

| cuRAND | - | 난수 생성 | - |

  

---

  

## 8. Build & Run

  

### 8.1 빌드 명령

  

```bash

# 기본 빌드

mkdir -p build && cd build

cmake ..

make -j4

  

# 디버그 빌드

cmake .. -DCMAKE_BUILD_TYPE=Debug

make -j4

  

# TensorRT 경로 지정

cmake .. -DTensorRT_ROOT=/opt/tensorrt

make -j4

```

  

### 8.2 실행

  

```bash

./oms-cpp

```

  

### 8.3 출력 결과

  

- **바이너리**: `build/oms-cpp`

- **JSON 출력**: `./json/YYYYMMDD_HHMMSS.json`

  

---

  

## 9. Extension Guide

  

### 9.1 새 ML 모델 추가

  

1. `ModelTemplate` 또는 `TensorRTEngine` 상속

2. 3단계 콜백 구현:

- `PreProcessInitializer/Runner/Deinitializer`

- `ModelProcessInitializer/Runner/Deinitializer`

- `PostProcessInitializer/Runner/Deinitializer`

3. 모델 상수 정의 (입력 크기, 임계값, 클래스 수)

4. WorkScheduler에 Work로 등록

  

### 9.2 새 알고리즘 추가

  

1. `dis_algorithm.hpp`에 정적 함수 선언

2. `dis_algorithm.cpp`에 구현

3. Occupant Work 파이프라인에 통합

  

### 9.3 새 입력 소스 추가

  

1. `DataGenerator` 기본 클래스 상속

2. `getFrame()` 메서드 구현

3. `InputManager`에 새 타입 등록

  

---

  

## 10. Known Limitations

  

| 제한사항 | 설명 | 권장 조치 |

|----------|------|-----------|

| x86_64 전용 | TensorRT 경로가 x86_64로 하드코딩 | CMake에서 아키텍처 감지 추가 |

| 데드라인 미감지 | 작업 오버런 감지 없음 | 타임아웃 메커니즘 추가 |

| 런타임 폴백 없음 | GPU 실패 시 종료 | CPU 폴백 경로 구현 |

| 하드코딩된 경로 | 설정 파일 경로 고정 | 환경 변수/CLI 지원 |

| 제한된 에러 처리 | 로깅 의존 | 명시적 에러 복구 추가 |

  

---

  

## Appendix A: File Structure

  

```

camosys-oms-cpp/

├── include/ (20 headers)

│ ├── application_controller/ - ICMU 상태 머신

│ ├── work_controller/ - 스케줄링, 타이머

│ ├── excute_controller/ - CPU/GPU 실행

│ ├── io_controller/ - 입출력 관리

│ ├── models/ - ML 모델

│ └── algorithm/ - 알고리즘

├── src/ (20 sources)

│ └── (include와 동일 구조)

├── docs/ - 문서

└── CMakeLists.txt - 빌드 설정

```

  

---

  

## Appendix B: Glossary

  

| 용어 | 설명 |

|------|------|

| ICMU | Integrated Control Management Unit - 애플리케이션 제어 |

| FHH | Face/Head/Hand - 감지 대상 객체 |

| OOP | Out-of-Position - 위험 자세 |

| CRS | Child Restraint System - 유아용 카시트 |

| NMS | Non-Maximum Suppression - 중복 감지 제거 |

| H2D | Host to Device - CPU→GPU 메모리 전송 |

| D2H | Device to Host - GPU→CPU 메모리 전송 |