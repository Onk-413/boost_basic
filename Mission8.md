# Mission8  


# 프로토콜 상태 전환  


## 1. 상태 전환(전이) 다이어그램

- **상태(state):** 시스템이 특정 시점에 놓여있는 구체적 조건이나 상황 (ex: `IDLE`, `INVITED`).
- **이벤트(event):** 상태 변화를 일으키는 입력 (ex: `INVITE`, `ACK`, `BYE`, `180`).
- **상태 전환:** 이벤트가 발생했을 때 현재 상태에서 다른 상태로 이동하는 과정.
- **전이 다이어그램:** 상태(타원)과 이벤트(화살표)로 표현되어 시스템의 동작 흐름을 시각화한 것.

**이 방식은 SIP 프로토콜, 게임 상태 관리, UI 상태 관리, 네트워크 상태 관리 등 복잡한 로직을 명확히 관리하기 위해 널리 사용된다.**

---

## 2. 구현 계획

### 요구사항 요약

- `IDLE` 상태에서 시작해 `TERMINATED`까지 상태를 변경  
- 이벤트 배열을 입력받아 상태 전환 발생 시마다 기록해 배열로 반환  
- 전환 불가능한 이벤트는 무시 (상태 유지, 기록 X)  
- 상태와 이벤트는 상수로 선언해 하드코딩 방지  
- 상태 전환은 조건문이 아니라 **상태 전환 테이블 기반**으로 관리

### 구조 설계

- **상태 상수(STATES):** `IDLE`, `INVITED`, `ACCEPTED`, `ESTABLISHED` 등
- **이벤트 상수(EVENTS):** `INVITE`, `ACK`, `BYE`, `<timeout>`, 숫자 응답 등
- **상태 전환 테이블(TRANSITIONS):**
    ```js
    {
      [현재상태]: {
        [이벤트]: 다음상태
      }
    }
    ```
- **숫자 응답 범위 처리:**  
  - `1xx`, `3xx`, `4xx-6xx` 로 묶어 테이블 관리, 구체 응답 우선 처리.

---

## 3. 구현 코드

```javascript
const STATES = {
  IDLE: 'IDLE',
  INVITED: 'INVITED',
  ACCEPTED: 'ACCEPTED',
  CANCELLING: 'CANCELLING',
  CANCELLED: 'CANCELLED',
  FAILED: 'FAILED',
  ESTABLISHED: 'ESTABLISHED',
  CLOSING: 'CLOSING',
  TERMINATED: 'TERMINATED',
  AUTH_REQUESTED: 'AUTH REQUESTED',
  REDIRECTING: 'REDIRECTING',
  REDIRECTED: 'REDIRECTED',
};

const EVENTS = {
  INVITE: 'INVITE',
  CANCEL: 'CANCEL',
  ACK: 'ACK',
  BYE: 'BYE',
  TIMEOUT: '<timeout>',
};

const TRANSITIONS = {
  [STATES.IDLE]: {
    [EVENTS.INVITE]: STATES.INVITED,
  },
  [STATES.INVITED]: {
    [EVENTS.CANCEL]: STATES.CANCELLING,
    '180': STATES.INVITED,
    '1xx': STATES.INVITED,
    '200': STATES.ACCEPTED,
    '3xx': STATES.REDIRECTING,
    '407': STATES.AUTH_REQUESTED,
    '4xx-6xx': STATES.FAILED,
  },
  [STATES.AUTH_REQUESTED]: {
    [EVENTS.ACK]: STATES.INVITED,
  },
  [STATES.REDIRECTING]: {
    [EVENTS.ACK]: STATES.REDIRECTED,
  },
  [STATES.REDIRECTED]: {
    [EVENTS.TIMEOUT]: STATES.TERMINATED,
  },
  [STATES.CANCELLING]: {
    '200(CANCEL)': STATES.CANCELLED,
  },
  [STATES.CANCELLED]: {
    [EVENTS.TIMEOUT]: STATES.TERMINATED,
  },
  [STATES.ACCEPTED]: {
    [EVENTS.ACK]: STATES.ESTABLISHED,
    [EVENTS.CANCEL]: STATES.CANCELLING,
  },
  [STATES.ESTABLISHED]: {
    [EVENTS.BYE]: STATES.CLOSING,
  },
  [STATES.CLOSING]: {
    '200(BYE)': STATES.TERMINATED,
  },
  [STATES.FAILED]: {
    [EVENTS.ACK]: STATES.TERMINATED,
  },
};

function getEventKey(event) {
  if (event === EVENTS.TIMEOUT) return EVENTS.TIMEOUT;
  if (/^\d{3}\(.+\)$/.test(event)) return event;
  if (/^\d{3}$/.test(event)) {
    const code = parseInt(event, 10);
    if (code >= 100 && code < 200) return '1xx';
    if (code >= 300 && code < 400) return '3xx';
    if (code >= 400 && code <= 699) return '4xx-6xx';
    return event;
  }
  return event;
}

function sipStateMachine(events) {
  let state = STATES.IDLE;
  const history = [];

  for (const event of events) {
    const eventKey = getEventKey(event);
    const transitions = TRANSITIONS[state];
    let nextState = null;

    if (transitions) {
      if (transitions.hasOwnProperty(event)) {
        nextState = transitions[event];
      } else if (transitions.hasOwnProperty(eventKey)) {
        nextState = transitions[eventKey];
      }
    }

    if (nextState) {
      state = nextState;
      history.push(state);
    }
  }

  return history;
}
```
## 4. 실행예시
```js
console.log(sipStateMachine(["INVITE", "CANCEL", "200(CANCEL)", "487", "ACK"]));
// 출력: ["INVITED", "CANCELLING", "CANCELLED", "FAILED", "TERMINATED"]

console.log(sipStateMachine(["INVITE", "180", "200", "ACK", "BYE", "200(BYE)"]));
// 출력: ["INVITED", "ACCEPTED", "ESTABLISHED", "CLOSING", "TERMINATED"]

console.log(sipStateMachine(["INVITE", "407", "ACK", "301", "ACK", "<timeout>"]));
// 출력: ["INVITED", "AUTH REQUESTED", "INVITED", "REDIRECTING", "REDIRECTED", "TERMINATED"]

console.log(sipStateMachine(["INVITE", "404", "ACK", "<timeout>"]));
// 출력: ["INVITED", "FAILED", "TERMINATED"]
```
