## SQLAlchemy

SQLAlchemy는 파이썬의 ORM(Object-Relational Mapping) 라이브러리로, 데이터베이스와 상호작용을 추상화하고 파이썬 객체로 데이터베이스를 다루 수 있게 해줍니다.

1. 설치

```lua
pip install sqlalchemy
```

2.  모델 정의
    SQLAlchemy를 사용하려면 먼저 데이터베이스에 매핑할 모델(테이블)을 정의해야 합니다. 모델은 declarative_base를 상속하는 클래스로 정의됩니다.

    ```python
    from sqlalchemy import Column, Integer, String
    from sqlalchemy.ext.declarative import declarative_base

     Base = declarative_base()

     class User(Base):
     __tablename__ = 'users'

     id = Column(Integer, primary_key=True, index=True)
     username = Column(String, unique=True, index=True)
     email = Column(String, unique=True, index=True)
    ```

3.  연결 생성:
    데이터베이스와의 연결을 생성합니다. SQLAlchemy는 다양한 데이터베이스 시스템과 호환되므로 적절한 드라이버와 연결 URL을 사용하여 데이터베이스와 연결합니다.

    ```python
    from sqlalchemy import create_engine

     DATABASE_URL = "sqlite:///./test.db" # SQLite 사용 예시
     engine = create_engine(DATABASE_URL)
    ```

4.  세션 생성:
    세션은 SQLAlchemy를 통해 데이터베이스와의 상호작용을 처리합니다. 세션을 생성하기 위해 sessionmaker를 사용합니다.

    ```python
    Copy code
    from sqlalchemy.orm import sessionmaker

    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
    ```

5.  CRUD 작업:
    세션을 사용하여 데이터베이스의 CRUD(Create, Read, Update, Delete) 작업을 수행합니다.

        ```python
        Copy code
        def create_user(db: Session, user: UserCreate):
        db_user = User(\*\*user.dict())
        db.add(db_user)
        db.commit()
        db.refresh(db_user)
        return db_user

        def get_user(db: Session, user_id: int):
        return db.query(User).filter(User.id == user_id).first()
        ```

위의 단계를 따라서 SQLAlchemy를 사용하여 데이터베이스를 다룰 수 있습니다.  
이는 간단한 예시로, 실제 응용 프로그램에서는 데이터베이스 관련 설정, 모델 정의, 세션 관리 등을 더 상세하게 구성해야 합니다.
SQLAlchemy의 공식 문서와 튜토리얼을 참고하면 더 많은 정보를 얻을 수 있습니다.
