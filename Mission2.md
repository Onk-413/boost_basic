# Mission2

- 기능 요구 사항 분석
    
    1~100 적힌 카드
    
    4개의 숫자 배열 [10], [30], [50], [80]
    
    참가자 3명 고정 → 10, 30, 50, 80 빼고 인당 10장씩 나눠가짐 (랜덤 구현 안해도 됨)
    
    96장에서 10장씩 분배하면 66장
    
    ```jsx
    **배열1 = [10]
    배열2 = [30]
    배열3 = [50]
    배열4 = [80]**
    ```
    
    게임 진행 방식
    
    각 턴마다 동시에 한 카드씩 내려놓고 작은거 낸 사람부터 배열 입력
    
    ex. 이민혁이 4 삼민혁이 9 사민혁이 21 냈다고 치면 카드 작은순인 4→9→21순으로 진행
    
    4는 10, 30, 50, 80 중에 10이랑 제일 가깝고 10보다 작음 → 배열1 = [10, 4] 변경 (4가 배열1 에 추가 됨) 이민혁 통과 
    
    9는 배열1의 4와 가장 가까운 수인데 4보다 커서 실패! → 삼민혁은 벌점 2점 (배열1의 요소가 2개였기 때문) / 배열1 내용 비우기 (한 번 비우고 나서 부터는 추가 불가능)
    
    21은 배열2의 30과 가장 가까운 수인데 30보다 작아서 넣을 수 있음 → 배열2 = [30, 21] 변경 (21이 배열2에 추가 됨) 삼민혁 통과 
    
    4개 배열이 다 비면 게임 종료
    
    play()함수 요구 사항 
    
    3명의 참가자에게 매 턴마다 카드 숫자 값을 입력으로 제공
    
    숫자를 보고 적당한 배열 뒤에 추가하거나 벌점+배열 비우기
    
    입력값x 또는 더 비울 배열이 없으면 게임 종료 → 종료시에는 각 참가자별 별점을 리턴함 
    
    만약 입력 배열 개수가 3의 배수가 아니라면 리턴값 모두 0점 처리 ⇒라는 말을 이해 못했었는데, 참가자가 3명이니까 점수가 3의 배수가 아니면 0으로 처리하라는 것.
    
    매개변수 **param0**는 참가자 A, B, C 순서로 매 턴마다 내려놓은 카드 3개씩 포함하는 1차원 배열 → ex. [1,2,3]  A 는 카드 1, B는 카드 2, C는 카드 3 냈음 (뒤에 숫자 3개가 추가되어도 3개씩 잘라서 abc가 낸 카드라고 생각하면 됨)
    
     
    
- 코드 구상
    
    배열 4개 선언 → 배열을 관리 할 배열도 하나 만들어야 할 것 같음 (코드 단순화하고 깔끔하게 만들려고)
    
    참여자의 벌점 저장 
    
    참여자의 카드 저장 
    
    참여자의 카드 번호 출력
    
    한 턴 진행하는 한수 
    
    -참여자가 낸 숫자와 배열 중에서 가장 가까운 숫자가 있는 배열 찾
    
    - 배열 마지막 수와 계산해서 배열에 해당 값ㅇ 추가 / 벌점 + 배열 비우기 
    

- 코드를 작성하며 생긴 오류
    
    mission2.js파일에 코드를 작성했는데 계속 이전에 작업했던 mission1.js만 출력이 되서 launch.json 파일을 확인해보니 `"program": "${workspaceFolder}\\mission1.js",` 이라고 적혀 있었다. 이 부분에서 실행할 파일을 mission1.js 로만 선택해버려서 그런 것 같아서 파일을 입력 받아서 출력하도록 `"program": "${file}",`라고 수정했다. 
    

- 작성한 코드
  ```
  // 1~100 중 10,30,50,80 제외한 카드 생성 및 랜덤 분배
const allCards = Array.from({ length: 100 }, (_, i) => i + 1).filter(v => ![10, 30, 50, 80].includes(v)); 
allCards.sort(() => Math.random() - 0.5); // 카드를 무작위로 섞음
const player1 = allCards.slice(0, 10); // player1에게 10장 분배
const player2 = allCards.slice(10, 20); // player2에게 10장 분배
const player3 = allCards.slice(20, 30); // player3에게 10장 분배

// 기준 배열 (각 배열의 첫 요소는 기준값)
let arr1 = [10];
let arr2 = [30];
let arr3 = [50];
let arr4 = [80];
let arrays = [arr1, arr2, arr3, arr4]; // 배열들을 관리하기 위한 배열

// 점수판 (플레이어별 벌점 관리)
const scores = { player1: 0, player2: 0, player3: 0 };

// 모든 배열이 비어있는지 확인하여 게임 종료 조건으로 사용
function allArraysEmpty(arrays) {
  return arrays.every(arr => arr.length === 0);
}

// 한 턴 진행 함수: 각 플레이어가 한 장씩 낸 카드를 처리
function playTurn(cards) {
  cards.sort((a, b) => a.card - b.card); // 낸 카드 작은 순으로 우선 처리

  for (let { name, card } of cards) {
    let minDiff = Infinity; // 최소 차이를 찾기 위한 초기값
    let targetArrIndex = -1; // 카드를 넣을 배열 인덱스

    // 기준 배열 중 가장 가까운 수가 있는 배열을 찾음
    arrays.forEach((arr, idx) => {
      if (arr.length === 0) return; // 비어있으면 스킵
      const last = arr[arr.length - 1]; // 배열의 마지막 값 (가장 최근 값)
      const diff = Math.abs(last - card); // 카드와의 차이 계산

      if (diff < minDiff) {
        minDiff = diff;
        targetArrIndex = idx;
      } else if (diff === minDiff) {
        // 동일한 차이라면 기준값이 큰 배열을 우선 선택
        if (arr[0] > arrays[targetArrIndex][0]) {
          targetArrIndex = idx;
        }
      }
    });

    if (targetArrIndex === -1) continue; // 배열이 모두 비어있으면 스킵

    const targetArr = arrays[targetArrIndex];
    const last = targetArr[targetArr.length - 1];

    if (card < last) {
      // 카드가 기준 배열의 마지막 값보다 작으면 추가 가능
      targetArr.push(card);
      console.log(`${name} ${card} → ${targetArr[0]} 배열에 추가됨`);
    } else {
      // 실패: 벌점 부여 및 배열 비우기
      scores[name] += targetArr.length;
      console.log(`${name} ${card} 실패! 벌점 ${targetArr.length}점, ${targetArr[0]} 배열 비움`);
      arrays[targetArrIndex] = []; // 배열 비우기
    }
  }
}

// 게임 루프: 각 턴마다 플레이어가 카드 한 장씩 내고 낸 카드는 삭제하며 진행
while (!allArraysEmpty(arrays) && (player1.length > 0 || player2.length > 0 || player3.length > 0)) {
  console.log("플레이어1:", player1);
  console.log("플레이어2:", player2);
  console.log("플레이어3:", player3);

  playTurn([
    { name: "player1", card: player1.shift() }, // 낸 패를 배열에서 삭제
    { name: "player2", card: player2.shift() }, // 낸 패를 배열에서 삭제
    { name: "player3", card: player3.shift() }  // 낸 패를 배열에서 삭제
  ]);

  console.log("현재 배열 상태:", arrays);
  console.log("현재 점수:", scores);
}

  ```
