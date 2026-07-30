현재 소프트웨어 업데이트 OTA를 **기능 및 개념 설계 중심의 구성도** 수준에서 블록 다이어그램을 그릴려고 하는데, acs는 nvidia jetson 플랫폼이고 영상 송수신 및 제어 신호 요청에 대한 응답을 하는 기능을 하고 iPad의 app은 영상을 수신하고 제어 신호를 요청하는 역할이야. ota에 대 한 자세한 동작 시나리오를 발주사에서 주지 못하고 있어서 내가 직접 동작 시나리오를 제안해야 할 거 같아. 우리 장비에 들어가는 소프트웨어는 기본적으로 nvidia jetson이고 추가적으로 beam projection을 위한 dlpc 컨트롤러 spi flash 펌웨어 및 ap 단 드라이버 그리고 ite6810fn hdmi convertor 펌웨어 정도야. 크게 보면 이렇게 3가지야. drawio로 시각화 해줘.

Worked for 14m 20s

## 생성 파일

ACS OTA 기능·개념 설계 Draw.io 파일

![전체 기능·개념 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_000000001d9072099ceff060259ac148&fn=ACS_OTA_Main_Architecture_Preview.png&cd=attachment&ts=495532&p=fs&cid=1&sig=4512168d89a4435c5b7b91bffd8a3c4768f2e0d992f9b96b3001401e64c0e4f6&v=0)

![전체 기능·개념 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_000000001d9072099ceff060259ac148&fn=ACS_OTA_Main_Architecture_Preview.png&cd=attachment&ts=495532&p=fs&cid=1&sig=4512168d89a4435c5b7b91bffd8a3c4768f2e0d992f9b96b3001401e64c0e4f6&v=0)

![전체 기능·개념 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_000000001d9072099ceff060259ac148&fn=ACS_OTA_Main_Architecture_Preview.png&cd=attachment&ts=495532&p=fs&cid=1&sig=4512168d89a4435c5b7b91bffd8a3c4768f2e0d992f9b96b3001401e64c0e4f6&v=0)

Share

ACS_OTA_Main_Architecture_Preview.svg

Draw.io 파일에는 다음 **7개 페이지**가 포함되어 있습니다.

1. **설계 전제 및 범례**  
    iPad, ACS, 세 가지 업데이트 대상의 역할과 기본 설계 원칙
2. **전체 기능·개념 구성도**  
    iPad App, OTA 서버, Jetson OTA Agent, 영상·제어 서비스, DLPC·ITE 업데이트 어댑터 및 하드웨어 연결 구조
3. **OTA 패키지 및 신뢰체계**  
    Jetson Image, AP Driver, DLPC Firmware, ITE Firmware를 하나의 Bundle로 구성하고 Manifest 서명·Hash·호환성 검증을 수행하는 구조
4. **정상 업데이트 시나리오**  
    업데이트 조회 → 사용자 승인 → 다운로드 → 검증 → Maintenance Mode → Jetson 비활성 슬롯 기록 → Pending Boot → DLPC/ITE 업데이트 → 통합 점검 → Commit 흐름
5. **실패 및 복구 시나리오**  
    다운로드 실패, 서명 실패, Jetson 부팅 실패, DLPC/ITE 기록 실패, Health Check 실패, 전원 차단 및 iPad 연결 단절 대응
6. **컴포넌트별 적용 전략**  
    Jetson OS/BSP/App, AP Driver, DLPC SPI Flash Firmware, ITE6810FN Firmware 각각의 적용 방식·검증·복구 정책
7. **발주사 결정 및 협의 항목**  
    배포 토폴로지, 권한 정책, 전원 조건, A/B 파티션, DLPC·ITE ISP 방식, 성공 판정 기준 등 상세 설계 전에 확정해야 할 사항

## 반영한 핵심 제안

기본 토폴로지는 **ACS가 OTA 서버에서 패키지를 직접 다운로드**하고, iPad는 업데이트 조회·승인·진행 표시를 담당하도록 구성했습니다. 폐쇄망 환경을 고려해 iPad가 패키지를 ACS로 전달하는 **Offline Package Relay**도 선택 경로로 표시했습니다.

업데이트 시작 후에는 iPad 연결이 끊기더라도 ACS가 독립적으로 업데이트를 계속합니다. 진행 상태는 ACS의 Persistent Metadata에 저장하고, iPad가 다시 연결되면 `/ota/status` 또는 이벤트 인터페이스를 통해 복원하는 개념입니다.

Jetson 영역은 **A/B 슬롯과 Pending Boot 방식**을 권고안으로 적용했습니다. 신규 이미지는 비활성 슬롯에 먼저 기록하고, 신규 슬롯 부팅 및 전체 기능 점검이 성공한 뒤에만 Commit하도록 구성했습니다. NVIDIA Jetson Linux는 패키지 기반 OTA와 Image-based OTA 방식을 제공하며, Image-based OTA는 전체 이미지를 파티션 단위로 업데이트할 수 있습니다. 다만 실제 A/B 적용 가능 여부는 사용 중인 Jetson Linux/L4T 버전과 저장장치 파티션 구성에 따라 확인해야 합니다.

DLPC와 ITE 펌웨어는 Jetson보다 복구 제약이 클 수 있으므로 별도 위험 경로로 표현했습니다. DLPC 계열은 모델에 따라 SPI Flash에 펌웨어를 저장하며, Boot/Update Mode, Sector Erase, Program, Readback 또는 Checksum 같은 절차가 달라질 수 있습니다. 따라서 정확한 DLPC 모델과 Flash Map이 확정되기 전까지는 구체적인 명령 대신 `DLPC Update Adapter`로 추상화했습니다.

**ITE6810FN은 제공해주신 표기를 그대로 사용했습니다.** 공개된 공식 자료에서 정확한 Part Number와 ISP 절차를 확인하지 못했기 때문에, 도면에서는 `Vendor ISP / I²C / SPI — Interface TBD`로 남겨 두었습니다. 상세 설계 시에는 ITE 벤더 자료를 통해 내장·외장 Flash 여부, ISP 진입 조건, Dual-Bank 지원, Readback 및 복구 방식을 반드시 확정해야 합니다.

또한 AP-side Driver는 독립적으로 먼저 교체하기보다 **신규 Jetson Slot에 포함**시키도록 했습니다. 신규 Driver가 기존 Peripheral Firmware와 목표 Firmware를 모두 처리하거나, 별도의 Low-level Transition Updater가 있어야 안전하게 주변 펌웨어를 전환할 수 있다는 제약도 도면에 반영했습니다.

【DevOps】 서버 구성 ( GitOps, NoOps ).pdf

PDF

【DevOps】 파이프라인 ( 빌드 _ 배포 _ 코드 품질 _ 시험 _ 배포 ).pdf

PDF

침부 파일의 개발 인프라가 우리 회사 소프트웨어 self-hosted 개발 인프라인데. 이 인프라를 활용한 방법도 반영해줘.

Worked for 20m 23s

## Self-hosted 개발 인프라 연계 반영본

수정된 ACS OTA 기능·개념 설계 Draw.io

![Self-hosted DevOps 연계 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000adb072068e9276f5a30b9fa8&ts=495533&p=fs&cid=1&sig=6dfd658695c365e3d923566072d3ffef47f5d751964adc5a5d0ba52dfa09e148&v=0)

![Self-hosted DevOps 연계 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000adb072068e9276f5a30b9fa8&ts=495533&p=fs&cid=1&sig=6dfd658695c365e3d923566072d3ffef47f5d751964adc5a5d0ba52dfa09e148&v=0)

![Self-hosted DevOps 연계 구성도 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000adb072068e9276f5a30b9fa8&ts=495533&p=fs&cid=1&sig=6dfd658695c365e3d923566072d3ffef47f5d751964adc5a5d0ba52dfa09e148&v=0)

Share

  
ACS_OTA_SelfHosted_DevOps_Architecture_Preview.svg

![CI/CD → Signed OTA Release Pipeline 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000cc0472099ed748dddf753209&ts=495533&p=fs&cid=1&sig=fbb5c2e8308586b114a22999c01c5c2bca4185e3d4dc2eefd47859e4fe2326a0&v=0)

![CI/CD → Signed OTA Release Pipeline 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000cc0472099ed748dddf753209&ts=495533&p=fs&cid=1&sig=fbb5c2e8308586b114a22999c01c5c2bca4185e3d4dc2eefd47859e4fe2326a0&v=0)

![CI/CD → Signed OTA Release Pipeline 미리보기 PNG](https://chatgpt.com/backend-api/estuary/content?id=file_00000000cc0472099ed748dddf753209&ts=495533&p=fs&cid=1&sig=fbb5c2e8308586b114a22999c01c5c2bca4185e3d4dc2eefd47859e4fe2326a0&v=0)

Share

  
ACS_OTA_SelfHosted_CICD_Pipeline_Preview.svg

기존 7페이지를 **9페이지로 확장**하고, 기존 페이지 일부도 함께 수정했습니다.

## 반영된 인프라 활용 방식

첨부된 서버 구성도의 **NGINX Reverse Proxy, OpenLDAP, Redmine, Jenkins, SCM, SonarQube, Docker Registry, Git Repository, Artifact & Log DB, File Server/Cloud 백업, Cron/Rsync, Parity 및 Hot Spare 구조**를 OTA 개발·릴리스 백엔드 역할로 매핑했습니다.

첨부된 파이프라인의 **Issue/SCM Trigger → Coverage → Build Agent → 다중 Test Agent → Deploy → Self-hosted 또는 Official Docker Hub → Release** 흐름은 OTA에 필요한 서명·패키징·GitOps 승격 절차를 추가한 형태로 확장했습니다.

### 기존 인프라별 OTA 역할

|기존 인프라|OTA에서의 활용|
|---|---|
|Redmine|OTA 변경 요청, 결함, 위험, Release 승인 이력 및 현장 실패 Issue 연계|
|SCM / Git|Jetson SW, AP Driver, DLPC FW, ITE FW Metadata, OTA Manifest 관리|
|Jenkins|빌드, 시험, Composite Bundle 생성, 게시 작업 자동화|
|Build Agent|Jetson Image, AP Driver, DLPC/ITE Payload 및 Update Adapter 생성|
|Test Agent|Unit, Integration, Performance, Security 및 HIL 시험|
|SonarQube|Coverage, 정적 분석, 보안 규칙 및 Quality Gate|
|Docker Registry|빌드 Toolchain과 시험 환경의 재현 가능한 Container Image 저장|
|Artifact & Log DB|Jenkins Build, SonarQube, 시험 결과, 서명 및 장비 적용 결과 저장|
|NGINX|OTA API와 Package Download Endpoint의 Reverse Proxy 및 TLS 종단|
|OpenLDAP|개발자·검토자·Release Manager·Service Engineer의 인증과 역할 관리|
|File Server / Backup|서명된 OTA Bundle, Release Evidence 및 Audit Log의 백업|
|Cron/Rsync, Parity, Hot Spare|Git, Artifact, Log, Registry 데이터의 기존 백업 체계 활용|

## 추가된 페이지

### 07. Self-hosted DevOps OTA 연계 구성도

다음 전체 흐름을 구성했습니다.

**개발자/Redmine/SCM → Jenkins → SonarQube → Build Agent → HIL Test Agent → Composite Bundle → 격리 Signing Agent → Immutable Artifact Store → GitOps Release Channel → NGINX → ACS**

배포 경로는 두 가지입니다.

- **Primary:** ACS가 사내 OTA Endpoint에서 직접 HTTPS/mTLS 다운로드
- **Optional:** 폐쇄망 현장에서 iPad가 패키지를 받아 ACS로 Local Relay

장비 적용 결과는 Device Serial, Package ID, 단계, 현재 Version Set, 실패 코드와 함께 다시 Artifact & Log DB 및 Redmine으로 연결하도록 표현했습니다.

### 08. CI/CD → Signed OTA Release Pipeline

다음 Release Gate를 추가했습니다.

1. Redmine Issue 및 변경 요청 등록
2. SCM Commit과 Merge Request 검토
3. Jenkins의 재현 가능한 빌드
4. Coverage 및 SonarQube Quality Gate
5. Jetson·DLPC·ITE가 연결된 HIL/E2E 시험
6. 세 업데이트 대상을 하나의 Composite Bundle로 패키징
7. 격리된 Signing Agent에서 Manifest 서명
8. Immutable Artifact 게시
9. GitOps 방식으로 `dev → pilot → stable` 승격
10. iPad 승인 또는 예약 후 ACS 배포
11. 장비 Health 결과에 따라 확대 배포 또는 Rollback

## 함께 수정한 기존 페이지

- **00 설계 전제:** 사내 Self-hosted DevOps 활용을 확정 전제로 추가
- **01 전체 기능 구성도:** 기존의 일반적인 OTA 서버 영역을 실제 사내 Release Chain으로 교체
- **02 패키지 및 신뢰체계:** Jenkins, 격리 Signing, Artifact Store, GitOps Channel, NGINX 및 mTLS 반영
- **06 발주사 협의 항목:** Signing Key 운영, Device 인증서, 방화벽/VPN, GitOps 정책, Traceability와 Backup 항목 추가

## 중요 설계 원칙

**OpenLDAP는 사람 인증에 사용하고, ACS 장비 인증에는 사용하지 않도록 분리**했습니다. ACS는 장비별 인증서 기반의 mTLS를 사용하는 구조입니다.

Signing Private Key는 Jenkins Workspace나 일반 파일 서버에 저장하지 않고, **격리 Signing Agent 또는 HSM/Offline Key**를 사용하는 것으로 표시했습니다. 이 항목은 현재 인프라에 이미 존재한다고 가정하지 않고 **추가 권장 구성요소**로 구분했습니다.

기존 Cron/Rsync 및 디스크 이중화 체계는 Artifact와 Log 백업에 활용할 수 있지만, **Signing Private Key는 일반 백업과 분리된 보안 백업 정책**이 필요합니다.

또한 Release Server와 Jenkins는 패키지를 만들어 배포하는 주체이고, 실제 업데이트가 시작된 후의 단계·복구 상태는 계속 **ACS의 Local Journal과 Last-good 정보가 기준**이 되도록 유지했습니다. 따라서 배포 서버 또는 iPad 연결이 끊겨도 ACS가 독립적으로 완료하거나 안전 복구할 수 있습니다.




![[ACS_OTA_SelfHosted_DevOps_Architecture_Preview(1) 1.svg]]

![[ACS_OTA_Functional_Concept_Design_SelfHosted_DevOps_v2.drawio]]

![[ACS_OTA_Functional_Concept_Design(1).drawio]]

![[ACS_OTA_SelfHosted_DevOps_Architecture_Preview.svg]]