# Mission3

각 도서의 정보를 담은 배열을 만든다.

원하는 기간과 권수를 입력하면 해당 기간에 판매 중인 책 중 그만큼 권 수가 남아있는 책을 알려준다 (절판된 책은 *이 붙어서 출력됨)

사진으로 준 기간에만 해당 책을 구할 수 있도록 가정하여 작업한다. 

- 문제점
    
    for이나 while같은 반복문을 넣어서 책을 계속 찾되, 그만이라는 단어가 입력되면 멈추도록 하고싶은데 js를 잘 몰라서 다른 언어 문법과 많이 헷갈렸다. 
    
    처음엔 다른 언어들도 for, whlie을 거의 다 사용하니 js에도 있겠지? 하고 넣었는데 오류가 나서 찾아보니 비동기 코드 구조상 적합하지 않다고 한다… 그래서 해당 부분을 함수로 만들어서 함수를 계속 실행하도록 했다. 
    
    ```jsx
    while(true){
        rl.question("날짜(YYYYMM)와 권수를 공백으로 구분하여 입력하세요 (예: 198402 2): ", (input) => {
      if(input === "그만")
        break;
      const [param0, param1Str] = input.trim().split(' ');
      const param1 = parseInt(param1Str, 10);
    
      const result = find(param0, param1);
      console.log(result);
    
      rl.close();
    
      
    });
    
    ```
    
    ```jsx
    console.log('도서 검색기입니다. 종료하려면 "그만"을 입력하세요.');
    
    function promptInput() {
      rl.question("날짜(YYYYMM)와 권수를 공백으로 구분하여 입력하세요 (예: 198402 2): ", (input) => {
        if (input.trim().toLowerCase() === "그만") {
          console.log("프로그램을 종료합니다.");
          rl.close();
          return;
        }
    
        const [param0, param1Str] = input.trim().split(' ');
        const param1 = parseInt(param1Str, 10);
    
        if (!param0 || isNaN(param1)) {
          console.log("잘못된 입력입니다. 다시 시도하세요.");
        } else {
          const result = find(param0, param1);
          console.log(result);
        }
    
        // 다음 입력을 기다림
        promptInput();
      });
    }
    ```
    
- 완성한 코드
    
    ```jsx
    function find(param0, param1) {
      const books = [
        { name: "Great", outOfPrint: true, type: "Novel", rating: 3.1, count: 2, start: "198001", end: "198512" },
        { name: "Laws", outOfPrint: true, type: "Novel", rating: 4.8, count: 3, start: "198101", end: "198805" },
        { name: "Dracula", outOfPrint: true, type: "Drama", rating: 2.3, count: 6, start: "197901", end: "198912" },
        { name: "Mario", outOfPrint: true, type: "Drama", rating: 3.8, count: 4, start: "198305", end: "199611" },
        { name: "House", outOfPrint: false, type: "Magazine", rating: 4.4, count: 1, start: "199001", end: "200001" },
        { name: "Art1", outOfPrint: true, type: "Design", rating: 4.2, count: 2, start: "199001", end: "199512" },
        { name: "Art2", outOfPrint: true, type: "Design", rating: 3.0, count: 3, start: "198810", end: "199912" },
        { name: "Wars", outOfPrint: true, type: "Novel", rating: 4.6, count: 2, start: "198404", end: "199004" },
        { name: "Solo", outOfPrint: false, type: "Poem", rating: 4.9, count: 2, start: "199911", end: "200512" },
        { name: "Lost", outOfPrint: false, type: "Web", rating: 3.2, count: 8, start: "199907", end: "200912" },
        { name: "Ocean", outOfPrint: true, type: "Magazine", rating: 4.3, count: 1, start: "199101", end: "199912" },
      ];
    
      const date = param0;
      const minCount = param1;
    
      const result = books
        .filter(book =>
          date >= book.start &&
          date <= book.end &&
          book.count >= minCount
        )
        .sort((a, b) => b.rating - a.rating)
        .map(book => {
          const star = book.outOfPrint ? '*' : '';
          return `${book.name}${star}(${book.type}) ${book.rating}`;
        });
    
      return result.length > 0 ? result.join(', ') : "!EMPTY";
    }
    
    // 입력을 반복해서 받기 위한 부분
    const readline = require('readline');
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });
    
    console.log('도서 검색기입니다. 종료하려면 "그만"을 입력하세요.');
    
    function promptInput() {
      rl.question("날짜(YYYYMM)와 권수를 공백으로 구분하여 입력하세요 (예: 198402 2): ", (input) => {
        if (input.trim().toLowerCase() === "그만") {
          console.log("프로그램을 종료합니다.");
          rl.close();
          return;
        }
    
        const [param0, param1Str] = input.trim().split(' ');
        const param1 = parseInt(param1Str, 10);
    
        if (!param0 || isNaN(param1)) {
          console.log("잘못된 입력입니다. 다시 시도하세요.");
        } else {
          const result = find(param0, param1);
          console.log(result);
        }
    
        // 다음 입력을 기다림
        promptInput();
      });
    }
    
    // 첫 입력 요청
    promptInput();
    
    ```
    
- 실행예시
    
    `도서 검색기입니다. 종료하려면 "그만"을 입력하세요.
    날짜(YYYYMM)와 권수를 공백으로 구분하여 입력하세요 (예: 198402 2): 200008 6
    Lost(Web) 3.2
    날짜(YYYYMM)와 권수를 공백으로 구분하여 입력하세요 (예: 198402 2):`
