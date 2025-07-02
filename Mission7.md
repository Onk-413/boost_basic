# Mission7  
  
# 파일중복  
  
## 1. 구현 계획

디렉토리 경로 + 파일명이 담긴 배열에서 버전을 제거한 원본 파일명 기준으로 중복된 파일을 찾아 개수와 함께 반환하는 match() 함수 구현

### 요구사항 요약
- 파일명은 영문 소문자 + .aa ~ .zz 형태로 고정
- _v1 ~ _v9 버전 정보는 제거하여 동일 파일로 인식
- 디렉토리 경로가 포함되나, 파일명만 추출하여 비교
- 2회 이상 중복된 파일만 Map 형태로 반환

### 처리 흐름
1. 입력 배열을 순회하여:  
- 마지막 / 뒤의 파일명만 추출
- _v[1-9] 패턴이 있으면 제거
- 카운트 맵에 개수 누적
2. 카운트가 2 이상인 항목만 추출하여 Map으로 반환

## 2. 구현한 코드

```js  
function match(filePaths) {
    const countMap = new Map();

    for (const path of filePaths) {
        // 경로에서 파일명 추출
        const fileName = path.substring(path.lastIndexOf('/') + 1);

        // 버전 정보 제거 (_v1 ~ _v9)
        const cleanedFileName = fileName.replace(/_v[1-9](?=\.[a-z]{2}$)/, '');

        // 개수 누적
        countMap.set(cleanedFileName, (countMap.get(cleanedFileName) || 0) + 1);
    }

    // 2회 이상 등장한 파일만 객체로 추출
    const result = {};
    for (const [key, count] of countMap.entries()) {
        if (count >= 2) {
            result[key] = count;
        }
    }

    return result;
}

// 테스트 예시

console.log(JSON.stringify(match([
    "/a/a_v2.xa",
    "/b/a.xa",
    "/c/t.zz",
    "/d/a/t.xa",
    "/e/z/t_v1.zz",
    "/k/k/k/a_v9.xa"
])));
// {"a.xa":3,"t.zz":2}

console.log(JSON.stringify(match([
    "/t.zp",
    "/z/z_v2.zp",
    "/a.za",
    "/d/b.zb",
    "/d/a/t.zp"
])));
// {"t.zp":2}

console.log(JSON.stringify(match([
    "/t.yg",
    "/b/b.zg",
    "/a.zg",
    "/e/a.zz",
    "/d/a/x_v2.zg"
])));
// {}

```

## 3. 동작 내용
### 입력 예제

```js  
[
    "/a/a_v2.xa",
    "/b/a.xa",
    "/c/t.zz",
    "/d/a정
```js
// 문제 풀이 부분
function match(param0) {
  const countMap = new Map();

  for (const path of param0) {
    // 경로에서 파일명 추출
    const fileName = path.substring(path.lastIndexOf('/') + 1);

    // 버전 정보 제거 (_v1 ~ _v9)
    const cleanedFileName = fileName.replace(/_v[1-9](?=\.[a-z]{2}$)/, '');

    // 개수 누적
    countMap.set(cleanedFileName, (countMap.get(cleanedFileName) || 0) + 1);
  }

  // 2회 이상 등장한 파일만 Map으로 반환
  const result = new Map();
  for (const [key, count] of countMap.entries()) {
    if (count >= 2) {
      result.set(key, count.toString());
    }
  }

  return result;
}

// 데이터 입력/출력 부분
const readline = require('readline');
const rl = readline.createInterface({
	input: process.stdin,
	output: process.stdout
});

let inputs = [];
rl.on('line', (line) => {
	inputs.push(line);
	if (inputs.length === 1) {
		rl.close();
	}
});

rl.on('close', () => {
	const fileArray = inputs[0].split(',');
	const answer = match(fileArray);
        if (answer.size == 0) {
             console.log("!EMPTY");
             rl.close();
             return;
        }
	for (const [key, value] of answer){
		console.log(key+"="+value);
	}
	rl.close();
});
```
