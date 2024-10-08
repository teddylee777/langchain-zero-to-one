


<style>
    .custom {
        background-color: #008d8d;
        color: white;
        padding: 0.25em 0.5em 0.25em 0.5em;
        white-space: pre-wrap;       /* css-3 */
        white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
        white-space: -pre-wrap;      /* Opera 4-6 */
        white-space: -o-pre-wrap;    /* Opera 7 */
        word-wrap: break-word;    
    }
    
    pre {
        background-color: #027c7c;
        padding-left: 0.5em;
    }
</style>
    

## LCEL 인터페이스

사용자 정의 체인을 가능한 쉽게 만들 수 있도록, [`Runnable`](https://api.python.langchain.com/en/stable/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable) 프로토콜을 구현했습니다. 

`Runnable` 프로토콜은 대부분의 컴포넌트에 구현되어 있습니다.

이는 표준 인터페이스로, 사용자 정의 체인을 정의하고 표준 방식으로 호출하는 것을 쉽게 만듭니다.
표준 인터페이스에는 다음이 포함됩니다.

- [`stream`](#stream): 응답의 청크를 스트리밍합니다.
- [`invoke`](#invoke): 입력에 대해 체인을 호출합니다.
- [`batch`](#batch): 입력 목록에 대해 체인을 호출합니다.

비동기 메소드도 있습니다.

- [`astream`](#async-stream): 비동기적으로 응답의 청크를 스트리밍합니다.
- [`ainvoke`](#async-invoke): 비동기적으로 입력에 대해 체인을 호출합니다.
- [`abatch`](#async-batch): 비동기적으로 입력 목록에 대해 체인을 호출합니다.
- [`astream_log`](#async-stream-intermediate-steps): 최종 응답뿐만 아니라 발생하는 중간 단계를 스트리밍합니다.

```python
# API KEY를 환경변수로 관리하기 위한 설정 파일
from dotenv import load_dotenv

# API KEY 정보로드
load_dotenv()
```

```python
# LangSmith 추적을 설정합니다. https://smith.langchain.com
# !pip install -qU langchain-teddynote
from langchain_teddynote import logging

# 프로젝트 이름을 입력합니다.
logging.langsmith("CH01-Basic")
```
LCEL 문법을 사용하여 chain 을 생성합니다.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

# ChatOpenAI 모델을 인스턴스화합니다.
model = ChatOpenAI()
# 주어진 토픽에 대한 농담을 요청하는 프롬프트 템플릿을 생성합니다.
prompt = PromptTemplate.from_template("{topic} 에 대하여 3문장으로 설명해줘.")
# 프롬프트와 모델을 연결하여 대화 체인을 생성합니다.
chain = prompt | model | StrOutputParser()
```
## stream: 실시간 출력

이 함수는 `chain.stream` 메서드를 사용하여 주어진 토픽에 대한 데이터 스트림을 생성하고, 이 스트림을 반복하여 각 데이터의 내용(`content`)을 즉시 출력합니다. `end=""` 인자는 출력 후 줄바꿈을 하지 않도록 설정하며, `flush=True` 인자는 출력 버퍼를 즉시 비우도록 합니다. 

```python
# chain.stream 메서드를 사용하여 '멀티모달' 토픽에 대한 스트림을 생성하고 반복합니다.
for token in chain.stream({"topic": "멀티모달"}):
    # 스트림에서 받은 데이터의 내용을 출력합니다. 줄바꿈 없이 이어서 출력하고, 버퍼를 즉시 비웁니다.
    print(token, end="", flush=True)
```
## invoke: 호출

`chain` 객체의 `invoke` 메서드는 주제를 인자로 받아 해당 주제에 대한 처리를 수행합니다.

```python
# chain 객체의 invoke 메서드를 호출하고, 'ChatGPT'라는 주제로 딕셔너리를 전달합니다.
chain.invoke({"topic": "ChatGPT"})
```
## batch: 배치(단위 실행)

함수 `chain.batch`는 여러 개의 딕셔너리를 포함하는 리스트를 인자로 받아, 각 딕셔너리에 있는 `topic` 키의 값을 사용하여 일괄 처리를 수행합니다.

```python
# 주어진 토픽 리스트를 batch 처리하는 함수 호출
chain.batch([{"topic": "ChatGPT"}, {"topic": "Instagram"}])
```
`max_concurrency` 매개변수를 사용하여 동시 요청 수를 설정할 수 있습니다

`config` 딕셔너리는 `max_concurrency` 키를 통해 동시에 처리할 수 있는 최대 작업 수를 설정합니다. 여기서는 최대 3개의 작업을 동시에 처리하도록 설정되어 있습니다.

```python
chain.batch(
    [
        {"topic": "ChatGPT"},
        {"topic": "Instagram"},
        {"topic": "멀티모달"},
        {"topic": "프로그래밍"},
        {"topic": "머신러닝"},
    ],
    config={"max_concurrency": 3},
)
```
## async stream: 비동기 스트림

함수 `chain.astream`은 비동기 스트림을 생성하며, 주어진 토픽에 대한 메시지를 비동기적으로 처리합니다.

비동기 for 루프(`async for`)를 사용하여 스트림에서 메시지를 순차적으로 받아오고, `print` 함수를 통해 메시지의 내용(`s.content`)을 즉시 출력합니다. `end=""`는 출력 후 줄바꿈을 하지 않도록 설정하며, `flush=True`는 출력 버퍼를 강제로 비워 즉시 출력되도록 합니다.


```python
# 비동기 스트림을 사용하여 'YouTube' 토픽의 메시지를 처리합니다.
async for token in chain.astream({"topic": "YouTube"}):
    # 메시지 내용을 출력합니다. 줄바꿈 없이 바로 출력하고 버퍼를 비웁니다.
    print(token, end="", flush=True)
```
## async invoke: 비동기 호출

`chain` 객체의 `ainvoke` 메서드는 비동기적으로 주어진 인자를 사용하여 작업을 수행합니다. 여기서는 `topic`이라는 키와 `NVDA`(엔비디아의 티커) 라는 값을 가진 딕셔너리를 인자로 전달하고 있습니다. 이 메서드는 특정 토픽에 대한 처리를 비동기적으로 요청하는 데 사용될 수 있습니다.


```python
# 비동기 체인 객체의 'ainvoke' 메서드를 호출하여 'NVDA' 토픽을 처리합니다.
my_process = chain.ainvoke({"topic": "NVDA"})
```

```python
# 비동기로 처리되는 프로세스가 완료될 때까지 기다립니다.
await my_process
```
## async batch: 비동기 배치

함수 `abatch`는 비동기적으로 일련의 작업을 일괄 처리합니다.

이 예시에서는 `chain` 객체의 `abatch` 메서드를 사용하여 `topic` 에 대한 작업을 비동기적으로 처리하고 있습니다.

`await` 키워드는 해당 비동기 작업이 완료될 때까지 기다리는 데 사용됩니다.


```python
# 주어진 토픽에 대해 비동기적으로 일괄 처리를 수행합니다.
my_abatch_process = chain.abatch(
    [{"topic": "YouTube"}, {"topic": "Instagram"}, {"topic": "Facebook"}]
)
```

```python
# 비동기로 처리되는 일괄 처리 프로세스가 완료될 때까지 기다립니다.
await my_abatch_process
```
## Parallel: 병렬성

LangChain Expression Language가 병렬 요청을 지원하는 방법을 살펴봅시다.
예를 들어, `RunnableParallel`을 사용할 때, 각 요소를 병렬로 실행합니다.

`langchain_core.runnables` 모듈의 `RunnableParallel` 클래스를 사용하여 두 가지 작업을 병렬로 실행하는 예시를 보여줍니다.

`ChatPromptTemplate.from_template` 메서드를 사용하여 주어진 `country`에 대한 **수도** 와 **면적** 을 구하는 두 개의 체인(`chain1`, `chain2`)을 만듭니다.

이 체인들은 각각 `model`과 파이프(`|`) 연산자를 통해 연결됩니다. 마지막으로, `RunnableParallel` 클래스를 사용하여 이 두 체인을 `capital`와 `area`이라는 키로 결합하여 동시에 실행할 수 있는 `combined` 객체를 생성합니다.


```python
from langchain_core.runnables import RunnableParallel

# {country} 의 수도를 물어보는 체인을 생성합니다.
chain1 = (
    PromptTemplate.from_template("{country} 의 수도는 어디야?")
    | model
    | StrOutputParser()
)

# {country} 의 면적을 물어보는 체인을 생성합니다.
chain2 = (
    PromptTemplate.from_template("{country} 의 면적은 얼마야?")
    | model
    | StrOutputParser()
)

# 위의 2개 체인을 동시에 생성하는 병렬 실행 체인을 생성합니다.
combined = RunnableParallel(capital=chain1, area=chain2)
```
`chain1.invoke()` 함수는 `chain1` 객체의 `invoke` 메서드를 호출합니다.

이때, `country`이라는 키에 `대한민국`라는 값을 가진 딕셔너리를 인자로 전달합니다.


```python
# chain1 를 실행합니다.
chain1.invoke({"country": "대한민국"})
```
이번에는 `chain2.invoke()` 를 호출합니다. `country` 키에 다른 국가인 `미국` 을 전달합니다.


```python
# chain2 를 실행합니다.
chain2.invoke({"country": "미국"})
```
`combined` 객체의 `invoke` 메서드는 주어진 `country`에 대한 처리를 수행합니다.

이 예제에서는 `대한민국`라는 주제를 `invoke` 메서드에 전달하여 실행합니다.


```python
# 병렬 실행 체인을 실행합니다.
combined.invoke({"country": "대한민국"})
```
### 배치에서의 병렬 처리

병렬 처리는 다른 실행 가능한 코드와 결합될 수 있습니다.
배치와 병렬 처리를 사용해 보도록 합시다.

`chain1.batch` 함수는 여러 개의 딕셔너리를 포함하는 리스트를 인자로 받아, 각 딕셔너리에 있는 "topic" 키에 해당하는 값을 처리합니다. 이 예시에서는 "대한민국"와 "미국"라는 두 개의 토픽을 배치 처리하고 있습니다.


```python
# 배치 처리를 수행합니다.
chain1.batch([{"country": "대한민국"}, {"country": "미국"}])
```
`chain2.batch` 함수는 여러 개의 딕셔너리를 리스트 형태로 받아, 일괄 처리(batch)를 수행합니다.

이 예시에서는 `대한민국`와 `미국`라는 두 가지 국가에 대한 처리를 요청합니다.


```python
# 배치 처리를 수행합니다.
chain2.batch([{"country": "대한민국"}, {"country": "미국"}])
```
`combined.batch` 함수는 주어진 데이터를 배치로 처리하는 데 사용됩니다. 이 예시에서는 두 개의 딕셔너리 객체를 포함하는 리스트를 인자로 받아 각각 `대한민국`와 `미국` 두 나라에 대한 데이터를 배치 처리합니다.


```python
# 주어진 데이터를 배치로 처리합니다.
combined.batch([{"country": "대한민국"}, {"country": "미국"}])
```




