#Mission5. AST 구조 설계 및 분석 리포트

> 작성일: 2025-06-30  
> 작성자: onk-413 
> 주제: `var a = new A.init();` 한 줄 코드의 AST 구조 설계 및 리서치 결과 정리

---
## 들어가기에 앞서 AST 구조란?  
**AST**는 Abstract Syntax Tree의 약자로, 한국어로는 추상 구문 트리라고 합니다.  
프로그래밍 언어의 소스 코드를 트리 구조로 표현한 것 또는 AST는 소스코드를 컴퓨터가 이해하기 쉬운 구조로 바꾼 것  

이 트리는 코드의 문법적 구조를 계층적으로 보여줘서, 컴파일러나 인터프리터가 코드를 이해하고 처리하는 데 핵심 역할을 합니다.  

**AST가 하는 역할**    
*소스 코드를 토큰화(분해)하고, 문법에 맞게 해석한 결과물  
*실제 코드 실행 전, 코드 구조와 의미를 분석하는 데 쓰임  
*변수 선언, 함수 호출, 연산 등 코드의 각 요소를 노드(node)로 표현  
*이후 최적화, 코드 변환, 에러 검출 등에 활용됨  

---
## 1. 현재까지의 컴파일러 동작 방식 이해

컴파일러는 소스 코드를 실행 가능한 형태로 바꾸기 위해 아래와 같은 단계를 거칩니다. 

1. **Lexical Analysis (어휘 분석)**: 소스 코드를 토큰 단위로 분해  
2. **Parsing (구문 분석)**: 문법 구조에 따라 트리(AST) 생성  
3. **Semantic Analysis (의미 분석)**: 변수 선언, 타입, 참조 등 의미적 오류 확인  
4. **Intermediate Representation (IR)**: AST를 기반으로 중간 코드 생성  
5. **Optimization**: 성능 향상 목적의 코드 최적화  
6. **Code Generation**: 목적 플랫폼에 맞는 실제 실행 코드 생성

특히 AST(Abstract Syntax Tree)는 소스 코드의 구조적 정보를 보존하는 핵심 자료 구조로, 이후 의미 분석, 최적화, 코드 생성 등 거의 모든 단계에 활용됩니다.

---

## 2. 학습 자료 정리

| 번호 | 자료명                      | 링크                                                                 | 핵심 학습 내용                       | 활용 목적                      |
|------|----------------------------|----------------------------------------------------------------------|------------------------------------|------------------------------|
| 1    | 컴파일러 기본 강의 (YouTube) | [🔗](https://www.youtube.com/watch?v=ZI198eFghJk)          | 컴파일러 파이프라인 흐름 이해       | 전체 컴파일러 구조 정리       |
| 2    | Lessons from Writing a Compiler |[🔗](https://borretti.me/article/lessons-writing-compiler) | AST 설계 실수 및 구현 교훈          | AST 설계 시 유의점 학습       |
| 3    | Cranelift e-graph RFC        | [🔗](https://github.com/bytecodealliance/rfcs/blob/main/accepted/cranelift-egraph.md) | e-graph 기반 최적화와 IR 변환 이해 | 최적화 구조 및 IR 설계 학습   |
| 4    | Gist: AST 예제 코드          | [🔗](https://gist.github.com/pizlonator/cf1e72b8600b1437dda8153ea3fdb963) | AST 트리 구조 구현 코드            | AST 노드 구현 방식 실습       |
| 5    | WebKit Speculation in JavaScriptCore | [🔗](https://webkit.org/blog/10308/speculation-in-javascriptcore/) | 런타임 추측 실행 최적화 기법       | JIT 최적화 동작 이해          |
| 6    | A Python Interpreter written in Python | [🔗](https://aosabook.org/en/500L/a-python-interpreter-written-in-python.html) | 파이썬 메타 해석기 구현 사례       | 인터프리터 구조 직접 구현 경험 |
| 7    | SIL High-Level IR (LLVM 세미나) | [🔗](https://llvm.org/devmtg/2015-10/slides/GroffLattner-SILHighLevelIR.pdf) | Swift SIL IR 설계와 LLVM 매핑      | 고수준 IR 설계 및 활용        |
| 8    | Kastree (Kotlin AST 라이브러리) | [🔗](https://github.com/cretz/kastree)                      | Kotlin AST 생성기 및 파싱 기술      | Kotlin AST 생성 및 변환 학습  |
| 9    | Kotlin IR 구조 설명          | [🔗](https://akuleshov7.com/2021-06-24-kotlin-representation.html) | Kotlin IR 설계 및 API 이해          | IR 기반 언어 도구 설계 학습   |


추가로 ECMAScript 기반 AST 명세인 [ESTree](https://github.com/estree/estree)를 참조해 JavaScript 문법에 대한 AST 노드 구조를 명확히 파악했습니다.

---

## 3. 코드 한 줄에 대한 AST 설계

###  대상 코드

```js
var a = new A.init();
```

이 코드는 객체 생성과 초기화를 포함하는 JavaScript 문장입니다. 이를 AST 형태로 해석하면 아래와 같은 트리 구조가 됩니다.

#AST 구조 다이어그램
```js
Program
 └── VariableDeclaration (kind: "var")
     └── VariableDeclarator
         ├── id: Identifier(name="a")
         └── init: NewExpression
             ├── callee: MemberExpression
             │   ├── object: Identifier(name="A")
             │   └── property: Identifier(name="init")
             └── arguments: []
```
#AST 노드 데이터 구조 (Python-like Pseudocode)
```js
class Node: pass

class Identifier(Node):
    def __init__(self, name): self.name = name

class MemberExpression(Node):
    def __init__(self, object, property):
        self.object = object
        self.property = property

class NewExpression(Node):
    def __init__(self, callee, arguments):
        self.callee = callee
        self.arguments = arguments

class VariableDeclarator(Node):
    def __init__(self, id, init):
        self.id = id
        self.init = init

class VariableDeclaration(Node):
    def __init__(self, declarations, kind):
        self.declarations = declarations
        self.kind = kind

# 전체 AST
ast = VariableDeclaration(
    declarations=[
        VariableDeclarator(
            id=Identifier("a"),
            init=NewExpression(
                callee=MemberExpression(
                    object=Identifier("A"),
                    property=Identifier("init")
                ),
                arguments=[]
            )
        )
    ],
    kind="var"
)
```

4. 종합적 학습 정리 및 메타 인사이트
배운 점 요약
AST 설계의 중요성: 코드 구조를 명확히 모델링하는 트리 설계가 전체 컴파일러 품질에 큰 영향을 미친다.

노드 분리의 필요성: 식(expression)과 문(statement)을 명확히 구분해야 한다. 예: Identifier, NewExpression, VariableDeclarator 등

언어별 AST 차이 이해: JavaScript와 Kotlin의 AST 구조 차이를 통해 공통적 추상화 방식과 언어 특수성을 비교할 수 있었다.

도구 활용: kastree, ESTree, Gist 예제 등을 통해 실제 코드를 다뤄보며 개념을 현실화할 수 있었다.

다이어그램
```js
[ Source Code ]
      ↓
[ Tokenizer ]
      ↓
[ Parser ]
      ↓
[ AST ]
      ↓
[ Semantic Analyzer ]
      ↓
[ IR Generator ]
      ↓
[ Optimizer ]
      ↓
[ Machine Code ]
```
##결론  
처음엔 어디서부터 공부를 시작해야하는지, 어떻게 공부를 해야할지 막막했는데 댓글에서   
> 컴파일 과정에서 “코드 -> 중간과정 -> AST -> 컴파일러 작동 -> 바이너리 (or etc...) 프로그램 “이 과정중 "중간과정 -> AST" 에 해당하는 과정을 찾아서 공부하면 이해가 빠를거에요!
라고 하시는 것을 보고 공부에 적용해보았습니다.   

이번 리서치를 통해 AST의 실질적인 구성 방식, 언어별 차이, 트리 구조 구현 방식 등을 명확히 이해할 수 있었습니다.   
특히 AST 구조를 "실제 코드"로 표현하면서 추상적 개념이 어떻게 코드와 연결되는지 실감할 수 있었습니다. 앞으로 더 복잡한 문장이나 함수 선언, 조건문 등을 포함한 AST 확장 설계로 발전해 나갈 계획입니다. 아직 모르는 게 많지만 계속해서 학습하여 활용할 수 있도록 노력하겠습니다. 



