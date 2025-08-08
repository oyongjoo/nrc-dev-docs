# OpenWrt HaLow 빌드 트러블슈팅 가이드

## 개요
OpenWrt HaLow 빌드 과정에서 발생할 수 있는 문제들과 해결 방법을 정리한 문서입니다.

## 알려진 빌드 오류 및 해결방법

### 1. staging_dir include 디렉토리 누락 오류

#### 증상
```
cc1: error: /home/builder/workspace/halow-main/staging_dir/target-mipsel_24kc_musl/usr/include: No such file or directory [-Werror=missing-include-dirs]
ERROR: target/linux failed to build.
```

#### 원인
- toolchain이 제대로 빌드되지 않아 target staging 디렉토리에 include 파일들이 생성되지 않음
- target/linux 컴파일 시 필요한 헤더 파일 경로를 찾을 수 없음

#### 해결방법
1. target/linux 클린업
   ```bash
   cd /home/builder/workspace/halow-main
   make target/linux/clean
   ```

2. toolchain 재설치
   ```bash
   make toolchain/install -j$(nproc)
   ```

3. staging 디렉토리 구조 확인
   ```bash
   ls -la staging_dir/target-mipsel_24kc_musl/usr/
   # include 디렉토리가 존재하는지 확인
   ```

4. 빌드 재시작
   ```bash
   make -j$(nproc) V=s
   ```

#### 예방방법
- 빌드 시작 전 `make menuconfig`로 설정 확인
- 의존성 순서대로 단계별 빌드 수행
- 중간에 빌드가 중단된 경우 전체 클린 후 재시작

### 2. 시계 동기화 경고

#### 증상
```
make[3]: Warning: File has modification time 1.3 s in the future
make[3]: warning: Clock skew detected. Your build may be incomplete.
```

#### 해결방법
컨테이너 시간 동기화:
```bash
docker exec openwrt-halow-builder date
# 필요시 컨테이너 재시작
docker-compose restart
```

## 일반적인 빌드 절차

### 정상 빌드 단계
1. **환경 설정**
   ```bash
   cd /home/builder/workspace/halow-main
   ./setup_halow_build.sh
   ```

2. **메뉴 설정**
   ```bash
   make menuconfig
   ```

3. **의존성 다운로드**
   ```bash
   make download
   ```

4. **단계별 빌드**
   ```bash
   make tools/install -j$(nproc)
   make toolchain/install -j$(nproc)
   make target/linux/prepare
   make -j$(nproc) V=s
   ```

### 빌드 로그 모니터링
```bash
# 백그라운드 빌드 시작
nohup make -j$(nproc) V=s > full_build.log 2>&1 &

# 실시간 로그 확인
tail -f full_build.log

# 에러 검색
grep -i "error\|failed" full_build.log
```

## 빌드 환경 검증

### 필수 확인사항
1. **디스크 공간** (최소 50GB 권장)
   ```bash
   df -h
   ```

2. **메모리 사용량**
   ```bash
   free -h
   ```

3. **컨테이너 상태**
   ```bash
   docker-compose ps
   ```

### 환경 초기화
전체 빌드 환경 리셋이 필요한 경우:
```bash
# 컨테이너 정리
docker-compose down
docker-compose up -d --build

# 빌드 디렉토리 정리
make clean
make distclean
```

## 성능 최적화

### 빌드 시간 단축
- CPU 코어 수에 맞춘 병렬 빌드: `-j$(nproc)`
- ccache 활성화 (menuconfig에서 설정)
- 필요한 패키지만 선택적 빌드

### 디버깅 옵션
- 상세 로그: `V=s` 또는 `V=99`
- 단일 스레드 빌드: `-j1` (디버깅 시)

## NRC 커널 모듈 (nrc.ko) 빌드 문제

### 3. nrc.ko 컴파일 실패 - 빌드 시스템이 nrc 디렉토리에 진입하지 않음

#### 증상
```bash
# 빌드는 성공하지만 nrc.ko 파일이 생성되지 않음
make package/kernel/mac80211/compile V=s  # 성공
find build_dir -name "nrc.ko"             # 결과 없음
```

#### 원인 분석
- 설정은 정상: `CPTCFG_NRC=m`, `CPTCFG_WLAN_VENDOR_NEWRACOM=y`
- 소스 파일들도 정상적으로 존재
- 하지만 빌드 시스템이 nrc 디렉토리에서 컴파일을 수행하지 않음
- Makefile 체인에서 silent failure 발생

#### 해결방법
**nrc-smart-build.sh** 스크립트 사용 (권장):

1. **Smart Build 모드** (.built 파일 삭제로 강제 재빌드):
   ```bash
   /home/builder/nrc-smart-build.sh smart
   ```

2. **Force Build 모드** (object 파일 전체 삭제):
   ```bash
   /home/builder/nrc-smart-build.sh force
   ```

3. **상태 확인**:
   ```bash
   /home/builder/nrc-smart-build.sh status
   ```

#### nrc-smart-build.sh 스크립트 특징
- **.built 파일 관리**: OpenWrt 빌드 시스템의 캐시 파일을 제어
- **백그라운드 빌드**: 타임아웃 방지를 위한 nohup 사용
- **실시간 모니터링**: 빌드 로그와 에러 추적 기능
- **NRC 수정사항 검증**: LED 기능 및 변수 수정 확인

### 4. C90 표준 준수 오류

#### 증상
```bash
error: ISO C90 forbids mixed declarations and code [-Werror=declaration-after-statement]
1491 |                 char hex_str[256] = {0};
     |                 ^~~~
```

#### 원인
- 커널 모듈은 C90 표준을 준수해야 함
- 변수 선언이 코드 중간에 위치하면 컴파일 오류

#### 해결방법
1. **변수 선언을 블록 시작 부분으로 이동**:
   ```c
   // 잘못된 예
   if (cmd) {
       pr_err("Processing...");
       char hex_str[256] = {0};  // 오류!
       int pos = 0;
   }
   
   // 올바른 예
   if (cmd) {
       char hex_str[256] = {0};  // 블록 시작
       int pos = 0;
       int i;                    // for 루프 변수도 미리 선언
       
       pr_err("Processing...");
       for (i = 0; i < len; i++) {
           // 처리 로직
       }
   }
   ```

2. **for 루프 변수 사전 선언**:
   ```c
   // C90 표준 준수
   int i;
   for (i = 0; i < len; i++) {
       // 루프 내용
   }
   ```

### 5. nrc.ko와 nrc_rtcm_ie 패키지 빌드 분리

#### 빌드 대상 구분
- **nrc.ko (커널 모듈)**: `make package/kernel/mac80211/compile`
- **nrc_rtcm_ie (유저스페이스)**: `make package/network/utils/nrc_rtcm_ie/compile`

#### 전용 빌드 스크립트 사용
1. **커널 모듈**: `/home/builder/nrc-smart-build.sh`
2. **유저스페이스 앱**: `/home/builder/nrc-rtcm-build.sh`

#### 수정사항 적용 후 빌드 순서
```bash
# 1. 커널 모듈 수정 후 (nrc-netlink.c 등)
/home/builder/nrc-smart-build.sh smart

# 2. 유저스페이스 앱 수정 후 (nrc_rtcm_vendor_ie.c 등)  
/home/builder/nrc-rtcm-build.sh build

# 3. 최종 상태 확인
/home/builder/nrc-smart-build.sh status
/home/builder/nrc-rtcm-build.sh status
```

### NRC 빌드 성공 검증 방법

#### 1. nrc.ko 파일 존재 확인
```bash
find build_dir -name "nrc.ko" -exec ls -lh {} \;
# 예상 결과: 285K 크기의 nrc.ko 파일
```

#### 2. LED 기능 포함 여부 확인
```bash
strings path/to/nrc.ko | grep -i "liam LED command"
# LED 기능이 포함되어 있으면 문자열 발견
```

#### 3. 빌드 로그에서 NRC 컴파일 확인
```bash
grep -E 'CC.*newracom|LD.*nrc' build_log.txt
# 예상 결과:
# CC [M] .../nrc-netlink.o
# LD [M] .../nrc.o  
# LD [M] .../nrc.ko
```

## 문의 및 지원

빌드 관련 추가 문제가 발생하면:
1. 빌드 로그 전체 보관
2. 에러 메시지와 발생 단계 기록  
3. 환경 정보 수집 (Docker 버전, 호스트 OS 등)

---
*최종 업데이트: 2025-07-29 - NRC 커널 모듈 빌드 및 LED 시스템 구현 추가*