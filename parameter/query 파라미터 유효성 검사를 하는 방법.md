## 🤔 유효성 검사란?

유효성 검사는 프로세스 또는 시스템의 맥락에서 개념이나 구성이 수용 가능한지 확인하는 프로세스입니다.  
예를 들어 데이터의 생성, 소비 및 조작을 중심으로하는 컴퓨터 시스템에서 오류가 발생하지 않도록 처리하기 전에 모든 데이터가 올바른지 확인하는 것이 중요합니다.  
입력 데이터가 시스템 요구 사항을 충족하는지 확인하기 위해 유효성 검사가 수행됩니다.

## 쿼리 매개변수 및 숫자형 유효성 검사

FastAPI는 추가적인 정보와 유효성 검사를 매개변수에 선언할 수 있게 해줍니다.

아래 애플리케이션 예시를 살펴보면

```python
from typing import Annotated
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(q : Annotated[str] = None):
    results = {"items":[{"item_id": "Foo"},{"item_id":"Bar"}]}
    if q:
        results.update({"q":q})
    return results
```

쿼리 매개변수 `q`는 `Annotated[str]` 자료형이고, 이는 곧 str 이면서 동시에 `None`이 될 수 있다는 걸 의미하며, 실제로, 기본 값이 `None`이게 됩니다.  
따라서 FastAPI는 이것이 필수로 요구되지 않는 다는 것을 알게 됩니다.

## 추가적인 유효성 검사

q가 선택 사항이지만, 값이 주어졌을 때 **그 길이가 50자를 초과하지 않도록** 강제할 것입니다.

**임포트 Query**
위 조건을 구현하기 위해서는 우선 fastapi로 부터 Query를 임포트해야 합니다.

```python
from typing import Optional
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
# 매개변수의 기본값을 사용하여, 다음과 같이 매개변수 max_length를 50으로 설정합니다.
async def read_items(q: Optional[str] = Query(None, max_length = 50)):
    results = {"items":[{"item_id": "Foo"},{"item_id":"Bar"}]}
    if q :
        results.update({"q":q})
    return results

```

기본 값 `None`을 `Query(None)`으로 대체했기 때문에, `Query`의 첫 번째 매개변수는  
기본 값을 정의한 것과 동일한 목적으로 사용됩니다.

따라서 다음과 같은 코드는

`q : Optional[str] = Query(None)`

다음과 동일하게, 매개변수를 선택 사항으로 만듭니다.

`q: Optional[str] = None`

그러나 쿼리 매개변수로 명시적이게 선언됩니다.

## query 파라미터 유효성 검사를 하는 방법 정리

FastAPI에서 query 파라미터의 유효성 검사를 하는 방법은 위 내용을 토대로 다음과 같습니다.

1. 필수 파리미터로 설정하기 : query 파리미터를 필수로 만들기 위해 '...'을 사용합니다.  
   값이 주어지지 않은 경우 FastAPI는 오류를 반환합니다.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(...)):
    return {"q": q}

```

2. 기본값 설정하기 : query 파라미터에 기본값을 설정하여 선택적으로 사용할 수 있도록 만들 수 있습니다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = None):
    return {"q": q}

```

3. 유효성 검사 : 'Query' 함수를 사용하여 query 파라미터의 유효성을 검사할 수 있습니다.  
   `Query` 함수는 여러 매개변수를 받아 검수 옵션을 설정할 수 있습니다.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(..., min_length=3, max_length=50)):
    return {"q": q}

```

위의 예시에서 '...'은 필수 파라미터임을 나타내며, `min_length` 와 `max_length`는 query 파라미터의 길이를 제한하는 옵션입니다.  
따라서, query 파라미터의 값이 지정된 범위를 벗어날 경우 FastAPI는 오류를 반환합니다.

이러한 방법을 사용하여 query 파라미터의 유효성을 검사하고 원하는 규칙을 적용할 수 있습니다.  
유효성 검사를 통해 잘못된 값이나 형식을 가진 query 파라미터 요청에 대해 적절한 오류 응답을 반환할 수 있습니다.
