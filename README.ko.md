# mytune — MySQL 실시간 모니터 (무료)

![.NET 11](https://img.shields.io/badge/.NET-11.0-512BD4)
![C# 15](https://img.shields.io/badge/C%23-15-239120)
![Blazor Hybrid](https://img.shields.io/badge/Blazor-Hybrid-5C2D91)
![MySQL 8.0+](https://img.shields.io/badge/MySQL-8.0%2B-4479A1)
![Windows x64](https://img.shields.io/badge/Windows-x64-0078D6)

**[최신 배포본 내려받기](https://github.com/Doni-Kim/mytune-release/releases/latest)** ·
[English](README.md) · [매뉴얼 (HTML)](mytune.html)

MySQL 상태를 실시간으로 보는 데스크톱 모니터입니다. 창 하나 · 실행 파일 하나이고, 서버에는 아무것도 설치하지 않습니다 —
`performance_schema` · `information_schema` · `sys` 만 읽습니다. 부담 없이 쓰시라고 공유드립니다.

## 화면

| 실시간 대시보드 | Top SQL |
|---|---|
| ![대시보드](screenshots/dashboard.png) | ![Top SQL](screenshots/top-sql.png) |

| Lock Chain | 세션 상세 |
|---|---|
| ![Lock Chain](screenshots/locks.png) | ![세션 상세](screenshots/session-detail.png) |

| History | 알림 |
|---|---|
| ![History](screenshots/history.png) | ![알림](screenshots/alerts.png) |

![어두운 테마](screenshots/dashboard-dark.png)

화면은 촬영용으로 잠깐 띄운 서버의 가상 `shop` 스키마입니다.

## 설치·설정

- 압축을 풀고 폴더째 둔 뒤 `mytune.exe` 를 실행하면 됩니다 (Single File Publishing).
- .NET 설치 불필요 — 런타임이 실행 파일에 포함되어 있습니다.
- 설정은 같은 폴더의 접속 파일입니다. zip 에 `mytuneNode1.json` · `mytuneNode2.json` 두 개가 들어 있습니다 — 서버 하나에 파일 하나.
  서버가 하나면 한 파일만 두고 나머지는 지우세요. 파일을 고쳐도 되고(아래 예시), 그냥 실행해도 됩니다 —
  들어 있는 값으로 붙지 못하면 그 값이 채워진 접속 창이 뜹니다. 실제로 접속된 뒤에 입력한 값을 저장합니다(비밀번호는 암호화해서 저장).

## 주요 기능

- **실시간 대시보드**: 버퍼 풀 적중률 · 접속 수(게이지), QPS · TPS · Running · Waiting(추이 그래프),
  행 락 대기 · 데드락 · 롤백 · 슬로 쿼리 · 읽고 고친 행 · 디스크 읽기/쓰기 · 네트워크 받기/보내기 · redo · history list, 대기 이벤트, 세션 표. 한 지표는 한 곳에만 나옵니다.
  서버의 누적 카운터는 늘 직전 주기와의 차이로 보여 줍니다.
- **진짜 blocker 를 짚는 Lock Chain**: 행 락은 `data_lock_waits` 로, 메타데이터 락은 서버의 락 호환성 규칙으로 판정해 실제로 충돌하는 세션만 blocker 로 냅니다.
  `F5` 는 막는 세션과 막힌 세션을 함께 보여 줍니다.
- **세션 상세**(`Enter`): 문장 전문 · 실행 계획 · 쥔 락 · 접속 속성. `Ctrl+K` 로 쿼리만 또는 접속째 KILL, `Ctrl+X` 로 Excel 저장(시트 넷: 세션 · SQL · 계획 · Objects).
- **Object Info**: 세션 상세와 Top SQL(줄을 누르면)의 `[Object Info]` 가 계획이 읽는 테이블마다 크기 · 컬럼(히스토그램) · 인덱스(이 계획이 쓴 것은 강조, 카디널리티 · 읽기 수) · 파티션을 보입니다.
  계획의 조건에 나온 컬럼에 표시하고, 형 · 콜레이션 변환으로 인덱스를 못 쓴 컬럼(경고 1739)을 ⚠ 로 짚습니다. MySQL 8.3 이상.
- **Find SQL**(`Ctrl+F`): 문장 digest 를 넣으면 Top SQL 을 거치지 않고 그 문장의 통계 · 실행 계획 · Object Info 를 엽니다.
- **팝업**: Server(`I`) · Connections(`C`) · Locks(`A`) · InnoDB(`V`) · Replication(`W`) · Top SQL(`T`, 델타 · 검색 · 머리글 정렬) · 인덱스 진단(`X`) · Disk(`D`).
- **임계값 알림** 12종: 접속 포화 · 대기 세션 · idle in transaction · 긴 문장 · 버퍼 풀 적중률 · 롤백 비율 · 디스크 임시 테이블 · Lock Chain · 데드락 · history list · 복제 지연 · 복제 스레드 중단.
- **History**: `L` 로 로컬 SQLite 에 모니터링 데이터를 쌓고, `H` 로 지난 흐름을 되짚습니다.
  - 구간은 1시간 / 6시간 / 24시간 / 1주 / 1개월 / 전체, 지표는 17가지입니다.
  - 오래된 기록은 자동으로 지웁니다(기본 지표 30일 · 세션 7일, `O` 설정 창에서 조정).
- Excel 저장: ClosedXML 기반이라 Excel 이 없어도 xlsx 파일이 저장됩니다.
- 접속이 끊기면 스스로 다시 붙습니다.
- **2.2 에서 더한 것**: Error log 팝업(`E`, `performance_schema.error_log`, MySQL 8.0.22 부터) · Server 의 Changed settings 와 Memory in use · Connections 의 Load by user · Index 의 Hot tables · Top SQL 의 P95 / P99.
- **막힘 트리**: Locks(`A`)가 누가 누구를 막는지 트리로 그리고, Excel 에도 사슬 전체가 실립니다.
- **찾기**: `/` 로 세션을 글자로 거릅니다.
- **알림이 켜지는 순간**: 막힘 트리 · 문장 · 데드락 내용을 `captures\` 아래 파일로 남기고, 위험 알림은 창이 앞에 없을 때 작업 표시줄 깜빡임 · Windows 알림으로 알려 줍니다.
- History 에서 한 시점을 누르면 그때 기록된 세션이 나옵니다.
- `sslMode` 에 `verify-ca` · `verify-full`, CA 파일은 `sslCa` — 인증서가 거부되면 접속 창에 사유와 고칠 곳이 나옵니다. Hot tables 는 50줄 · 전체 대비 비율 · `Δ delta`.
- **Admin Reference**: `F1` 의 두 번째 탭에서 서버가 가진 설명을 글자를 칠 때마다 찾습니다 — `sys` 스키마 루틴(설명 · 매개변수 · 예제)과
  `HELP` 가 읽는 서버 도움말 표(명령문 · 함수 700개, 문서 주소 포함). 접속한 서버의 것이라 버전이 저절로 맞고 인터넷이 필요 없습니다.
  자주 쓰는 관리 작업 54개에는 복사해 쓰는 샘플이 있습니다 — mytune 은 관리 명령을 실행하지 않습니다.
- **서버마다 설정 파일 하나**: exe 옆에 둘 이상이면 시작할 때 어느 것으로 붙을지 고르는 창이 뜹니다.
  손으로 고치다 깨진 파일은 목록에 빨갛게, 몇째 줄 몇째 글자가 틀렸는지와 함께 보입니다.
- **설정 창**: `O` 로 수집 주기(3~60초, 기본 5초) · 로그 보관 · Top SQL · Excel · 알림 임계값 · `L` 로깅이 남길 세션을 화면에서 고칩니다.
  값을 검사한 뒤 지금 쓰는 설정 파일에 저장하고 바로 적용합니다. 접속 정보는 시작 접속 창에서만 바꿉니다.
- **테마 12종**: 밝은 6 · 어두운 6, 기본은 GitHub Light. 상단 바에서 고릅니다.
- **시작할 때 서버가 응답하지 않으면**: 20초 남짓 빈 화면 대신, 누구에게 몇 초째 붙는 중인지 보이는 작은 창이 뜨고 Cancel 로 접속 정보를 고칠 수 있습니다.
- `F1` 을 누르면 단축키 도움말이, 한 번 더 누르면 Admin Reference 탭이 나옵니다.

자세한 사용법은 첨부한 `mytune.html`(스크린샷이 든 매뉴얼)을 참고해 주세요. 사용상 제한 없습니다.

## 지원 범위 · 제약사항

- UI 가 Web 기반(Blazor Hybrid)이라 **Windows 전용**입니다. Linux·macOS 에서는 단독 실행되지 않습니다.
- **MySQL 8.0 이상** + `performance_schema=ON`. 8.0.46 과 최신판에서 확인했습니다.
  `performance_schema` 구성이 같은 MySQL 호환 배포판(Percona Server 8.0+ 등)도 같은 방식으로 동작합니다.
- **MariaDB 는 지원하지 않습니다** — `performance_schema` 에 `data_locks` · `data_lock_waits` 가 없습니다. 접속하면 접속 창에서 알려 줍니다.
- 코드는 ConfuserEx 로 난독화(무료 툴이라 강력한 수준은 아닙니다).

## 모니터링 전용 계정

```sql
CREATE USER 'mytune'@'%' IDENTIFIED BY '...';
GRANT PROCESS ON *.* TO 'mytune'@'%';                  -- 다른 사용자의 세션, innodb_metrics
GRANT SELECT ON performance_schema.* TO 'mytune'@'%';
GRANT SELECT ON sys.* TO 'mytune'@'%';                 -- 인덱스 진단
GRANT REPLICATION CLIENT ON *.* TO 'mytune'@'%';       -- 선택: Replication 팝업 · 복제 알림
GRANT CONNECTION_ADMIN ON *.* TO 'mytune'@'%';         -- 선택: 남의 세션에 Ctrl+K
```

이런 계정에서는 세션 상세의 실행 계획이 *추정 계획*으로 나옵니다(화면에 그렇게 적힙니다). 다른 사용자가 돌리는 문장의 실제 계획은 root 같은 관리자 계정에서만 볼 수 있습니다.

Disk(`D`) · 인덱스 진단(`X`)은 계정이 권한을 가진 스키마만 보입니다 — `information_schema` 가 나머지를 오류 없이 빼기 때문입니다. mytune 이 가려진 스키마 수를 알려 주니, 보려면 그 스키마에 `SELECT` 를 주면 됩니다.

## 화면이 안 뜬다면 (WebView2)

창은 뜨는데 내용이 백지라면 WebView2 런타임이 없는 경우입니다.

- **Windows 11**: OS 에 기본 내장이라 항상 있습니다.
- **Windows 10**: 2021년 이후 Windows Update 로 대부분 자동 배포됐지만, 업데이트를 오래 안 한 PC 나 LTSC 같은 특수 에디션에는 없을 수 있습니다.
- **Windows Server (2016/2019/2022)**: 기본 미포함인 경우가 많아 별도 설치가 필요할 수 있습니다.

없으면 Microsoft 의 "에버그린 독립형 설치 프로그램"(`MicrosoftEdgeWebView2RuntimeInstallerX64.exe`)을 설치하시면 됩니다.
→ https://developer.microsoft.com/microsoft-edge/webview2/

## 문제가 생기면

오류가 나면 실행 파일 옆에 `mytune.log` 가 생깁니다(평소에는 만들지 않습니다).

- **버그 · 질문**: [Issues](https://github.com/Doni-Kim/mytune-release/issues) 에 남겨 주세요.
  로그 파일은 거기에 올리지 마세요 — 비밀번호는 없지만 서버 주소와 SQL 문장이 들어 있을 수 있습니다.
- **로그 파일**이나 공개하기 어려운 내용은 메일로 보내 주세요: **doniikim@gmail.com**

## 기술 스택

- .NET 11.0 (x64), C# 15, Blazor Hybrid
- 패키지: MySqlConnector · Microsoft.Data.Sqlite · ClosedXML · Microsoft.Web.WebView2 · Microsoft.AspNetCore.Components.WebView.WindowsForms
- 실행 파일에 묶인 위 오픈소스들의 저작권 고지 · 라이선스 전문: [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) (zip 안에도 들어 있습니다)

## 접속 파일 예시 (`mytuneNode1.json` …)

zip 에는 작은 틀 두 개 — `mytuneNode1.json`(이 PC, `localhost:3306` · `root`) · `mytuneNode2.json`(다른 서버, 모니터링 계정 `mytune`)이 들어 있습니다 —
서버마다 파일 하나, 이름은 자유입니다(`prod.json` · `dev.json` …). `mytune.exe` 옆에 둘 이상이면 시작할 때 고르는 창이 뜨고
(목록에는 `user@server:port/schema` 만 — 비밀번호는 보이지 않습니다), 하나뿐이면 바로 붙습니다.
고른 파일이 그 실행의 설정이 되어 암호화된 비밀번호 · 창 위치 · 테마가 그 파일에 저장됩니다.
같은 폴더의 mytune 은 한 번에 하나만 뜨니, 여러 서버를 동시에 보려면 폴더를 나누세요. 파일을 고쳐서 쓰거나, 접속 창에 맡기면 됩니다:

```json
{
  "databases": [
    {
      "userId": "root",
      "password": "change-me",
      "server": "127.0.0.1",
      "port": "3306",
      "database": "",
      "sslMode": "preferred"
    }
  ],
  "interval": 5
}
```

- `password` 는 평문으로 적으면 첫 실행 때 자동 암호화됩니다.
- `database` 는 비워도 됩니다 — mytune 은 서버 전체를 봅니다.
- `sslMode`: `none` / `preferred` / `required` / `verify-ca` / `verify-full`. `sslCa`: 검증 모드가 믿을 CA 파일(.pem) — 비우면 Windows 인증서 저장소. 모르는 값이면 접속하지 않고 알립니다.
- `interval` 은 수집 주기(초)입니다. 3~60, 적지 않으면 5.
- `alerts`, `topSql`, `logRetention`, `logFilter`(`L` 로깅이 남길 세션) 같은 절은 적지 않아도 됩니다. 프로그램을 닫을 때 기본값으로 채워 넣어 주고, `O` 로 화면에서 고칠 수 있습니다.
  zip 안의 `mytune_sample_kr.json` 에 모든 설정의 설명이 있습니다.

## 사용 조건

업무든 개인이든 자유롭게 쓰셔도 됩니다. 실행 파일 재배포와 리버스 엔지니어링은 삼가 주세요.
소스는 공개하지 않습니다.

## 연락처

DBMS Works — **doniikim@gmail.com**

Oracle → PostgreSQL / MySQL 마이그레이션, DB 성능 튜닝 문의도 받습니다.
