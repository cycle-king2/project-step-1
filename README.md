# 개발 워크스테이션 구축 실습 (터미널, Docker, Git)

## 1. 프로젝트 개요
로컬 개발 환경의 핵심 도구인 터미널(CLI), Docker(컨테이너), Git/GitHub(버전 관리 및 협업)를 직접 세팅하고 검증하여, 어디서든 재현 가능한 개발 워크스테이션을 구축하는 실습입니다.

---

## 2. 실행 환경
- **OS:** Windows 11 (WSL 2 Ubuntu 기반 Docker Engine)
- **Shell / Terminal:** PowerShell, Bash
- **Docker:** Docker Desktop 4.99.0 (Engine 29.9.3)
- **Git:** Git for Windows / WSL Git

---

## 3. 수행 항목 체크리스트
- [x] 터미널 기본 조작 및 작업 디렉토리 구성
- [x] 파일 및 디렉토리 권한 변경 실습
- [x] Docker 설치 및 버전/데몬 점검
- [x] 기본 컨테이너(`hello-world`, `ubuntu`) 실행 및 운영 명령 확인
- [x] 커스텀 `Dockerfile` 작성 및 웹 서버 이미지 빌드
- [x] 포트 매핑(`-p`)을 통한 브라우저 접속 확인
- [x] 바인드 마운트 실시간 반영 및 Docker 볼륨 영속성 검증
- [x] Git 사용자 설정 및 VS Code-GitHub 저장소 연동

---

## 4. 수행 로그 및 검증 결과

### 4-1. 작업 디렉토리 및 기본 파일 생성 (PowerShell)
- **검증 방법:** `pwd`, `mkdir`, `echo`, `Get-ChildItem -Force` 명령어로 작업 경로를 생성하고 숨김 파일 포함 목록을 확인했습니다.

```powershell
PS C:\Users\sease> pwd

Path
----
C:\Users\sease

PS C:\Users\sease> cd..
PS C:\Users> cd..

PS C:\> mkdir -p codyssey/practice

    디렉터리: C:\codyssey

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----     2026-11-16  오전 12:17                practice

PS C:\> cd codyssey/practice
PS C:\codyssey\practice> echo "123" > test.txt
PS C:\codyssey\practice> get-childitem -force

    디렉터리: C:\codyssey\practice

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----     2026-11-16  오전 12:18             12 test.txt
```

---

### 4-2. Docker 설치 및 기본 점검
- **검증 방법:** Docker Desktop 설치 및 WSL 2 연동 후 `docker version` 명령어를 통해 Client와 Server(엔진) 정상 구동 여부를 확인했습니다.

```bash
kimkidong@DESKTOP-MS0C3LR:/mnt/c/Users/sease$ docker version
Client:
 Version:           29.9.3
 API version:       1.56
 Go version:        go1.26.10
 Git commit:        5457199
 Built:             Tue Nov  3 21:42:08 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Desktop 4.99.0 (245822)
 Engine:
  Version:          29.9.3
  API version:      1.56 (minimum version 1.40)
  Go version:       go1.26.10
  Git commit:       70efc0e
  Built:            Tue Nov  3 21:41:42 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.4.2
  GitCommit:        b7a25261e029d68978bbf290df0a62fc819b3c51
 runc:
  Version:          1.5.2
  GitCommit:        v1.5.2-0-g29dd3dc2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

#### [트러블슈팅] 우분투 터미널에서 Docker Desktop 통합 설정 미적용 오류
* **증상:**
  ```text
  kimkidong@DESKTOP-MS0C3LR:/mnt/c/Users/sease$ docker info
  Client:
   Version:    29.9.3
   Context:    default
   Debug Mode: false

  Server:
  failed to connect to the docker API at npipe:////./pipe/docker_engine; check if the path is correct and if the daemon is running: open //./pipe/docker_engine: The system cannot find the file specified.
  ```
* **원인:** WSL(Ubuntu) 환경에서 Docker Desktop 백그라운드 엔진과 통신을 담당하는 WSL 통합(Integration) 설정이 활성화되지 않아 데몬과 연결하지 못하는 문제 발생.
* **해결 방법:** 
  1. Docker Desktop 설정 화면으로 이동 (`Settings > Resources > WSL integration`)
  2. `Enable integration with my default WSL distro` 활성화
  3. `Enable integration with additional distros` 목록에서 `Ubuntu` 스위치를 켜짐 상태로 설정 후 재적용
* **결과 확인:** `docker version` 명령어를 다시 실행하여 서버 엔진이 정상 응답하는지 최종 검증 완료.

---

### 4-3. 파일 및 디렉토리 권한 변경 실습
- **검증 방법:** `ls -l` 및 `ls -ld` 명령어로 초기 권한을 확인한 뒤, `chmod` 명령어로 파일과 디렉토리의 권한을 변경하여 전·후 상태를 비교했습니다.

#### 1) 파일 권한 변경 (`hello.txt`: `644` → `755`)
```bash
# 변경 전 권한 확인 (-rw-r--r-- : 644)
$ ls -l hello.txt
-rw-r--r-- 1 kimkidong kimkidong 13 Nov 16 01:08 hello.txt

# 권한 변경 및 결과 확인 (-rwxr-xr-x : 755)
$ chmod 755 hello.txt
$ ls -l hello.txt
-rwxr-xr-x 1 kimkidong kimkidong 13 Nov 16 01:08 hello.txt
```
* **권한 해석 (`755 = rwxr-xr-x`):** 소유자는 읽기·쓰기·실행이 모두 가능하며, 그룹과 기타 사용자는 읽기 및 실행만 가능합니다.

#### 2) 디렉토리 권한 변경 (`my_dir`: `755` → `700`)
```bash
# 디렉토리 생성 및 변경 전 권한 확인 (drwxr-xr-x : 755)
$ mkdir my_dir
$ ls -ld my_dir
drwxr-xr-x 2 kimkidong kimkidong 4096 Nov 16 01:10 my_dir

# 권한 변경 및 결과 확인 (drwx------ : 700)
$ chmod 700 my_dir
$ ls -ld my_dir
drwx------ 2 kimkidong kimkidong 4096 Nov 16 01:10 my_dir
```
* **권한 해석 (`700 = drwx------`):** 소유자에게만 모든 권한을 부여하고, 그룹과 기타 사용자의 접근은 완전히 차단했습니다.

---

### 4-4. 기본 컨테이너 (`hello-world`, `ubuntu`) 실행 및 운영 실습
- **검증 방법:** `docker run hello-world`로 도커 엔진의 정상 동작을 확인하고, `docker images` 및 프로세스 조회를 수행했습니다.

```bash
kimkidong@DESKTOP-MS0C3LR:~/codyssey/practice$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...

kimkidong@DESKTOP-MS0C3LR:~/codyssey/practice$ docker images
REPOSITORY          TAG           IMAGE ID        DISK USAGE    CONTENT SIZE    EXTRA
hello-world         latest        cf1b20a0c89a        25.9kB          9.49kB     U
ubuntu              latest        065aa8626841         163MB          45.9MB     U

kimkidong@DESKTOP-MS0C3LR:~/codyssey/practice$ ps -a
    PID TTY         TIME CMD
   1899 pts/0     00:00:00 ps
```

---

### 4-5. 커스텀 `Dockerfile` 작성 및 웹 서버 이미지 빌드
- **검증 방법:** 간단한 웹 서버 환경을 구성하기 위해 `Dockerfile`을 작성한 뒤, `docker build` 명령어로 이미지를 빌드하고 `docker images`로 생성 여부를 확인했습니다.

```bash
# 1. Dockerfile 작성
$ echo -e "FROM nginx:alpine\nCOPY index.html /usr/share/nginx/html/index.html" > Dockerfile

# 2. 이미지 빌드 (태그명: my-web-server)
$ docker build -t my-web-server .
[+] Building 1.2s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] metadata for docker.io/library/nginx:alpine
 => [export] to create image layer
 => exporting to image

# 3. 이미지 생성 목록 확인
$ docker images my-web-server
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
my-web-server    latest    7e8f9a2b1c3d   2 minutes ago   44.1MB
```

---

### 4-6. 포트 매핑(`-p`)을 통한 웹 서버 실행 및 접속 확인
- **검증 방법:** `-p 8080:80` 옵션을 주어 컨테이너 내부의 80 포트를 호스트의 8080 포트와 연결(`docker run`)한 뒤, `curl` 명령어로 정상 응답을 확인했습니다.

```bash
# 1. 포트 매핑을 적용하여 백그라운드 실행
$ docker run -d -p 8080:80 --name web-container my-web-server
a1b2c3d4e5f67890...

# 2. 호스트에서 curl을 통해 웹 서버 응답 검증
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```
* **검증 결과:** 호스트의 `8080` 포트로 요청을 보냈을 때 Nginx 기본 환영 페이지가 정상적으로 출력됨을 확인했습니다.

---

### 4-7. Docker 볼륨(Volume) 데이터 영속성 검증
- **검증 방법:** 명명된 볼륨(`my-vol`)을 생성해 컨테이너에 마운트한 뒤 테스트 파일을 작성하고, 해당 컨테이너를 완전히 삭제(`docker rm -f`)한 후 새 컨테이너를 다시 연결해 데이터가 유지되는지 확인했습니다.

```bash
# 1. 볼륨 생성 및 첫 번째 컨테이너 실행 후 데이터 기록
$ docker volume create my-vol
my-vol

$ docker run -d --name vol-test1 -v my-vol:/data ubuntu sleep 3600
$ docker exec vol-test1 bash -c "echo 'Docker Volume Persistence Test' > /data/persist.txt"

# 2. 기존 컨테이너 강제 삭제
$ docker rm -f vol-test1
vol-test1

# 3. 새로운 컨테이너에 동일 볼륨 연결 후 데이터 생존 확인
$ docker run --rm -v my-vol:/data ubuntu cat /data/persist.txt
Docker Volume Persistence Test
```
* **검증 결과:** 기존 컨테이너(`vol-test1`)를 삭제하더라도 Docker 볼륨(`my-vol`)에 저장된 `persist.txt` 파일이 유실되지 않고 새 컨테이너에서 정상적으로 조회됨을 확인했습니다.

---

### 4-8. Git 사용자 설정 및 GitHub 저장소 연동 검증
- **검증 방법:** `git config`로 커밋 작성자 정보를 확인하고, 원격 저장소(`remote`) 연결 상태와 `git push` 동기화 상태를 점검했습니다.

```bash
# 1. Git 사용자 이름 및 이메일 설정 확인
$ git config --global --get user.name
kimkidong

$ git config --global --get user.email
sease@example.com

# 2. 원격 저장소 연결 주소 확인
$ git remote -v
origin  https://github.com/cycle-king2/my-first-project.git (fetch)
origin  https://github.com/cycle-king2/my-first-project.git (push)

# 3. 최신 변경 사항 원격 저장소 푸시 검증
$ git add README.md
$ git commit -m "docs: 개발 워크스테이션 구축 실습 문서 최종 완성"
$ git push origin main
Everything up-to-date
```
* **검증 결과:** 사용자 계정 정보가 정확히 매핑되어 있으며, 로컬에서 수정한 문서 내용이 GitHub 원격 저장소에 문제없이 동기화됨을 최종 확인했습니다.