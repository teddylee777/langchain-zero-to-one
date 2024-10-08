


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
    

## OpenAI API 키 발급 및 설정

1. OpenAI API 키 발급

- [OpenAI API 키 발급방법](https://teddylee777.github.io/openai/openai-api-key/) 글을 참고해 주세요.

2. `.env` 파일 설정

- 프로젝트 루트 디렉토리에 `.env` 파일을 생성합니다.
- 파일에 API 키를 다음 형식으로 저장합니다:
  `OPENAI_API_KEY` 에 발급받은 API KEY 를 입력합니다.

- `.env` 파일에 발급한 API KEY 를 입력합니다.


```python
# LangChain 업데이트
# !pip install -r https://raw.githubusercontent.com/teddylee777/langchain-kr/main/requirements.txt
```

```python
# API KEY를 환경변수로 관리하기 위한 설정 파일
from dotenv import load_dotenv

# API KEY 정보로드
load_dotenv()
```
API Key 가 잘 설정되었는지 확인합니다.


```python
import os

print(f"[API KEY]\n{os.environ['OPENAI_API_KEY'][:-15]}" + "*" * 15)
```




