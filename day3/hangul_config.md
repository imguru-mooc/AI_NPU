# 라즈베리파이 4 브라우저 한글 깨짐 조치 가이드

라즈베리파이 4 브라우저에서 한글이 깨지는 것은 대부분 한글 폰트가 설치되지 않아서입니다. 아래 순서로 조치하면 됩니다.

## 1. 한글 폰트 설치 (핵심)

```bash
sudo apt update
sudo apt install fonts-nanum fonts-nanum-coding fonts-nanum-extra -y
```

또는 Noto 폰트를 선호하면:

```bash
sudo apt install fonts-noto-cjk -y
```

## 2. 폰트 캐시 갱신

```bash
fc-cache -fv
```

## 3. 로케일 설정 (입력/시스템 메시지까지 깨질 경우)

```bash
sudo raspi-config
```

→ `Localisation Options` → `Locale` → `ko_KR.UTF-8 UTF-8` 선택 (스페이스바로 체크) → 기본 로케일도 `ko_KR.UTF-8`로 지정 → 재부팅

또는 명령어로:

```bash
sudo apt install locales -y
sudo sed -i 's/# ko_KR.UTF-8/ko_KR.UTF-8/' /etc/locale.gen
sudo locale-gen
sudo update-locale LANG=ko_KR.UTF-8
sudo reboot
```

## 4. 브라우저 재시작 후 확인

Chromium을 완전히 종료하고 다시 실행하면 한글이 정상 표시됩니다. 폰트 설치만으로 해결되는 경우가 대부분이라 1~2번만 해도 브라우저 깨짐은 잡힙니다.

## 추가: 한글 입력 설정

한글 **입력**까지 필요하면 아래 명령어로 입력기를 설치한 후 재부팅하면 됩니다.

```bash
sudo apt install ibus ibus-hangul -y
```
