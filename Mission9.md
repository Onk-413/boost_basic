# Mission9

# 윷놀이 말판

## 1️. 목표

- 배열, Map, 객체 활용으로 보드 상태 관리
- 규칙 기반의 경로 탐색 구현
- 함수 구조적 분리 및 유지보수 용이한 설계
- 오류 및 예외 처리 포함

---

## 2️. 요구사항 요약

- Z → Z로 돌아오는 윷놀이 기반 경로
- D(1), K(2), G(3), U(4), M(5)
- 분기점(W, V, X) 도착 시 짧은 경로 자동 진입
- 지나치며 진입 불가, 정확 도착 시만 진입
- 참가자 수 2~10명, 모두 동일 횟수 던져야 함
- 규칙 위반 시 "ERROR", 잘못된 문자 시 "ERR"
- 도착 위치 Z면 1점, 아니면 0점
- 출력 예: `["0, ZW5", "1, Z"]`

---

## 3️. 구조 설계

### 주요 함수

*** `parseInput(inputs)`  
- 참가자 수, 던진 횟수 확인
- 유효성 체크

*** `movePiece(start, steps)`  
- 입력 steps 만큼 경로 탐색
- 분기점 도착 시 짧은 경로 자동 진입
- 경로 dict 기반 이동

*** `calculateScore(pos)`  
- "Z" 도착 시 1, 아니면 0 반환

*** `score(inputs)`  
- 참가자별 점수 및 최종 위치 구해 배열 반환

---

## 4. 경로 Map 설계

```js
const pathMap = {
    Z: ["ZW1", "ZW2", "ZW3", "ZW4", "W"],
    W: ["WV1", "WV2", "V"],
    V: ["VZ1", "VZ2", "Z"],
    X: ["XV1", "XV2", "V"],
    Y: ["YV1", "YV2", "V"]
};
```

## 5. 전체 구현 코드

```js
function score(inputs) {
    if (inputs.length < 2 || inputs.length > 10) return ["ERROR"];
    const lengthSet = new Set(inputs.map(v => v.length));
    if (lengthSet.size !== 1) return ["ERROR"];

    const moveMap = { D: 1, K: 2, G: 3, U: 4, M: 5 };
    const pathMap = {
        Z: ["ZW1", "ZW2", "ZW3", "ZW4", "W"],
        W: ["WV1", "WV2", "V"],
        V: ["VZ1", "VZ2", "Z"],
        X: ["XV1", "XV2", "V"],
        Y: ["YV1", "YV2", "V"]
    };

    function movePiece(start, steps) {
        let current = start;
        while (steps > 0) {
            if (pathMap[current]) {
                current = pathMap[current][0]; // 다음 위치
            } else if (current.slice(0, -1) in pathMap) {
                const base = current.slice(0, -1);
                const idx = parseInt(current.slice(-1));
                if (idx < pathMap[base].length) {
                    current = pathMap[base][idx];
                } else {
                    return [current, false];
                }
            } else {
                return [current, false];
            }
            steps--;
            if (current === "Z") break;
        }
        return [current, true];
    }

    function calculateScore(pos) {
        return pos === "Z" ? 1 : 0;
    }

    const results = [];

    for (let i = 0; i < inputs.length; i++) {
        const moves = inputs[i];
        let pos = "Z";
        let valid = true;

        for (let j = 0; j < moves.length; j++) {
            const m = moves[j];
            if (!(m in moveMap)) {
                valid = false;
                break;
            }
            const steps = moveMap[m];
            const [newPos, ok] = movePiece(pos, steps);
            if (!ok) {
                valid = false;
                break;
            }
            pos = newPos;
            if (pos === "Z") break;
        }

        if (!valid) {
            results.push(`${calculateScore(pos)}, ERR`);
        } else {
            results.push(`${calculateScore(pos)}, ${pos}`);
        }
    }

    return results;
}
```

## 6. 실행 예시

예시 1
```js
console.log(score(["DGD", "MGG"]));
// 출력: ["0, ZW5", "1, Z"]
```

예시 2
```js
console.log(score(["DGGG", "MGGA"]));
// 출력: ["0, WX5", "1, ERR"]
```

예시 3
```js
console.log(score(["DGDGGK", "DDDDDK", "KKKKKD"]));
// 출력: ["1, ZW2", "0, WV2", "0, XV1"]
```

예시 4 (에러 조건: 던진 횟수 다름)
```js
console.log(score(["DG", "D"]));
// 출력: ["ERROR"]
```

## 7. 주의 사항 (실수 방지)
- 경로 딕셔너리 인덱싱 시 off-by-one 주의
- 지나치며 진입 X, 정확 도착 시만 분기 진입 가능
- 참가자 수 2~10명 필수, 던진 횟수 동일 필수
- DKGUM 이외 문자는 "ERR" 처리
- "Z" 도착 시 점수 1점 부여

## 8. 선택 규칙 파괴 연습
규칙: 분기점 도착 시 짧은 경로 강제 → 플레이어가 긴/짧은 경로 선택 가능하도록 확장 가능

방법:
입력 문자열에 "L", "S" 명령 포함해 선택 경로 전달
movePiece() 확장하여 선택 경로 처리


