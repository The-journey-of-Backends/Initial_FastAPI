## FastAPI에서 DB 세션을 사용하는 방법

이번 포스팅은 FastAPI에서 DB 세션을 사용하는 방법에 대해서 정리하려고 합니다.
우션 DB 세션을 사용하는 방법에 대해서 알아보기 전에 DB(데이터베이스) Session(세션)이란?</span>
데이터베이스와의 연결을 관리하고 데이터베이스 작업을 수행하는 단위입니다.
세션은 데이터베이스에 대한 일련의 작업을 하나의 트랜잭션으로 묶어주는 역할을 합니다.

여러 개의 데이터베이스 작업이 하나의 세션 내에서 이루어질 때, 이러한 작업은 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)을 보장받습니다.
이를 ACID 특성이라고 합니다.

세션은 데이터베이스와의 연결을 유지하며, 하나의 세션 내에서 여러 개의 쿼리와 데이터 조작 작업을 실행할 수 있습니다.
세션을 통해 데이터베이스 작업을 집중적으로 수행하고, 연결 및 해제에 따른 오버헤드를 최소화할 수 있습니다.

또한 세션은 웹서버에 웹 컨테이너의 상태를 유지하기 위한 정보를 저장합니다.
브라우저를 닫거나 서버에서 세션을 삭제하면 세션이 삭제됩니다.
세션은 각 클라이언트의 고유 세션 ID를 부여하는데, 이것으로 클라이언트를 구분하여 각 클라이언트의 요구에 맞는 응답을 반환합니다.

**세션의 주요 개념 및 기능**

1. 연결 관리 : 세션은 데이터베이스 연결을 유지하며, 세션 객체를 통해 데이터베이스 작업을 실행합니다.

2. 트랜잭션 관리 : 세션 내에서 수행되는 작업은 하나의 트랜잭션으로 묶일 수 있습니다. 이로써 모든 작업이 성공적으로 완료되면 데이터베이스에 반영되며, 실패하면 롤백 될 수 있습니다.

3. 쿼리 실행 및 결과 처리 : 세션을 통해 데이터베이스에 쿼리를 전달하고, 그 결과를 받아올 수 있습니다.

4. 스카폴딩과 인스턴스 관리 : ORM을 사용하는 경우, 세션은 데이터베이스 테이블과 매핑된 객체를 생성하고 관리하는 역할을 합니다.

5. 격리 수준 관리 : 데이터베이스 트랜잭션의 격리 수준을 관리하며, 동시에 여러 트랜잭션이 동작할 때 일관성과 격리성을 보장합니다.

6. 세션 상태 관리 : 세션 내의 객체의 변경 사항을 추적하고, 트랜잭션이 커밋되거나 롤백 될 때 변경 사항을 데이터베이스에 반영하거나 취소합니다.

## 🤔 FastAPI에선 어떻게 사용하는 가?

FastAPI에서 DB 세션을 사용하려면 SQLAlchemy를 통해 DB 세션을 관리하는 것이 일반적입니다.

1. SQLAIchemy 설정 및 모델 생성
   - SQLAlchemy를 사용하기 위해 모델을 정의하고 데이터베이스 설정을 구성합니다.
2. 의존성 주입
   - FastAPI는 의존성 주입(Dependency Injection)을 통해 필요한 의존성을 관리합니다.
   - DB 세션을 사용하기 위해 의존성을 만들고 설정합니다.
3. DB 세션 생성 및 종료
   - 의존성으로 주입받은 DB 세션을 사용하여 데이터베이스 작업을 처리합니다.
   - 필요한 경우에는 세션을 생성하고 작업이 끝나면 세션을 종료해야 합니다.

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session

# FastAPI 초기화
app = FastAPI()

# SQLAlchemy 설정
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 모델 정의
Base = declarative_base()

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String)

# 의존성을 통한 DB 세션 관리
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# API 라우트에서 DB 세션 사용
@app.post("/items/")
async def create_item(item: Item, db: Session = Depends(get_db)):
    db.add(item)
    db.commit()
    db.refresh(item)
    return item

@app.get("/items/{item_id}")
async def read_item(item_id: int, db: Session = Depends(get_db)):
    item = db.query(Item).filter(Item.id == item_id).first()
    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

```

위의 예제에서는 SQLAIchemy를 사용하여 SQLite 데이터베이스와 상호작용하고 있습니다.  
FastAPI의 `Depends`를 통해 으존성을 주입을 받아 DB 세션을 사용합니다.  
API 라우트에서 'db' 파라미터로 DB 세션을 사용하여 데이터베이스 작업을 수행하고 필요한 경우 세션을 종료합니다.
