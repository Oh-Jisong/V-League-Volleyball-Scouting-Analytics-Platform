# V-League Volleyball Scouting & Analytics Platform
데이터 기반의 배구 경기 분석을 위해 설계된 웹 기반 스카우팅·퍼포먼스 분석 플랫폼입니다.

포지션별 기술 성공률, 팀·선수 비교, 라운드별 득점 흐름, OLAP 기반 집계, 랭킹 분석 등

코치/스카우터가 실제로 활용할 수 있는 기능들을 하나의 대시보드에서 제공합니다.

## 1. 프로젝트 개요

본 플랫폼은 다음 문제를 해결하기 위해 제작되었습니다:
(기존 사이트 : https://kovo.co.kr/KOVO/game/v-league?first=%ED%8C%80+%EC%88%9C%EC%9C%84 의 한계)
1. 포지션별로 서로 다른 기술 지표를 통합 분석하기 어려움

2. 팀/선수별 기술 성공률을 한 화면에서 비교하기 힘듦

3. 라운드 단위 경기 흐름 파악이 비효율적

4. 스카우팅 메모가 주관적이며 데이터화되지 않음

5. SQL 기반 고급 분석(윈도우 함수, ROLLUP)이 자동화되지 않음

→ 이를 모두 해결하는 웹 기반 종합 분석 플랫폼입니다.

## 2. 시스템 아키텍처 

* PHP Application — UI 렌더링, 세션 관리, SQL 처리
* MariaDB / MySQL — 팀/선수/게임/스탯/노트 저장
* Web Browser UI — 코치·스카우터용 인터페이스

## 3. 데이터베이스 구조

| 테이블명            | 설명         |
| --------------- | ---------- |
| team            | 구단 정보      |
| player          | 선수 기본 정보   |
| user            | 코치·스카우터 계정 |
| Att_Stats       | 공격수 스탯     |
| L_Stats         | 리베로 스탯     |
| S_Stats         | 세터 스탯      |
| Game            | 경기/라운드 정보  |
| Scouting_Report | 스카우팅 노트    |

## 4. 주요 기능

### 4.1 포지션별 성능 분석
포지션별 서로 다른 기술 지표를 자동 매핑하여 분석

* 공격수(Attacker) : `open_suc` , `backquick_suc` , `serve_suc`
* 세터(Setter) : `set_suc` , `dig_suc`
* 리베로(Libero) : `receive_good` , `dig_suc`

### 4.2 성공률 기반 랭킹 (Window Functions)
```
SELECT
    p.player_name,
    a.open_suc,
    RANK() OVER (ORDER BY a.open_suc DESC) AS rnk
FROM Att_Stats AS a
JOIN player AS p
    ON p.player_ID = a.player_ID;
```
**RANK() , ROW_NUMBER(), DENSE_RANK()** 을 활용해 자동 정렬

### 4.3 OLAP / ROLLUP 집계

```
SELECT
    t.team_Name,
    p.position_Name,
    SUM(a.fail_count) AS total_fail
FROM Att_Stats AS a
JOIN player AS p ON a.player_ID = p.player_ID
JOIN team AS t ON p.team_ID = t.team_ID
GROUP BY t.team_Name, p.position_Name WITH ROLLUP;
```
**팀 합계, 포지션 합계, 상세 레벨, 전체 합계** 모두 한 번의 SQL로 처리

### 4.4 Drill-Down 분석

**팀 → 선수 → 기술 → 라운드** 까지 단계적 분석 가능

### 4.5 스카우팅 노트 기능

* 계정별 개인 노트 저장
* 팀 전체 공유 노트 구분
* CRUD 지원
* 세션 기반 권한 분리

## 5. 프로젝트 구조
```
v_league/
 ├── dashboard.php            # 대시보드
 ├── login.php / logout.php   # 로그인/로그아웃
 ├── player_select.php        # 팀/포지션 선택
 ├── player_profile.php       # 선수 상세 페이지
 ├── aggregate.php            # Fail 합산 비교
 ├── analysis_value.php       # 성공/실패율 분석
 ├── score_accumulation.php   # 라운드별 득점
 ├── drilldown.php            # 계층적 분석
 ├── rollup.php               # OLAP ROLLUP 집계
 ├── mynotes.php              # 스카우팅 노트
 ├── db.php                   # DB 연결
 ├── dbcreate.sql             # DB 생성 스크립트
 ├── dbinsert.sql             # 기본 데이터
 ├── dbdrop.sql               # 테이블 삭제
 └── dbdump.sql               # 예시 데이터 덤프
```

## 6. 설치 및 실행

### 6.1 클론
```
git clone https://github.com/yourname/v_league.git
```

### 6.2 DB 생성 (phpMyAdmin)
1. `dbcreate.sql`
2. `dbinsert.sql`
순서로 import

### 6.3 DB 여결 수정
`db.php` 수정 : 
```
$pdo = new PDO(
  "mysql:host=localhost;
        dbname='';
        charset=utf8",
  "",
  ""
);
```

### 6.4 실행
```
http://localhost/v_league/login.php
```
