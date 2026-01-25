# 산출물 제출 안내

## 1. 산출물 대상

* **Ubuntu 24 환경에 존재하는 프로젝트 전체 소스코드**
* 실행 및 검토에 필요한 파일만 포함
* 빌드 결과물, 캐시 파일은 제외

---

## 2. Ubuntu 24에서 소스코드 압축하기

### 2-1. zip 패키지 install
```bash
sudo apt install zip
```

### 2-2. 압축하기:

```bash
cd ~/projects/koreanit-PBL

zip -r web-app.zip web-app \
  -x "web-app/node_modules/*" \
  -x "web-app/demo/build/*" \
  -x "web-app/demo/dist/*" \
  -x "web-app/.git/*"
```

---

## 3. Windows에서 scp로 파일 가져오기

### 3-1. Windows 터미널(또는 PowerShell) 실행

### 3-2. scp 명령어 형식

```bash
scp 사용자명@서버IP:/경로/project.zip .
```

### 3-3. 예시

```bash
cd ~/Downloads
scp ubuntu24:~/projects/koreanit-PBL/web-app.zip .
```

* 실행 위치: Windows에서 파일을 저장하고 싶은 폴더

---

## 4. 제출 파일 기준

* 제출 파일명: `web-app.zip` -> `이름.zip` 변경
* 압축 해제 시 **프로젝트 루트 구조가 유지되어야 함**
* 소스코드 누락 시 산출물 미인정

---

## 5. 주의 사항

* 압축 파일 안에 **다시 zip 파일이 포함되지 않도록 주의**
* 실행에 필요한 설정 파일은 포함
* 개인 정보(API 키, 비밀번호 등)는 제거 후 제출
