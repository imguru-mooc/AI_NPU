# NPU 보드 테스트 실습 가이드

Raspberry Pi 4 + Tachy-Shield(NPU) 보드를 처음 켜는 것부터 객체 검출 예제를 실행하는 것까지의 전체 절차를 다룹니다.

---

## 0. 실습 개요

### 학습 목표

| 단계 | 내용 |
| --- | --- |
| 1 | 보드에 모니터·키보드·마우스를 연결하고 정상 부팅을 확인한다 |
| 2 | Raspberry Pi OS 데스크탑에서 Wi-Fi를 설정하고 IP 주소를 확인한다 |
| 3 | Windows 11에서 PuTTY로 보드에 원격 접속한다 |
| 4 | FileZilla로 `npu_board.zip` 실습 파일을 보드에 업로드한다 |
| 5 | Tachy-Shield를 부팅하고 YOLOv9 객체 검출 예제를 실행한다 |

### 준비물

**보드 측**
- Raspberry Pi 4 (Tachy-Shield 결합 상태)
- Raspberry Pi 4 전용 USB-C 전원 어댑터
- micro-HDMI to HDMI 케이블
- USB 키보드, USB 마우스
- 모니터

**PC 측**
- Windows 11 노트북/데스크탑
- 보드와 **같은 Wi-Fi 네트워크**에 접속 가능한 환경
- 실습 파일 `npu_board.zip`

> ⚠️ Tachy-Shield는 **전원이 꺼진 상태**에서 20x2 핀 헤더에 결합되어 있어야 합니다. 이미 결합된 보드를 받았다면 결합 상태와 MIPI FFC 케이블 체결만 눈으로 확인해 주세요.

---

## 1. 하드웨어 연결

### 1.1 연결 순서

전원은 **가장 마지막에** 연결합니다. 다른 케이블을 꽂는 도중 전원이 인가되면 보드가 손상될 수 있습니다.

```text
① micro-HDMI 케이블  →  모니터
② USB 키보드         →  USB 포트
③ USB 마우스         →  USB 포트
④ USB-C 전원         →  마지막에 연결  ← 여기서 부팅 시작
```

### 1.2 HDMI 포트 주의사항

Raspberry Pi 4에는 **micro-HDMI 포트가 2개** 있습니다. 반드시 **HDMI0**(USB-C 전원 포트에 가까운 쪽)에 연결해야 합니다. HDMI1에만 연결하면 화면이 나오지 않는 경우가 있습니다.

```text
   ┌─────────────────────────────────────┐
   │  [USB-C]  [HDMI0]  [HDMI1]  [AUDIO] │
   │     ↑        ↑                      │
   │   전원    여기에 연결                 │
   └─────────────────────────────────────┘
```

일반 HDMI 케이블은 커넥터 크기가 맞지 않습니다. **micro-HDMI** 케이블 또는 변환 젠더를 사용해 주세요.

### 1.3 USB 포트

키보드와 마우스는 어느 USB 포트에 꽂아도 무방합니다. 파란색 포트가 USB 3.0, 검은색 포트가 USB 2.0이며 키보드·마우스는 USB 2.0(검은색) 쪽을 사용하는 편이 좋습니다.

### 1.4 부팅 확인

전원을 연결하면 자동으로 부팅이 시작됩니다.

| LED | 정상 상태 |
| --- | --- |
| 빨간색 (PWR) | 계속 켜져 있음 |
| 초록색 (ACT) | 불규칙하게 깜빡임 (SD 카드 접근 중) |

- 빨간 LED만 켜지고 초록 LED가 전혀 깜빡이지 않으면 → SD 카드 문제
- 화면이 계속 검은색이면 → HDMI0 포트에 연결했는지 확인 후 재부팅

부팅에는 약 30초~1분이 소요되며, 완료되면 Raspberry Pi OS 데스크탑 화면이 나타납니다.

---

## 2. 로그인 계정 정보

이 보드에는 Raspberry Pi OS(라즈비안) **데스크탑 버전**이 이미 설치되어 있습니다.

| 항목 | 값 |
| --- | --- |
| 사용자 이름 (ID) | `pi` |
| 비밀번호 (Password) | `raspberry` |
| 기본 홈 디렉터리 | `/home/pi` |

> 이 계정 정보는 이후 **PuTTY 접속**과 **FileZilla 접속**에서 동일하게 사용합니다. 반드시 기억해 주세요.

자동 로그인이 설정되어 있으면 데스크탑 화면이 바로 나타납니다. 로그인 창이 뜬다면 위 정보를 입력합니다.

---

## 3. Wi-Fi 설정 (데스크탑 GUI)

데스크탑 버전이므로 명령어 없이 화면에서 설정할 수 있습니다.

### 3.1 설정 절차

1. 화면 **우측 상단**의 네트워크 아이콘을 클릭합니다.
   - 아직 연결되지 않았다면 상하 화살표(↑↓) 모양
   - 연결되면 Wi-Fi 신호 막대 모양으로 바뀝니다
2. 주변 Wi-Fi 목록이 나타나면 **접속할 SSID**를 클릭합니다.
3. 비밀번호 입력 창이 뜨면 **Pre Shared Key** 칸에 Wi-Fi 비밀번호를 입력하고 **OK**를 누릅니다.
4. 몇 초 후 아이콘이 Wi-Fi 신호 막대 모양으로 바뀌면 연결 완료입니다.

> 💡 처음 설정 시 국가(Country) 선택 창이 나타나면 **KR (Korea, Republic of)** 를 선택합니다. 국가가 설정되지 않으면 Wi-Fi가 비활성화되어 목록이 보이지 않습니다.

### 3.2 IP 주소 확인 (중요)

PuTTY와 FileZilla 접속에 필요하므로 **반드시 메모**해 두세요.

**방법 A — 마우스만 사용**

Wi-Fi 아이콘 위에 마우스 커서를 올리고 잠시 기다리면 툴팁으로 IP 주소가 표시됩니다.

**방법 B — 터미널 사용 (권장)**

데스크탑 상단의 터미널 아이콘(`>_`)을 클릭하고 다음을 입력합니다.

```bash
hostname -I
```

출력 예시:

```text
192.168.0.42
```

> 여러 개가 출력되면 보통 **첫 번째 주소**가 Wi-Fi 주소입니다. 이 문서에서는 이후 이 주소를 `192.168.0.42`로 표기하니, 본인 보드의 실제 주소로 바꿔서 진행해 주세요.

### 3.3 SSH 활성화 확인

PuTTY로 접속하려면 보드에서 SSH가 켜져 있어야 합니다. 이미 활성화되어 있을 수 있으나, 접속이 안 될 경우 아래를 확인합니다.

**GUI 방법**

```text
좌측 상단 라즈베리 메뉴 → 기본 설정(Preferences)
  → Raspberry Pi Configuration → Interfaces 탭
  → SSH 항목을 Enabled(사용)로 전환 → OK
```

**터미널 방법**

```bash
sudo raspi-config
# 3 Interface Options → I2 SSH → Yes → Finish
```

현재 상태는 다음 명령으로 확인할 수 있습니다.

```bash
sudo systemctl status ssh
```

`active (running)` 이 보이면 정상입니다.

---

## 4. Windows 11에서 PuTTY로 접속하기

보드에 직접 연결된 키보드로 작업할 수도 있지만, PC에서 원격 접속하면 명령어 복사·붙여넣기가 편해 실습 진행이 훨씬 수월합니다.

### 4.1 PuTTY 설치

1. 웹 브라우저에서 **PuTTY 공식 다운로드 페이지**로 이동합니다.
   - `https://www.putty.org` → Download 링크
2. **64-bit x86: `putty-64bit-<버전>-installer.msi`** 를 다운로드합니다.
3. 다운로드한 `.msi` 파일을 실행하고 `Next` → `Next` → `Install` 순으로 진행합니다.
4. 설치 완료 후 시작 메뉴에서 **PuTTY**를 실행합니다.

### 4.2 접속 설정

PuTTY 실행 후 첫 화면(Session)에서 다음과 같이 입력합니다.

| 항목 | 입력 값 |
| --- | --- |
| Host Name (or IP address) | `192.168.0.42` (3.2에서 확인한 IP) |
| Port | `22` |
| Connection type | `SSH` |
| Saved Sessions | `npu_board` (선택 사항 — 저장해두면 재사용 편리) |

설정 후 **Save**를 누르면 다음부터는 목록에서 더블클릭으로 바로 접속할 수 있습니다.

**Open** 버튼을 클릭합니다.

### 4.3 보안 경고 및 로그인

1. 최초 접속 시 **PuTTY Security Alert** 창이 나타납니다. 서버의 호스트 키를 처음 보기 때문이며, **Accept**를 클릭하면 됩니다.
2. 검은 터미널 창이 열리면서 로그인 프롬프트가 나타납니다.

```text
login as: pi
pi@192.168.0.42's password:
```

- `login as:` 에 **`pi`** 입력 후 Enter
- `password:` 에 **`raspberry`** 입력 후 Enter

> ⚠️ 비밀번호는 **입력해도 화면에 아무것도 표시되지 않습니다.** (`*` 표시조차 없음) 정상 동작이니 그대로 입력하고 Enter를 누르세요.

접속에 성공하면 다음과 같은 프롬프트가 나타납니다.

```text
pi@raspberrypi:~ $
```

### 4.4 참고 — Windows 11 기본 SSH 사용

PuTTY 없이 Windows 11에 내장된 OpenSSH 클라이언트를 사용할 수도 있습니다. PowerShell 또는 명령 프롬프트에서:

```powershell
ssh pi@192.168.0.42
```

### 4.5 PuTTY 사용 팁

| 동작 | 방법 |
| --- | --- |
| 붙여넣기 | 터미널 안에서 **마우스 오른쪽 클릭** (Ctrl+V 아님) |
| 복사 | 마우스로 드래그하면 자동 복사됨 |
| 글자 크기 조정 | Window → Appearance → Font 설정 |
| 접속 종료 | `exit` 입력 또는 창 닫기 |

---

## 5. FileZilla로 실습 파일 업로드

`npu_board.zip`(약 17MB)을 PC에서 보드로 전송합니다.

### 5.1 FileZilla 설치

1. `https://filezilla-project.org` 접속
2. **Download FileZilla Client** 클릭 (Server 아님에 주의)
3. Windows 64bit 버전 다운로드 후 실행
4. 설치 중 **추가 프로그램 설치 제안 화면**이 나오면 `Decline` 또는 체크 해제 후 진행합니다.

### 5.2 보드에 접속

FileZilla 상단의 **빠른 연결(Quickconnect)** 바에 다음을 입력합니다.

| 항목 | 입력 값 |
| --- | --- |
| 호스트(Host) | `sftp://192.168.0.42` |
| 사용자명(Username) | `pi` |
| 비밀번호(Password) | `raspberry` |
| 포트(Port) | `22` |

> 💡 호스트 앞에 반드시 **`sftp://`** 를 붙입니다. 생략하면 FTP로 접속을 시도해 실패합니다.

**빠른 연결** 버튼을 클릭합니다. 최초 접속 시 **알 수 없는 호스트 키** 경고가 나타나면 **확인(OK)** 을 누릅니다. (`항상 이 호스트를 신뢰` 체크 시 다음부터 생략됩니다)

### 5.3 파일 전송

접속에 성공하면 화면이 좌우로 나뉩니다.

```text
┌──────────────────────┬──────────────────────┐
│   로컬 사이트         │   리모트 사이트        │
│   (내 Windows PC)     │   (Raspberry Pi 보드)  │
│                      │                      │
│  C:\Users\...\다운로드 │  /home/pi            │
│    npu_board.zip  ───┼───→ (여기로 드래그)    │
└──────────────────────┴──────────────────────┘
```

1. **왼쪽(로컬 사이트)** 에서 `npu_board.zip`이 있는 폴더로 이동합니다. (보통 `다운로드` 폴더)
2. **오른쪽(리모트 사이트)** 경로가 `/home/pi` 인지 확인합니다. 아니라면 상단 경로 입력란에 `/home/pi`를 입력하고 Enter를 누릅니다.
3. 왼쪽의 `npu_board.zip` 파일을 **오른쪽 창으로 드래그 앤 드롭** 합니다.
   - 또는 파일을 우클릭 → **업로드(Upload)**
4. 하단의 전송 큐에서 진행률을 확인합니다. 완료되면 **성공한 전송** 탭으로 이동합니다.

### 5.4 업로드 확인

PuTTY 터미널로 돌아가 파일이 정상 도착했는지 확인합니다.

```bash
cd ~
ls -lh npu_board.zip
```

출력 예시:

```text
-rw-r--r-- 1 pi pi 17M Aug 21 10:23 npu_board.zip
```

### 5.5 압축 해제

```bash
cd ~
unzip npu_board.zip
```

> `unzip: command not found` 오류가 나면 `sudo apt install -y unzip` 으로 설치 후 다시 실행합니다.

압축 해제 후 구조를 확인합니다.

```bash
ls npu_board
```

```text
README.md  boot  object_detection_tracking
```

### 5.6 디렉터리 구조

```text
npu_board/
├── README.md                          ← 실행 방법 안내 문서
│
├── boot/                              ← Tachy-Shield 부팅용
│   ├── boot.py                        ← 부팅 실행 스크립트
│   ├── boot_binary/                   ← 보드에 올릴 펌웨어 4종
│   │   ├── spl.bin                    (0x0        번지)
│   │   ├── u-boot.bin                 (0x2000_0000 번지)
│   │   ├── image.ub                   (0x4000_0000 번지, 커널)
│   │   └── fpga_top.bin               (0x3000_0000 번지, FPGA)
│   └── tachy_rt/                      ← Tachy Runtime 라이브러리
│
└── object_detection_tracking/         ← 객체 검출 예제
    ├── app.py                         ← 검출 실행 스크립트
    ├── configs/
    │   └── face_detection_yolov9/
    │       └── post_process_256x416.json   ← 후처리 임계값 설정
    ├── params/
    │   ├── face_detection_yolov9_xwn4/     ← 모델 파라미터
    │   └── face_detection_yolov9_xwn4p50/  ← 모델 파라미터(pruning 50%)
    ├── src/
    │   ├── detection_utils.py         ← 검출 결과 변환 / ByteTrack 추적
    │   ├── yolov9_post_process.py     ← YOLOv9 후처리
    │   └── operations.py
    └── tachy_rt/                      ← Tachy Runtime 라이브러리
```

> 📌 `boot/`과 `object_detection_tracking/` **각 폴더 안에 `tachy_rt/`가 따로 들어있습니다.** 이 때문에 스크립트는 **반드시 해당 폴더 안에서 실행**해야 합니다. 다른 위치에서 실행하면 `ModuleNotFoundError: No module named 'tachy_rt'` 가 발생합니다.

---

## 6. README.md 내용 설명 및 실행

`npu_board/README.md`는 예제 실행 방법을 3단계로 안내합니다. 각 단계가 실제로 무엇을 하는지 함께 살펴봅니다.

### 6.0 사전 확인 — 가상환경 활성화

실행 전 Python 가상환경이 활성화되어 있는지 확인합니다.

```bash
source /opt/venv/bin/activate
```

프롬프트 앞에 `(venv)`가 표시되면 정상입니다. `~/.bashrc`에 자동 활성화가 설정되어 있다면 이미 표시되어 있을 수 있습니다.

```bash
python3 -m pip show tachy-rt      # Runtime 설치 확인
ls /dev/tachy*                     # SPI 제어 장치 노드 확인
ls /dev/video0                     # MIPI 수신 경로 확인
```

---

### 6.1 [1단계] Tachy Shield 부팅

**README 원문**

```bash
cd boot
python boot.py
```

**실제 실행**

```bash
cd ~/npu_board/boot
python boot.py
```

**이 단계에서 일어나는 일**

`boot.py`는 SPI 인터페이스를 통해 Tachy-Shield에 펌웨어를 올리고 NPU를 기동시킵니다. 내부 동작 순서는 다음과 같습니다.

1. `/dev/tachy*` 장치 노드의 소유권을 현재 사용자로 변경 (`sudo chown`)
2. GPIO 4번 핀을 Low → 3초 대기 → High로 전환하여 **하드웨어 리셋**
3. `boot_binary/` 안의 펌웨어 4개를 각각 지정된 메모리 주소로 전송
   - `spl.bin` → `0x0`
   - `u-boot.bin` → `0x2000_0000`
   - `fpga_top.bin` → `0x3000_0000`
   - `image.ub` → `0x4000_0000`
4. 부팅 완료 여부 확인

**정상 출력**

```text
[INFO] Success to boot. Check the status via uart or other api
Success to boot.
```

**참고 사항**

- `sudo` 사용으로 비밀번호를 물어볼 수 있습니다. → `raspberry` 입력
- 펌웨어 전송에 약 10~30초가 소요됩니다. 출력이 없어도 기다려 주세요.
- 부팅은 **전원 인가 후 1회만** 수행하면 됩니다. 예제를 여러 번 재실행할 때는 다시 부팅하지 않아도 됩니다.
- 단, **전원을 껐다 켰다면 반드시 다시 부팅**해야 합니다.
- `--path_firmware` 옵션으로 펌웨어 경로를 바꿀 수 있으며, 기본값은 `./boot_binary` 입니다. 이 상대 경로 때문에 반드시 `boot/` 폴더 안에서 실행해야 합니다.

---

### 6.2 [2단계] 객체 검출 실행

**README 원문**

```bash
cd ../object_detection_tracking
python app.py
```

**이 단계에서 일어나는 일**

`app.py`는 두 개의 스레드로 동작합니다.

```text
[NPU 경로 — 추론 스레드]                [카메라 경로 — 화면 스레드]
Tachy-Shield NPU                        /dev/video0 (YUYV, 1920x1080)
  YOLOv9 얼굴 검출 수행                        ↓
  annotation 발행                          프레임 읽기
       ↓                                       ↓
  ByteTrack 추적 ID 부여          ────→   박스 오버레이 그리기
       ↓                                       ↓
  터미널에 결과 출력                      1/3 크기로 축소해 화면 표시
```

주요 설정값은 `configs/face_detection_yolov9/post_process_256x416.json`에 정의되어 있습니다.

| 항목 | 값 | 의미 |
| --- | --- | --- |
| `SHAPES_INPUT` | `[256, 416, 3]` | NPU 입력 해상도 (H×W×C) |
| `OBJ_THRESHOLD` | `0.2` | 객체 신뢰도 임계값 |
| `NMS_THRESHOLD` | `0.2` | 중복 박스 제거 기준 |
| `N_MAX_OBJ` | `30` | 프레임당 최대 검출 개수 |
| `N_CLASSES` | `1` | 클래스 수 (얼굴 1종) |

> ⚠️ **PuTTY로 접속해서 실행하는 경우** — `app.py`는 OpenCV 창을 띄우기 때문에 SSH 세션에서 그대로 실행하면 `cannot open display` 오류가 발생합니다. 다음 중 하나를 선택하세요.
>
> **(A) 보드에 연결된 모니터의 터미널에서 직접 실행** (권장)
>
> **(B) PuTTY에서 실행하되 화면은 보드 모니터로 출력**
> ```bash
> export DISPLAY=:0
> python app.py
> ```

---

### 6.3 [3단계] 실행 결과 확인

**화면 출력**

카메라 영상 위에 초록색 사각형으로 검출 영역이 표시되고, 각 박스 위에 `track=#8 class=0 score=0.634` 형태의 라벨이 붙습니다. 창 크기는 원본의 1/3(640×360)로 축소되어 표시됩니다.

**터미널 출력**

```text
[INFO 2026-08-20 22:13:36.337] Detections: 0
[INFO 2026-08-20 22:13:36.377] Detections: 1
track_id=#8 class_id=0 confidence=0.6343 box=[ 47.884617 -11.337891 717.9808   666.2988  ]
[INFO 2026-08-20 22:13:36.405] Detections: 1
track_id=#8 class_id=0 confidence=0.2063 box=[ 59.423077 -19.77539  727.21155  655.75195 ]
[INFO 2026-08-20 22:13:36.442] Detections: 1
```

**출력 해석**

| 항목 | 의미 |
| --- | --- |
| `[INFO 시각]` | 결과가 수신된 시각 (밀리초까지 표시) |
| `Detections: N` | 해당 프레임에서 검출된 객체 개수 |
| `track_id=#8` | ByteTrack이 부여한 추적 ID. **같은 얼굴은 프레임이 바뀌어도 같은 번호를 유지**합니다 |
| `class_id=0` | 클래스 번호. 이 모델은 클래스가 1개(얼굴)이므로 항상 `0` |
| `confidence=0.6343` | 신뢰도. `OBJ_THRESHOLD=0.2` 이상인 것만 출력됩니다 |
| `box=[x0 y0 x1 y1]` | 좌상단·우하단 좌표 (좌표계 1920×1080 기준) |

> 💡 **좌표에 음수가 나오는 이유** — 위 예시의 `-11.337891`처럼 음수가 보이는 것은 정상입니다. 얼굴이 화면 가장자리에 걸쳐 있을 때 모델이 예측한 박스가 화면 밖으로 넘어간 경우이며, 실제 화면에 그릴 때는 `np.clip()`으로 화면 범위 안에 맞춰집니다.

> 💡 **`Detections: 0`이 계속 나온다면** — 카메라 앞에 얼굴이 없거나, 조명이 어둡거나, 거리가 너무 먼 경우입니다. 카메라에서 50cm~1m 거리에 얼굴을 위치시켜 보세요.

**종료 방법**

영상 창을 클릭해 포커스를 준 뒤 **`q`** 또는 **`Esc`** 키를 누릅니다. 터미널에서 `Ctrl+C`를 눌러도 종료됩니다.

---

## 7. 전체 실행 요약

지금까지의 절차를 한 번에 정리하면 다음과 같습니다.

```bash
# 0) 가상환경 활성화
source /opt/venv/bin/activate

# 1) 파일 업로드 후 압축 해제 (최초 1회)
cd ~
unzip npu_board.zip

# 2) Tachy-Shield 부팅 (전원 인가 후 1회)
cd ~/npu_board/boot
python boot.py
#   → "Success to boot." 확인

# 3) 객체 검출 실행
cd ../object_detection_tracking
export DISPLAY=:0        # PuTTY로 접속한 경우에만 필요
python app.py
#   → 영상 창에서 q 또는 Esc 로 종료
```

---

## 8. 문제 해결

### 하드웨어 / 접속

| 증상 | 확인 항목 | 조치 |
| --- | --- | --- |
| 화면이 나오지 않음 | HDMI 포트 위치 | **HDMI0**(전원 포트에 가까운 쪽)에 연결했는지 확인 후 재부팅 |
| 초록 LED가 깜빡이지 않음 | SD 카드 | SD 카드를 다시 삽입하거나 다른 카드로 교체 |
| Wi-Fi 목록이 비어 있음 | 국가 설정 | Raspberry Pi Configuration → Localisation → WLAN Country를 `KR`로 설정 |
| PuTTY `Connection refused` | SSH 활성화 | 보드에서 `sudo raspi-config` → Interface Options → SSH 활성화 |
| PuTTY `Network error: Connection timed out` | 네트워크 | PC와 보드가 **같은 Wi-Fi**에 있는지, IP가 맞는지 (`hostname -I`) 재확인 |
| PuTTY 비밀번호 입력이 안 되는 것 같음 | 정상 동작 | 비밀번호는 화면에 표시되지 않습니다. 그대로 입력 후 Enter |
| FileZilla 접속 실패 | 프로토콜 | 호스트 앞에 `sftp://`를 붙였는지 확인 |

### 실행

| 증상 | 확인 항목 | 조치 |
| --- | --- | --- |
| `ModuleNotFoundError: No module named 'tachy_rt'` | 실행 위치 / 가상환경 | 반드시 `boot/` 또는 `object_detection_tracking/` 폴더 **안에서** 실행. `source /opt/venv/bin/activate` 확인 |
| `ModuleNotFoundError: No module named 'cv2'` (또는 `supervision`) | 패키지 설치 | `pip install opencv-python supervision==0.24.0` |
| `Failed to boot` | 전원 / 결합 / 펌웨어 경로 | 전원을 껐다 켠 뒤 20x2 헤더 결합 상태와 `boot_binary/` 파일 존재 여부 확인 |
| `/dev/tachy*` 가 없음 | 드라이버 | `tachy_bs_host_driver` 설치 여부 확인 후 재부팅 |
| `/dev/video0` 가 없음 | MIPI 케이블 / overlay | FFC 케이블 방향과 `/boot/firmware/config.txt`의 `dtoverlay=dummy-csi-sensor,2lanes` 설정 확인 후 재부팅 |
| `cannot open /dev/video0 with V4L2` | 장치 점유 | 이전에 실행한 `app.py`가 남아있는지 확인 (`ps aux \| grep app.py`) 후 종료 |
| `cannot open display` / `qt.qpa.xcb` 오류 | GUI 출력 경로 | SSH 세션이면 `export DISPLAY=:0` 실행 후 재시도 |
| `save_model failed` / `make_standalone_instance failed` | 부팅 상태 | `boot.py`를 다시 실행해 Tachy-Shield를 재부팅 |
| 영상은 나오나 검출이 전혀 안 됨 | 거리 / 조명 | 카메라에서 50cm~1m 거리, 밝은 환경에서 재시도 |

---

## 9. 실습 체크리스트

진행하면서 각 항목을 확인해 주세요.

- [ ] micro-HDMI를 **HDMI0**에 연결하고 화면 출력 확인
- [ ] 키보드·마우스 연결 확인, 전원은 마지막에 연결
- [ ] `pi` / `raspberry` 로 데스크탑 로그인
- [ ] 데스크탑 우측 상단에서 Wi-Fi 연결 완료
- [ ] `hostname -I` 로 IP 주소 확인 및 메모: `____________________`
- [ ] Windows 11에 PuTTY 설치 후 SSH 접속 성공
- [ ] Windows 11에 FileZilla 설치 후 `sftp://` 로 접속 성공
- [ ] `npu_board.zip` 을 `/home/pi` 에 업로드 완료
- [ ] `unzip npu_board.zip` 으로 압축 해제 완료
- [ ] `boot/` 에서 `python boot.py` 실행 → `Success to boot.` 확인
- [ ] `object_detection_tracking/` 에서 `python app.py` 실행
- [ ] 영상 창에 초록색 검출 박스 표시 확인
- [ ] 터미널에 `track_id`, `confidence`, `box` 출력 확인
- [ ] `q` 키로 정상 종료
