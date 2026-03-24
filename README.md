
# 🚀 springboot-jar-deploy

> **SCP 전송 한 번으로 완성되는 무중단 자동 재배포 파이프라인**  
> **빌드와 배포 사이의 공백을 자동화로 채우다**

로컬 Windows 환경에서 빌드한 JAR 파일을 SCP로 전송하면, Ubuntu 서버가 이를 자동으로 감지하여 기존 프로세스를 정리하고 새로운 서비스를 실행하는 자동 배포 실습 프로젝트입니다.

---
## 🙋‍♀️ 팀원 소개

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/Yewon0106.png" width="200px" /><br />
      <a href="https://github.com/Yewon0106">유예원</a>
    </td>
    <td align="center">
      <img src="https://github.com/minykang.png" width="200px" /><br />
      <a href="https://github.com/minykang">강민영</a>
    </td>
  </tr>
</table>

---
## 📌 프로젝트 개요

springboot-jar-deploy는  
Windows에서 빌드한 Spring Boot JAR 파일을 Ubuntu 서버에 전송하면,  
서버가 이를 자동 감지해 기존 프로세스를 종료하고 새 버전을 실행하는 배포 자동화 프로젝트입니다.

기존에는 JAR 전송 후 서버에 직접 접속해  
프로세스 종료와 재실행을 수동으로 처리해야 했습니다.

이 프로젝트에서는 inotify-tools와 Bash 스크립트를 활용해  
**감지 → 종료 → 실행** 흐름을 자동화했습니다.

---

## 🔄 Deployment Comparison

| 분류 | ❌ 기존 수동 배포 | ✅ 자동화 배포 |
|---|---|---|
| 과정 | SCP 전송 → 서버 접속 → PID 검색 → 프로세스 종료 → JAR 실행 | SCP 전송 → 자동 감지 → 자동 종료 → 자동 실행 |
| 소요 시간 | 약 2~3분 | 감지 후 5~10초 내 |
| 실수 가능성 | 포트 종료 누락, 경로 오타 등 휴먼 에러 발생 가능 | 스크립트 기반으로 일관된 배포 가능 |

---

## 🎖️ 주요 기능

### 1. 실시간 파일 시스템 감시
inotifywait를 활용하여 /home/ubuntu/deploy 폴더에서 **app.jar** 파일의 close_write 이벤트를 감지합니다.  
파일 전송이 완전히 끝난 시점을 기준으로 배포 스크립트를 실행해 불완전한 파일 상태에서 배포되지 않도록 구성했습니다.

### 2. 포트 충돌 자동 방지
배포 전에 lsof 명령어로 8080 포트를 사용 중인 기존 Java 프로세스를 탐지하고, kill -15로 안전하게 종료합니다.  
이를 통해 포트 충돌로 인해 새 애플리케이션이 실행되지 않는 상황을 줄였습니다.

### 3. 중복 실행 방지
파일 전송 과정에서 발생할 수 있는 중복 이벤트를 처리하기 위해 **COOLDOWN** 로직을 적용했습니다.  
짧은 시간 안에 같은 파일 이벤트가 여러 번 발생해도 한 번만 배포가 수행되도록 구성해 불필요한 재실행을 방지했습니다.

---

## 🧰 기술 스택

| 분류 | 기술 |
|---|---|
| OS | Ubuntu 24.04 LTS, Windows 11 |
| Framework | Spring Boot 3.x, Java 17 |
| Monitoring | inotify-tools |
| Scripting | Bash Shell Script |
| Connectivity | Git Bash, SCP |

---

## ⚙️ 시스템 아키텍처

- **Windows**
  - ./gradlew build로 JAR 파일 생성
  - scp로 Ubuntu 서버의 배포 폴더에 전송

- **Ubuntu**
  - inotifywait가 app.jar 파일 변경 감지
  - deploy.sh 실행

- **배포 흐름**
  - 기존 8080 포트 점유 프로세스 종료
  - 잠시 대기
  - 새 버전 JAR 실행
  - 로그 파일 기록

---

## 🚀 Workflow Steps

### 1. Build & Push
로컬에서 애플리케이션을 빌드하고 서버로 JAR 파일을 전송합니다.

```bash
./gradlew clean build
scp ./build/libs/*.jar ubuntu@server:/home/ubuntu/deploy/app.jar
```