# Mission6

## 이벤트 히스토리 설계하기   

ACME 서비스의 화면 전환 및 사용자 행동 이력 기록을 위한 데이터 구조 및 타입 설계를 설명합니다.  
이 데이터는 사용자의 화면별 사용 이력을 기반으로 유입, 전환, 사용 행동 분석, A/B 테스트, 서비스 개선 및 통계 집계 등에 활용됩니다.    

---  


### 핵심 요구 동작별 추출  

- 로그인 화면 진입 여부 기록  
- 메인 화면 진입 및 이벤트(광고 랜덤 표시) 여부 기록  
- 전환 vs 푸시 구분 필요  
- 상세 화면에서 스크롤 횟수/페이지/입력 값/선택 값 기록  
- 화면별 FROM → TO 전환 경로 기록  
- 시간(timestamp) 기록  
- 사용자 ID 식별 필요  


---  

## 1️. 설계 방식을 이 방식으로 선택한 이유   
- 유연한 JSON 기반 기록으로 스크롤 수, 선택 값, 광고 ID 등 상황별 다양한 데이터 적재 가능  
- ENUM으로 ACTION 구분하여 집계/통계 시 GROUP BY/COUNT 쉽게 가능  
- 화면 간 전환 분석 가능 (from_screen, to_screen)  
- 시간/사용자 기준 집계 가능하여 분석에 용이   
- 신규 기능 추가 시 metadata 확장만으로 기록 유지 가능 (확장성)  

---

## 2. 데이터 구조

### 컬렉션명 (또는 테이블명)  
### 필드 및 타입  

| 필드명 | 타입 | 설명 |  
|--------|------|------|  
| `id` | UUID | PK (히스토리 고유 ID) |  
| `user_id` | STRING | 사용자 ID |  
| `timestamp` | DATETIME (ISO 8601, UTC) | 이벤트 발생 시각 |  
| `screen_name` | STRING | 화면 이름 (`LOGIN`, `MAIN`, `MENU_1_DETAIL`, `AD_MODAL` 등) |  
| `action` | ENUM | `ENTER`, `EXIT`, `PUSH`, `POP`, `SELECT`, `SAVE`, `SCROLL`, `EVENT_SHOW` |  
| `from_screen` | STRING (nullable) | 전환 시 FROM 화면 이름 |  
| `to_screen` | STRING (nullable) | 전환 시 TO 화면 이름 |  
| `metadata` | JSON (nullable) | 이벤트별 상세 데이터 |  
| `session_id` | STRING (nullable) | 사용자 세션 구분용 |  

---  

## 3️. `action` ENUM 값 정의  

- `ENTER` : 화면 진입  
- `EXIT` : 화면 종료  
- `PUSH` : 스택 푸시 방식 화면 진입  
- `POP` : 스택 팝 방식 화면 종료  
- `SELECT` : ON/OFF, 메뉴 선택 등 옵션 선택  
- `SAVE` : 값 저장 이벤트  
- `EVENT_SHOW` : 이벤트 광고 랜덤 표시  
- `SCROLL` : 스크롤 발생 기록  

---  

## 4️. `metadata` JSON 구조 예시  
상황에 따라 다음과 같이 기록합니다.  

```js
{
  "ad_id": "ad_3",               // 광고 표시 시
  "scroll_count": 14,            // 상하좌우 스크롤 횟수
  "page_reached": 5,             // 콘텐츠 화면 마지막 페이지 번호
  "input_value": 77,             // 메뉴 2 입력 값
  "selection": "ON"              // 메뉴 3 ON/OFF 선택 값
}
```
## 5. 예시 데이터  
### (1) 로그인 화면 진입  

```js
{
  "user_id": "user_abc",
  "timestamp": "2025-07-01T09:00:00Z",
  "screen_name": "LOGIN",
  "action": "ENTER"
}
```

### (2) 메인 화면 진입 (로그인 성공 후)  
```js
{
  "user_id": "user_abc",
  "timestamp": "2025-07-01T09:00:15Z",
  "screen_name": "MAIN",
  "action": "ENTER",
  "from_screen": "LOGIN",
  "to_screen": "MAIN"
}
```

### (3) 광고 모달 표시  
```js
{
  "user_id": "user_abc",
  "timestamp": "2025-07-01T09:00:30Z",
  "screen_name": "MAIN",
  "action": "EVENT_SHOW",
  "metadata": {
    "ad_id": "ad_2"
  }
}
```

### (4) 메뉴 1 상세에서 상하좌우 스크롤  
```js
{
  "user_id": "user_abc",
  "timestamp": "2025-07-01T09:02:00Z",
  "screen_name": "MENU_1_DETAIL",
  "action": "SCROLL",
  "metadata": {
    "scroll_count": 12
  }
}
```

### (5) 메뉴 2 상세에서 값 저장 후 메인 화면으로 이동  
```js
{
  "user_id": "user_abc",
  "timestamp": "2025-07-01T09:03:00Z",
  "screen_name": "MENU_2_DETAIL",
  "action": "SAVE",
  "metadata": {
    "input_value": 45
  },
  "from_screen": "MENU_2_DETAIL",
  "to_screen": "MAIN"
}
```

## 6. 활용 예시 (집계 쿼리)  
이 데이터 구조를 사용하면 다음과 같은 집계를 수행할 수 있습니다.  

- 하루 동안 로그인 화면 접속 수  
- 이벤트 광고 화면을 가장 많이 본 사용자  
- 메인 화면 접속 시간대별 분석  
- 메뉴 1-2-3 전환 경로 분석  
- 메뉴 2 값 저장 후 메인 화면 이동 횟수  
- 메뉴 3 ON/OFF 선택 사용자 수  
- 최근 일주일 화면별 노출 수 (최소 화면 탐색)  

## 7️. 확장성  

신규 화면/이벤트 추가 시 screen_name, action, metadata 확장만으로 대응 가능  
사용자 행동 기반 퍼널 분석, 이탈 구간 분석, 전환율 측정 등에 사용 가능  
A/B 테스트 및 리텐션 분석 구조로 확장 가능  
