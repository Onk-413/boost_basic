# Mission1

- 출력 결과
- ```js
    from= 1 , dice= 3 , next= 14  
    from= 14 , dice= 4 , next= 18  
    from= 18 , dice= 3 , next= 63  
    from= 63 , dice= 5 , next= 68  
    from= 68 , dice= 1 , next= 69  
```
    
- 출력 결과 분석
    주사위 숫자만큼 이동해서 도착할 위치를 계산하는 판단조건, 변수의 의미 

    사용된 변수
    -start (현 위치, 즉 시작하는 위치)
    -dice (주사위를 굴려서 나온 숫자)
    -next (현 위치에서 주사위의 숫자만큼 이동한 위치, 즉 다음 위치)
    
    사용된 함수
    -nextPosition 
    
    2번라인에서 current, dice값을 참조하여  next 값을 두 값을 더한 값으로 상수선언. 
    if-else문의 조건으로 next의 값이 사다리가 있는 숫자인 4, 8, 21, 28, 50, 71, 80 중 하나면 dice의 값에 사다리를 타고 올라갔을 때의 칸이 나오도록 숫자를 더하여서 값을 반환함. 아닐 경우의 디폴트 값은 dice를 리턴.

    
- 출력 결과 오류
    1에서 시작, 주사위 3, (4에서 사다리) 14로 이동 ⇒ 현 위치 14
    14에서 시작, 주사위 4, 18로 이동 ⇒ 현 위치 18
    18에서 시작, 주사위 3, 63로 이동 ⇒ 현 위치 63 (원래 위치는 42)
    63에서 시작, 주사위 5, 68로 이동 ⇒ 현 위치 68 
    68에서 시작, 주사위 1, 69로 이동 ⇒ 현 위치 69

  오류 원인: line 13
    사다리가 있는 숫자인 4, 8, 21, 28, 50, 71, 80에는 값이 리턴 될 때 사다리로 올라간 값이 출력되도록 숫자를 더해야함. 
    그런데 해당 코드에서는 dice의 값에 42를 더해서 숫자가 잘못 출력되었음. 
    

- 원본 코드
    
    ```jsx
    function nextPosition(current, dice) {
        const next = current + dice;
        if (next == 4) {
            return dice + 10;
        }
        else if (next == 8) {
            return dice + 22;
        }
        else if (next == 28) {
            return dice + 48;
        }
        else if (next == 21) {
            return dice + 42;
        }
        else if (next == 50) {
            return dice + 17;
        }
        else if (next == 71) {
            return dice + 92;
        }
        else if (next == 80) {
            return dice + 19;
        }
        
        return dice;    
    }
    
    let start = 1;
    let next = 1;
    let dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 4;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 5;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 1;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    ```
    
- 수정된 코드
    
    ```jsx
    function nextPosition(current, dice) {
        const next = current + dice;
        if (next == 4) {
            return dice + 10;
        }
        else if (next == 8) {
            return dice + 22;
        }
        else if (next == 28) {
            return dice + 48;
        }
        else if (next == 21) {
            return dice + 21;
        }
        else if (next == 50) {
            return dice + 17;
        }
        else if (next == 71) {
            return dice + 92;
        }
        else if (next == 80) {
            return dice + 19;
        }
        
        return dice;    
    }
    
    let start = 1;
    let next = 1;
    let dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 4;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 5;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 1;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    ```

    
- 수정된 코드의 출력 결과
    from= 1 , dice= 3 , next= 14
    from= 14 , dice= 4 , next= 18
    from= 18 , dice= 3 , next= 42
    from= 42 , dice= 5 , next= 47
    from= 47 , dice= 1 , next= 48
    

- 새로운 요구사항 추가하기
    요구사항: 뱀에 물리면 낮은 칸으로 떨어진다.
    32→10
    36→6
    48→26
    62→18
    88→24
    95→56
    97→78
    
    조건1. 사다리/뱀으로 함수를 분리한다. 
    조건2. 구현 후 동작을 확인하기 위한 입력/출력 조건을 추가

    
- 조건에 맞춰 수정된 코드
    
    ```jsx
    function nextPosition_ladder(current, dice) {
        const next = current + dice;
        if (next == 4) {
            return dice + 10;
        }
        else if (next == 8) {
            return dice + 22;
        }
        else if (next == 28) {
            return dice + 48;
        }
        else if (next == 21) {
            return dice + 21;
        }
        else if (next == 50) {
            return dice + 17;
        }
        else if (next == 71) {
            return dice + 92;
        }
        else if (next == 80) {
            return dice + 19;
        }
        
        return dice;    
    }
    
    function nextPosition_snake(current, dice) {
        const next = current + dice;
        if (next == 32) {
            return dice - 22;
        }
        else if (next == 36) {
            return dice - 30;
        }
        else if (next == 48) {
            return dice - 22;
        }
        else if (next == 62) {
            return dice - 44;
        }
        else if (next == 88) {
            return dice - 64;
        }
        else if (next == 95) {
            return dice - 39;
        }
        else if (next == 97) {
            return dice - 19;
        }
        
        return dice;    
    }
    
    function nextPosition(current, dice){
        dice = nextPosition_ladder(current, dice);
        dice = nextPosition_snake(current, dice);
        return dice;
    }
    
    let start = 1;
    let dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 4;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 3;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 5;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    start = next;
    dice = 1;
    next = start + nextPosition(start, dice);
    console.log("from=",start,", dice=",dice,", next=", next);
    
    ```
    
- 조건에 맞춰 수정된 코드의 출력 결과
    from= 1 , dice= 3 , next= 14
    from= 14 , dice= 4 , next= 18
    from= 18 , dice= 3 , next= 42
    from= 42 , dice= 5 , next= 47
    from= 47 , dice= 1 , next= 26
    

- 실수했던 코드1
    
    문제점
    1. function nextPosition(current, dice) 의 내부 함수들이 실행이 아닌 선언만 되어 실행되지 않음.
    2. nextPosition 함수가 실제로 사다리→뱀 순으로 실행 후 반환되지 않음
    3. next = start + c;에서  nextPosition(current, dice)이 아니라 c로 오타가 난 것을 미처 확인 못함
    4. next값을  nextPosition_ladder에서 nextPositio으로 변경했어야 했는데 수정을 깜빡해서 뱀 적용 없이 사다리만 호출됨
    
    에러
     Cannot find module 'C:\Users\사용\boostcamp\maission1.js’
    
    해결 방법 
    1. nextPosition 함수 내부에 사다리 → 뱀 순으로 모두 적용 되도록 코드를 수정함 
    2. next값을  nextPosition_ladder에서 nextPositio으로 변경하여 nextPosition에서 dice의 값을 반환하도록 함
    3. next = start + c;에서 c를  nextPosition(current, dice)로 변경함
