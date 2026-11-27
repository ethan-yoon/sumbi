# 0단계: 맥북으로 AIS 선박 수신하기

목표: RTL-SDR 동글과 AIS-catcher로 주변 선박의 AIS 신호를 받아 웹 지도에 띄웁니다.
완료 기준: AIS-catcher 웹 화면에 선박이 한 척 이상 표시됩니다.

## 1. 살 것

| 품목 | 권장 | 메모 |
| --- | --- | --- |
| SDR 동글 | RTL-SDR Blog V4L (Lite) + 다이폴 안테나 키트 | 공식 스토어 기준 약 48달러. 기존 V4는 품절인 경우가 많고, V4L이 후속 모델입니다 |
| USB-C 어댑터 | USB-A(암) → USB-C(수) | 맥북에 USB-A 포트가 없기 때문입니다. USB-C 버전(V4Lc, 동글만)을 사면 필요 없습니다 |
| (선택) 연장 케이블 | 키트의 3m RG174 케이블 | 안테나를 창가나 실외에 두고 동글은 맥북 옆에 둡니다 |

- 공식 판매처: rtl-sdr.com 스토어, AliExpress "RTLSDRBlog Store", eBay "rtl-sdr-blog". 가품이 많으니 공식 판매처에서 사세요.
- 키트 구성: 23cm~1m 신축 안테나 2개, 5~13cm 신축 안테나 2개, 다이폴 베이스, 3m 케이블, 삼각대·흡착 마운트.

## 2. 안테나 세팅 (162MHz)

- AIS 주파수 162MHz의 1/4 파장은 약 46cm입니다. 긴 신축 안테나 두 개를 **각각 약 46cm**로 뽑습니다.
- AIS는 수직 편파라서 다이폴을 **세로(위아래)**로 세웁니다.
- 실내보다 창가, 창가보다 실외, 낮은 곳보다 높은 곳이 훨씬 잘 됩니다.

## 3. 소프트웨어 설치 (macOS, Apple Silicon)

```bash
# Homebrew가 없다면 먼저 설치
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 빌드 도구와 RTL-SDR 드라이버
brew update
brew install git make cmake pkg-config sqlite3 librtlsdr

# AIS-catcher 빌드
git clone https://github.com/jvde-github/AIS-catcher.git --depth 1
cd AIS-catcher
mkdir build && cd build
cmake ..
make
sudo make install
```

## 4. 동글 인식 확인

```bash
rtl_test -t      # 동글이 인식되는지 확인 (Ctrl+C로 종료)
AIS-catcher -l   # AIS-catcher가 보는 장치 목록
```

`rtl_test` 결과에 `PLL not locked` 오류가 나오거나 장치가 인식되지 않으면, Homebrew 드라이버 대신 RTL-SDR Blog 드라이버를 직접 빌드합니다.

```bash
brew uninstall librtlsdr
brew install libusb
git clone https://github.com/rtlsdrblog/rtl-sdr-blog.git
cd rtl-sdr-blog && mkdir build && cd build
cmake .. && make && sudo make install
```

드라이버를 바꿨다면 AIS-catcher를 다시 빌드합니다.

## 5. 실행

```bash
AIS-catcher -v 10 -N 8100
```

- `-v 10`: 10초마다 수신 통계를 출력합니다.
- `-N 8100`: 웹 화면을 켭니다. 브라우저에서 `http://localhost:8100`을 열면 지도에 선박이 표시됩니다.
- 터미널에 `!AIVDM,...`으로 시작하는 줄이 보이면 AIS 신호를 받고 있는 것입니다.

## 6. 어디서 해볼까

- 서울 내륙에서는 AIS 선박이 거의 잡히지 않습니다.
- 바다가 보이는 곳이 좋습니다. 인천(월미도, 송도, 영종도 해안)처럼 배가 많이 다니는 곳을 추천합니다.
- 맥북과 동글, 안테나만 들고 가면 됩니다. 노트북 배터리로 충분합니다.

## 7. 기록할 것

다음 단계 설계에 쓰기 위해 아래를 기록해 둡니다.

| 항목 | 예시 |
| --- | --- |
| 날짜, 장소 | 2026-12-05, 월미도 해안 |
| 안테나 위치와 높이 | 방파제 위, 지면에서 약 1.5m |
| 수신된 선박 수 | 10분 동안 12척 |
| 가장 먼 선박 거리 | 약 18km |
| 메모 | 바람 강함, 파도 1m |

결과는 `field-logs/` 폴더에 날짜별로 남깁니다.

## 문제 해결

| 증상 | 확인할 것 |
| --- | --- |
| 장치를 못 찾음 | USB 어댑터 연결, `rtl_test -t` 결과, 다른 SDR 프로그램이 동글을 점유하고 있지 않은지 |
| `PLL not locked` | 4번의 RTL-SDR Blog 드라이버로 교체 |
| 선박이 안 잡힘 | 장소(바다가 보이는지), 안테나 길이와 방향, 실외로 나가기 |
| 메시지는 오는데 지도에 없음 | 위치 정보가 없는 메시지일 수 있습니다. 몇 분 더 기다려 봅니다 |
