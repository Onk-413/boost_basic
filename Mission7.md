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
        const fileNameWithExt = path.substring(path.lastIndexOf('/') + 1);

        // 버전 정보 제거 (_v1 ~ _v9)
        const cleanedFileName = fileNameWithExt.replace(/_v[1-9](?=\.[a-z]{2}$)/, '');

        // 3개수 누적
        countMap.set(cleanedFileName, (countMap.get(cleanedFileName) || 0) + 1);
    }

    // 2회 이상 등장한 파일만 반환
    const result = new Map();
    for (const [key, count] of countMap.entries()) {
        if (count >= 2) {
            result.set(key, count);
        }
    }

    return result;
}
```

## 3. 동작 내용
### 입력 예제

```js  
[
    "/a/a_v2.xa",
    "/b/a.xa",
    "/c/t.zz",
    "/d/a/t.xa",
    "/e/z/t_v1.zz",
    "/k/k/k/a_v9.xa"
]
```

### 처리 과정
```js  
/a/a_v2.xa → a.xa
/b/a.xa → a.xa
/c/t.zz → t.zz
/d/a/t.xa → t.xa
/e/z/t_v1.zz → t.zz
/k/k/k/a_v9.xa → a.xa
```

### 중복 파일 개수:

```js  
a.xa → 3회
t.zz → 2회
t.xa → 1회
```

### 반환 결과

```js  
Map {
  'a.xa' => 3,
  't.zz' => 2
}
```

## 4. 요약
- 디렉토리 경로와 버전을 제거하여 파일명 기준으로 비교
- 중복된 파일과 개수를 Map으로 반환
- 간단하며 확장 가능한 구조로, 추후 실제 로컬 파일 스캔 연동 시에도 재사용 가능

