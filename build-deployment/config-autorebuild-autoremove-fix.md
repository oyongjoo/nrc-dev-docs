# CONFIG_AUTOREBUILD/AUTOREMOVE 설정 변경 문제 해결 가이드

## 문제 상황

빌드 중 `CONFIG_AUTOREBUILD=y`와 `CONFIG_AUTOREMOVE=y` 설정이 예기치 않게 활성화되어 빌드 성능에 영향을 미치는 문제가 발생했습니다.

## 원인 분석

### 1. 근본 원인
**`prepare_halow/.config` 파일에 잘못된 기본값 설정:**
```bash
CONFIG_AUTOREBUILD=y
CONFIG_AUTOREMOVE=y
```

### 2. 설정 전파 과정
1. `prepare_halow.sh` 실행 시 → `prepare_halow/.config` 복사
2. `.config` 파일에 자동으로 `CONFIG_AUTOREBUILD=y` 설정됨
3. 이후 사용자가 `# CONFIG_AUTOREBUILD is not set`를 추가해도 중복 설정 존재
4. 빌드 시스템이 `=y` 설정을 우선 적용

### 3. 확인 방법
```bash
# 현재 설정 확인
grep -E "CONFIG_AUTOREBUILD|CONFIG_AUTOREMOVE" .config

# 중복 설정이 있는 경우 출력 예시:
CONFIG_AUTOREBUILD=y
CONFIG_AUTOREMOVE=y
# CONFIG_AUTOREBUILD is not set
# CONFIG_AUTOREMOVE is not set
```

## 해결 방법

### 1. 즉시 수정 (현재 빌드용)
```bash
# 잘못된 설정 제거
sed -i '/^CONFIG_AUTOREBUILD=y/d' .config
sed -i '/^CONFIG_AUTOREMOVE=y/d' .config

# 올바른 설정 확인
grep -E "CONFIG_AUTOREBUILD|CONFIG_AUTOREMOVE" .config
# 출력: # CONFIG_AUTOREBUILD is not set
#       # CONFIG_AUTOREMOVE is not set
```

### 2. 근본 원인 수정 (향후 빌드용)
```bash
# prepare_halow/.config 파일 수정
sed -i 's/CONFIG_AUTOREBUILD=y/# CONFIG_AUTOREBUILD is not set/' prepare_halow/.config
sed -i 's/CONFIG_AUTOREMOVE=y/# CONFIG_AUTOREMOVE is not set/' prepare_halow/.config
```

### 3. 자동 검증 스크립트 추가
`nrc-protection/verify-config.sh`:
```bash
#!/bin/bash
# 빌드 전 설정 검증 스크립트

CONFIG_FILE="$1"
if [ -z "$CONFIG_FILE" ]; then
    CONFIG_FILE=".config"
fi

echo "🔍 CONFIG 설정 검증 중..."

# AUTOREBUILD/AUTOREMOVE 중복 확인
auto_rebuild=$(grep -c "CONFIG_AUTOREBUILD" "$CONFIG_FILE")
auto_remove=$(grep -c "CONFIG_AUTOREMOVE" "$CONFIG_FILE")

if [ "$auto_rebuild" -gt 1 ] || [ "$auto_remove" -gt 1 ]; then
    echo "❌ AUTOREBUILD/AUTOREMOVE 중복 설정 발견"
    echo "   AUTOREBUILD 항목: $auto_rebuild개"
    echo "   AUTOREMOVE 항목: $auto_remove개"
    
    # 자동 수정
    echo "🔧 자동 수정 중..."
    sed -i '/^CONFIG_AUTOREBUILD=y/d' "$CONFIG_FILE"
    sed -i '/^CONFIG_AUTOREMOVE=y/d' "$CONFIG_FILE"
    
    echo "✅ 중복 설정 제거 완료"
fi

# 최종 설정 확인
echo "📋 최종 AUTOREBUILD/AUTOREMOVE 설정:"
grep -E "CONFIG_AUTOREBUILD|CONFIG_AUTOREMOVE" "$CONFIG_FILE"
```

## 예방 조치

### 1. prepare_halow.sh 개선
`prepare_halow.sh` 스크립트 끝에 추가:
```bash
# CONFIG_AUTOREBUILD/AUTOREMOVE 강제 비활성화
sed -i '/^CONFIG_AUTOREBUILD=/d' .config
sed -i '/^CONFIG_AUTOREMOVE=/d' .config
echo '# CONFIG_AUTOREBUILD is not set' >> .config
echo '# CONFIG_AUTOREMOVE is not set' >> .config

echo "✅ AUTOREBUILD/AUTOREMOVE 비활성화 완료"
```

### 2. safe-nrc-build.sh에 검증 단계 추가
빌드 전 자동 검증:
```bash
# CONFIG 설정 검증
echo "🔍 CONFIG 설정 검증 중..."
/home/builder/verify-config.sh
```

### 3. 정기적 모니터링
빌드 스크립트에 설정 모니터링 추가:
```bash
# 빌드 전후 설정 비교
grep -E "CONFIG_AUTOREBUILD|CONFIG_AUTOREMOVE" .config > /tmp/config_before.txt
# ... 빌드 실행 ...
grep -E "CONFIG_AUTOREBUILD|CONFIG_AUTOREMOVE" .config > /tmp/config_after.txt

if ! cmp -s /tmp/config_before.txt /tmp/config_after.txt; then
    echo "⚠️ 빌드 중 CONFIG 설정 변경 감지"
    echo "변경 전:" && cat /tmp/config_before.txt
    echo "변경 후:" && cat /tmp/config_after.txt
fi
```

## 기술적 설명

### CONFIG_AUTOREBUILD/AUTOREMOVE의 역할
- **CONFIG_AUTOREBUILD**: 의존성 변경 시 자동 재빌드
- **CONFIG_AUTOREMOVE**: 사용하지 않는 패키지 자동 제거

### 성능에 미치는 영향
- **활성화 시**: 빌드 시간 증가, 불필요한 재컴파일
- **비활성화 시**: 빠른 빌드, 수동 의존성 관리 필요

### HaLow 개발에서의 권장 설정
```bash
# 개발 단계에서는 비활성화 권장
# CONFIG_AUTOREBUILD is not set
# CONFIG_AUTOREMOVE is not set
```

## 트러블슈팅

### 문제: 설정을 수정했는데도 계속 활성화됨
**해결**: 설정 중복 확인 및 제거
```bash
grep -n "CONFIG_AUTOREBUILD" .config
# 여러 줄이 나오면 중복 설정 존재
```

### 문제: prepare_halow.sh 실행 후 설정이 되돌아감
**해결**: `prepare_halow/.config` 원본 파일 수정

### 문제: 빌드 중 설정이 자동으로 변경됨
**해결**: make 명령어 옵션 확인 및 .config 파일 권한 설정

## 검증 방법

### 1. 설정 확인
```bash
# 올바른 설정 (OK)
# CONFIG_AUTOREBUILD is not set
# CONFIG_AUTOREMOVE is not set

# 잘못된 설정 (NG)
CONFIG_AUTOREBUILD=y
CONFIG_AUTOREMOVE=y
```

### 2. 빌드 성능 비교
- 활성화 시: 초기 빌드 후 약 30-40% 더 오래 걸림
- 비활성화 시: 빠른 빌드, 개발 효율성 향상

---

**작성일**: 2025-07-28  
**버전**: 1.0  
**최종 업데이트**: CONFIG_AUTOREBUILD/AUTOREMOVE 설정 문제 완전 해결