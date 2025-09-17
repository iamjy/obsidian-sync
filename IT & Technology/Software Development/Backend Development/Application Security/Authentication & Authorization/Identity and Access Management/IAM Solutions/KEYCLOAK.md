# KEYCLOAK 완전 초보자를 위한 체계적 학습 가이드

## 📋 1단계: 기본 개념 이해 (1-2주)

### **Keycloak이란?**
Keycloak은 Red Hat에서 개발한 오픈소스 Identity and Access Management (IAM) 솔루션으로, 인증(Authentication)과 인가(Authorization)를 쉽게 해주고 SSO(Single-Sign-On)을 가능하게 해주는 서비스입니다.

### **핵심 개념 학습**
- **SSO (Single Sign-On)**: 한번의 로그인을 통해 그와 연결된 여러가지 다른 사이트들을 자동으로 접속하여 이용할 수 있도록 하는 방법
- **Realm**: 인증, 권한 부여가 적용되는 범위의 단위로, SSO가 적용되는 범위가 하나의 Realm 단위
- **Client**: 인증, 권한 부여 행위를 수행할 어플리케이션을 나타내는 단위
- **User**: 인증을 필요로하는 사용자
- **Role**: User에게 부여할 권한의 내용

### **학습 자료**
1. **공식 문서**: Keycloak 공식 가이드 (https://www.keycloak.org/guides)
2. **한국어 블로그**: [Keycloak] 개념부터 실행까지 - 기본 개념 이해에 좋음

---

## 🛠️ 2단계: 설치 및 환경 구축 (1주)

### **설치 방법 선택**
1. **Docker 사용 (권장)**: `docker run -d --name keycloak -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin quay.io/keycloak/keycloak:legacy`
2. **직접 설치**: Java 11 이상 설치 후 Keycloak 바이너리 다운로드

### **실습 순서**
1. **환경 준비**: Java 11 이상 또는 Docker 설치
2. **Keycloak 실행**: 관리자 계정 생성 (export KEYCLOAK_ADMIN=admin, export KEYCLOAK_ADMIN_PASSWORD=123)
3. **Admin Console 접속**: http://localhost:8080 접속하여 관리자 콘솔 확인

### **학습 자료**
- 자습서-Keycloak 설치 구성 및 테스트 단계 - NGINX STORE
- Keycloak 설치하기: Ubuntu 및 CentOS

---

## 🎯 3단계: 기본 설정 실습 (1-2주)

### **Realm 생성 및 설정**
1. **새 Realm 생성**: Admin Console에서 Create Realm 클릭하여 새 Realm 생성
2. **사용자 생성**: Users 메뉴에서 사용자 추가 및 비밀번호 설정
3. **Client 등록**: 애플리케이션을 보호하기 위해 Keycloak 인스턴스에 application 등록

### **기본 테스트**
- 테스트 방법은 Keycloak의 접속 정보를 직접 입력하는 방식으로 접근
- Account Console을 통한 사용자 관리 테스트

### **학습 자료**
- Getting started using KEYCLOAK - Medium
- Red Hat build of Keycloak Getting Started Guide

---

## 💻 4단계: 애플리케이션 연동 실습 (2-3주)

### **Spring Boot 연동**
1. **기본 연동**: Spring Boot와 Keycloak을 이용한 OAuth2 설정
2. **JWT 토큰 처리**: Spring Security를 활용한 토큰 검증
3. **실습 프로젝트**: Spring Boot REST API와 React 애플리케이션을 Keycloak으로 보안 구성

### **React 연동**
1. **프론트엔드 인증**: React Vite와 Keycloak OAuth 2 & OpenID 연동
2. **토큰 관리**: JWT 토큰을 이용한 API 호출
3. **풀스택 예제**: React, Spring Boot, Keycloak을 활용한 완전한 인증 시스템

### **실습 프로젝트**
- GitHub - springboot-react-keycloak 프로젝트
- MyToDoList 애플리케이션 보안 구현

---

## 🔧 5단계: 고급 기능 및 운영 (2-3주)

### **고급 설정**
1. **외부 데이터베이스 연동**: H2에서 PostgreSQL, MySQL 등으로 변경
2. **SSL/HTTPS 설정**: 프로덕션 환경을 위한 보안 설정
3. **소셜 로그인**: Google, Facebook 등 Identity Provider 연동

### **실제 운영 환경**
1. **Kubernetes 배포**: Helm을 사용한 Kubernetes 클러스터에 Keycloak 설치
2. **성능 최적화**: 캐싱, 클러스터링 설정
3. **모니터링**: 메트릭 수집 및 health check 설정

### **학습 자료**
- Keycloak을 이용한 SSO 구축(web + wifi + ssh) - SOCAR Tech Blog
- Keycloak Server Administration Guide

---

## 📚 추천 학습 리소스

### **공식 자료**
1. **Keycloak 공식 사이트**: https://www.keycloak.org
2. **공식 문서**: https://www.keycloak.org/guides
3. **Server Developer Guide**: 커스텀 프로바이더 개발을 위한 가이드

### **실습 코드**
1. **Baeldung 튜토리얼**: Spring Boot와 Keycloak 연동 가이드
2. **GitHub 예제들**: springboot-react-keycloak 등 실무 예제
3. **Medium 아티클들**: 최신 통합 가이드

---

## 🎯 학습 로드맵 요약

```
주차별 학습 계획:
1-2주: 기본 개념 및 이론 학습
3주: 설치 및 기본 환경 구축
4-5주: Realm, Client, User 설정 실습
6-8주: Spring Boot 연동 실습
9-11주: React 연동 및 풀스택 구현
12-14주: 고급 기능 및 운영 환경 구축
```

### **핵심 학습 포인트**
1. **개념 이해**: IAM, OAuth2, OpenID Connect, JWT 이해
2. **실습 중심**: 직접 구현하면서 배우는 것이 가장 효과적
3. **실무 연결**: Spring Boot, React 등 실제 사용하는 기술과 연동
4. **운영 고려**: 보안, 성능, 모니터링 등 운영 환경 고려사항

**추측입니다**: 이 학습 로드맵을 따라 진행하면 약 3-4개월 내에 Keycloak을 실무에서 활용할 수 있는 수준에 도달할 것으로 예상됩니다.

## 📝 메모 및 학습 노트

### 학습 진행 상황
- [ ] 1단계: 기본 개념 이해
- [ ] 2단계: 설치 및 환경 구축
- [ ] 3단계: 기본 설정 실습
- [ ] 4단계: 애플리케이션 연동 실습
- [ ] 5단계: 고급 기능 및 운영

### 중요한 링크들
- [Keycloak 공식 사이트](https://www.keycloak.org)
- [Keycloak 공식 문서](https://www.keycloak.org/guides)
- [Getting Started Guide](https://www.keycloak.org/getting-started)

### 실습 환경
```bash
# Docker로 Keycloak 실행
docker run -d --name keycloak -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:legacy start-dev
```

### 관련 태그
#keycloak #iam #sso #authentication #authorization #spring-boot #react #oauth2 #openid-connect #jwt