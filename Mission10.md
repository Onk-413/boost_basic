# Mission 10

# Model File String Reading Guide

### 문자가 저장된 파일을 읽기 처리하는 방법
- `fs.readFileSync(path, 'utf-8')`를 사용하여 파일의 문자열 내용을 한 번에 읽어온다.
- `split('\n')`으로 줄 단위로 파싱 후 `map`, `filter`, `slice` 등을 사용하여 필요한 데이터만 배열로 정리한다.
- 정규식을 사용하여 `match(/pattern/)`으로 원하는 값 추출 후 `parseFloat`, `parseInt`로 숫자로 변환한다.

---

### 기능 요구사항 요약
- 제공된 models.zip 압축 해제 후 구조를 유지한 상태로 분석.
- `data.model`에서 objectid와 경로 파싱.
- 각 모델 파일에서 vertex, triangle 데이터를 파싱.
- Triangle 중 X, Y, Z 축으로 가장 긴 Triangle과 면적이 가장 큰 Triangle의 objectid 출력.

### 프로그래밍 요구사항 요약
- 5초 동안 분석 진행 Progress Bar를 터미널에 출력.
- 최소 10% 단위 이상으로 진행률을 갱신.
- 분석 완료 후 분석 결과를 출력.

---

## 구현 코드
```js
const fs = require('fs');
const path = require('path');

function parseModelFile(filePath) {
    const content = fs.readFileSync(filePath, 'utf-8');
    const [vertexSection, triangleSection] = content.split('\n\n');

    const vertexLines = vertexSection.trim().split('\n').slice(1);
    const vertices = vertexLines.map(line => {
        const x = parseFloat(line.match(/x="([^"]+)"/)[1]);
        const y = parseFloat(line.match(/y="([^"]+)"/)[1]);
        const z = parseFloat(line.match(/z="([^"]+)"/)[1]);
        return { x, y, z };
    });

    const triangleLines = triangleSection.trim().split('\n').slice(1);
    const triangles = triangleLines.map(line => {
        const v1 = parseInt(line.match(/v1="(\d+)"/)[1]);
        const v2 = parseInt(line.match(/v2="(\d+)"/)[1]);
        const v3 = parseInt(line.match(/v3="(\d+)"/)[1]);
        return { v1, v2, v3 };
    });

    return { vertices, triangles };
}

function calculateTriangleMetrics(vertices, triangle) {
    const p1 = vertices[triangle.v1];
    const p2 = vertices[triangle.v2];
    const p3 = vertices[triangle.v3];

    const xLen = Math.max(p1.x, p2.x, p3.x) - Math.min(p1.x, p2.x, p3.x);
    const yLen = Math.max(p1.y, p2.y, p3.y) - Math.min(p1.y, p2.y, p3.y);
    const zLen = Math.max(p1.z, p2.z, p3.z) - Math.min(p1.z, p2.z, p3.z);

    const a = Math.hypot(p2.x - p1.x, p2.y - p1.y, p2.z - p1.z);
    const b = Math.hypot(p3.x - p2.x, p3.y - p2.y, p3.z - p2.z);
    const c = Math.hypot(p1.x - p3.x, p1.y - p3.y, p1.z - p3.z);
    const s = (a + b + c) / 2;
    const area = Math.sqrt(s * (s - a) * (s - b) * (s - c));

    return { xLen, yLen, zLen, area };
}

function showProgressBar() {
    const total = 50;
    for (let i = 0; i <= total; i++) {
        const percent = Math.floor((i / total) * 100);
        const bar = '▓'.repeat(i) + '░'.repeat(total - i);
        process.stdout.write(`\r${bar} ${percent}%`);
        Atomics.wait(new Int32Array(new SharedArrayBuffer(4)), 0, 0, 100);
    }
    console.log('\n');
}

function analyzeModels(basePath) {
    const dataModelContent = fs.readFileSync(path.join(basePath, 'data.model'), 'utf-8');
    const objectPaths = {};
    dataModelContent.trim().split('\n').slice(1).forEach(line => {
        const modelPath = line.match(/path="([^"]+)"/)[1];
        const objectId = line.match(/objectid="(\d+)"/)[1];
        objectPaths[objectId] = path.join(basePath, modelPath);
    });

    let maxX = { len: -Infinity, objectId: null };
    let maxY = { len: -Infinity, objectId: null };
    let maxZ = { len: -Infinity, objectId: null };
    let maxArea = { area: -Infinity, objectId: null };

    Object.entries(objectPaths).forEach(([objectId, filePath]) => {
        const { vertices, triangles } = parseModelFile(filePath);
        triangles.forEach(triangle => {
            const { xLen, yLen, zLen, area } = calculateTriangleMetrics(vertices, triangle);
            if (xLen > maxX.len) maxX = { len: xLen, objectId };
            if (yLen > maxY.len) maxY = { len: yLen, objectId };
            if (zLen > maxZ.len) maxZ = { len: zLen, objectId };
            if (area > maxArea.area) maxArea = { area, objectId };
        });
    });

    showProgressBar();

    console.log(`X 축으로 가장 긴 Triangle objectid: ${maxX.objectId}`);
    console.log(`Y 축으로 가장 긴 Triangle objectid: ${maxY.objectId}`);
    console.log(`Z 축으로 가장 긴 Triangle objectid: ${maxZ.objectId}`);
    console.log(`가장 면적이 넓은 Triangle objectid: ${maxArea.objectId}`);
}

// 실제 실행 시:
// analyzeModels('./models');
```
위 Node.js 코드를 사용하여 압축 해제된 models 디렉토리 내 모델 데이터를 분석합니다.

## 실행 예시
```jd
▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░ 50%
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ 100%

X 축으로 가장 긴 Triangle objectid: 1
Y 축으로 가장 긴 Triangle objectid: 5
Z 축으로 가장 긴 Triangle objectid: 9
가장 면적이 넓은 Triangle objectid: 5
