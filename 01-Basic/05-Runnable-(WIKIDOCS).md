


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
    


```python
# .env 파일을 읽어서 환경변수로 설정
from dotenv import load_dotenv

# 토큰 정보로드
load_dotenv()
```

```python
# LangSmith 추적을 설정합니다. https://smith.langchain.com
# !pip install -qU langchain-teddynote
from langchain_teddynote import logging

# 프로젝트 이름을 입력합니다.
logging.langsmith("CH01-Basic")
```
## 데이터를 효과적으로 전달하는 방법

- `RunnablePassthrough` 는 입력을 변경하지 않거나 추가 키를 더하여 전달할 수 있습니다. 
- `RunnablePassthrough()` 가 단독으로 호출되면, 단순히 입력을 받아 그대로 전달합니다.
- `RunnablePassthrough.assign(...)` 방식으로 호출되면, 입력을 받아 assign 함수에 전달된 추가 인수를 추가합니다.
### RunnablePassthrough


```python
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI


# prompt 와 llm 을 생성합니다.
prompt = PromptTemplate.from_template("{num} 의 10배는?")
llm = ChatOpenAI(temperature=0)

# chain 을 생성합니다.
chain = prompt | llm
```
chain 을 `invoke()` 하여 실행할 때는 입력 데이터의 타입이 딕셔너리여야 합니다.

```python
# chain 을 실행합니다.
chain.invoke({"num": 5})
```
하지만, langchain 라이브러리가 업데이트 되면서 1개의 변수만 템플릿에 포함하고 있다면, 값만 전달하는 것도 가능합니다.

```python
# chain 을 실행합니다.
chain.invoke(5)
```
아래는 `RunnablePassthrough` 를 사용한 예제입니다.

`RunnablePassthrough` 는 `runnable` 객체이며, `runnable` 객체는 `invoke()` 메소드를 사용하여 별도 실행이 가능합니다.


```python
from langchain_core.runnables import RunnablePassthrough

# runnable
RunnablePassthrough().invoke({"num": 10})
```
아래는 `RunnablePassthrough` 로 체인을 구성하는 예제입니다.

```python
runnable_chain = {"num": RunnablePassthrough()} | prompt | ChatOpenAI()

# dict 값이 RunnablePassthrough() 로 변경되었습니다.
runnable_chain.invoke(10)
```
다음은 `RunnablePassthrough.assign()` 을 사용하는 경우와 비교한 결과입니다.


```python
RunnablePassthrough().invoke({"num": 1})
```
`RunnablePassthrough.assign()`

- 입력 값으로 들어온 값의 key/value 쌍과 새롭게 할당된 key/value 쌍을 합칩니다.

```python
# 입력 키: num, 할당(assign) 키: new_num
(RunnablePassthrough.assign(new_num=lambda x: x["num"] * 3)).invoke({"num": 1})
```
## RunnableParallel

```python
from langchain_core.runnables import RunnableParallel

# RunnableParallel 인스턴스를 생성합니다. 이 인스턴스는 여러 Runnable 인스턴스를 병렬로 실행할 수 있습니다.
runnable = RunnableParallel(
    # RunnablePassthrough 인스턴스를 'passed' 키워드 인자로 전달합니다. 이는 입력된 데이터를 그대로 통과시키는 역할을 합니다.
    passed=RunnablePassthrough(),
    # 'extra' 키워드 인자로 RunnablePassthrough.assign을 사용하여, 'mult' 람다 함수를 할당합니다. 이 함수는 입력된 딕셔너리의 'num' 키에 해당하는 값을 3배로 증가시킵니다.
    extra=RunnablePassthrough.assign(mult=lambda x: x["num"] * 3),
    # 'modified' 키워드 인자로 람다 함수를 전달합니다. 이 함수는 입력된 딕셔너리의 'num' 키에 해당하는 값에 1을 더합니다.
    modified=lambda x: x["num"] + 1,
)

# runnable 인스턴스에 {'num': 1} 딕셔너리를 입력으로 전달하여 invoke 메소드를 호출합니다.
runnable.invoke({"num": 1})
```
Chain 도 RunnableParallel 적용할 수 있습니다.


```python
chain1 = (
    {"country": RunnablePassthrough()}
    | PromptTemplate.from_template("{country} 의 수도는?")
    | ChatOpenAI()
)
chain2 = (
    {"country": RunnablePassthrough()}
    | PromptTemplate.from_template("{country} 의 면적은?")
    | ChatOpenAI()
)
```

```python
combined_chain = RunnableParallel(capital=chain1, area=chain2)
combined_chain.invoke("대한민국")
```
## RunnableLambda

RunnableLambda 를 사용하여 사용자 정의 함수를 맵핑할 수 있습니다.


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from datetime import datetime


def get_today(a):
    # 오늘 날짜를 가져오기
    return datetime.today().strftime("%b-%d")


# 오늘 날짜를 출력
get_today(None)
```

```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough

# prompt 와 llm 을 생성합니다.
prompt = PromptTemplate.from_template(
    "{today} 가 생일인 유명인 {n} 명을 나열하세요. 생년월일을 표기해 주세요."
)
llm = ChatOpenAI(temperature=0, model_name="gpt-4o")

# chain 을 생성합니다.
chain = (
    {"today": RunnableLambda(get_today), "n": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```

```python
# 출력
print(chain.invoke(3))
```
`itemgetter` 를 사용하여 특정 키를 추출합니다.

```python
from operator import itemgetter

from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
from langchain_openai import ChatOpenAI


# 문장의 길이를 반환하는 함수입니다.
def length_function(text):
    return len(text)


# 두 문장의 길이를 곱한 값을 반환하는 함수입니다.
def _multiple_length_function(text1, text2):
    return len(text1) * len(text2)


# _multiple_length_function 함수를 사용하여 두 문장의 길이를 곱한 값을 반환하는 함수입니다.
def multiple_length_function(_dict):
    return _multiple_length_function(_dict["text1"], _dict["text2"])


prompt = ChatPromptTemplate.from_template("{a} + {b} 는 무엇인가요?")
model = ChatOpenAI()

chain1 = prompt | model

chain = (
    {
        "a": itemgetter("word1") | RunnableLambda(length_function),
        "b": {"text1": itemgetter("word1"), "text2": itemgetter("word2")}
        | RunnableLambda(multiple_length_function),
    }
    | prompt
    | model
)
```

```python
chain.invoke({"word1": "hello", "word2": "world"})
```




