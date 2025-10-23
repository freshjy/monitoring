# Monitoring Stack (Prometheus + Grafana + Alertmanager)

Docker Compose 기반 모니터링 스택

## 폴더 구조
```
monitoring/
├── docker-compose.yml        # Docker Compose 설정
├── prometheus/               # Prometheus 설정
│   ├── prometheus.yml        # 메인 설정 파일
│   ├── rules.yml             # 알림 규칙
│   └── targets/              # 모니터링 대상 정의
│       └── department.yml    # 부서별 타겟 설정
├── alertmanager/             # Alertmanager 설정
│   └── alertmanager.yml      # 알림 라우팅 설정
├── grafana-data/             # Grafana 데이터 (자동 생성)
└── prometheus-data/          # Prometheus 데이터 (자동 생성)
```

## 구성 요소
- **Prometheus** (9090) - 메트릭 수집 및 저장
- **Grafana** (3000) - 대시보드 및 시각화
- **Node Exporter** (9100) - 시스템 메트릭 수집
- **Alertmanager** (9093) - 알림 관리 및 라우팅

## 설정 파일

### docker-compose.yml
- 모든 서비스 정의 및 네트워크 설정
- 볼륨 마운트 및 포트 매핑

### prometheus/prometheus.yml
- 스크래핑 간격: 15초
- 평가 간격: 1분
- 부서별 job 분리 
- 파일 기반 서비스 디스커버리 사용

### prometheus/rules.yml
시스템 알림 규칙:
- **CPU 사용률**: 90% 이상 (2분간 지속)
- **메모리 사용률**: 90% 이상 (2분간 지속)
- **디스크 사용률**: 95% 이상 (즉시)

### alertmanager/alertmanager.yml
- Slack 웹훅 연동
- 알림 그룹화 및 반복 설정

## 사용법
```bash
# 스택 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f

# 스택 중지
docker-compose down

# 설정 리로드 (Prometheus)
curl -X POST http://localhost:9090/-/reload
```

## 접속 정보
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **Alertmanager**: http://localhost:9093
- **Node Exporter**: http://localhost:9100

## 모니터링 대상 추가
1. `prometheus/targets/` 디렉토리에 새 YAML 파일 생성
2. `prometheus/prometheus.yml`에 새 job 추가
3. Prometheus 설정 리로드

### 타겟 파일 예시 (department.yml)
```yaml
- labels:
    account: 'department name'
    instance: 'monitoring-server'
  targets:
    - 'server ip:9100'
```

## Grafana 대시보드 메트릭

### CPU 사용률
```promql
100 * (1 - avg by (instance, account) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
```

### 메모리 사용률
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

### 디스크 사용률
```promql
(1 - (node_filesystem_avail_bytes{mountpoint="/", fstype!="tmpfs"} / node_filesystem_size_bytes{mountpoint="/", fstype!="tmpfs"})) * 100
```

## 트러블슈팅
- 컨테이너 상태 확인: `docker-compose ps`
- 설정 파일 검증: `docker-compose config`
- Prometheus 타겟 상태: http://localhost:9090/targets
