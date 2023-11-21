
## Construct project:
``` shell 
.
├── Dockerfile
├── Dockerfile.prod
├── README.md
├── config.yaml
├── docker-compose.yml
├── gunicorn
│   └── gunicorn_conf.py
├── project.md
├── requirements
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── scripts
│   ├── downgrade
│   ├── format
│   ├── makemigrations
│   ├── migrate
│   ├── start-dev.sh
│   └── start-prod.sh
└── src
    ├── __init__.py
    ├── api.py
    ├── auth
    │   ├── __init__.py
    │   ├── config.py
    │   ├── constants.py
    │   ├── dependencies.py
    │   ├── exceptions.py
    │   ├── schemas.py
    │   ├── security.py
    │   ├── service.py
    │   └── utils.py
    ├── config.py
    ├── constants.py
    ├── exceptions.py
    ├── main.py
    ├── models.py
    ├── rate_limiter.py
    ├── utils.py
    └── voice
        ├── __init__.py
        ├── models.py
        ├── router.py
        └── service.py

```

## Library:
- fastapi 
- pydantic 
- [slowapi](https://slowapi.readthedocs.io/en/latest/): A rate limiting library for Starlette and FastAPI adapted from [flask-limiter](http://github.com/alisaifee/flask-limiter).
- [starlette](https://www.starlette.io/): lightweight [ASGI](https://asgi.readthedocs.io/en/latest/) framework/toolkit, which is ideal for building async web services in Python.

## Design API 

### models.py 
Define Input and Output Object for Api using pydantic validation: 
**Response model:** 
- Response code 
- Response message 
- Data 

``` python 
from datetime import datetime
from typing import Any, Callable
import typing
from zoneinfo import ZoneInfo

from fastapi import status
import orjson
from pydantic import BaseModel, Field, validator, root_validator
from pydantic.types import conint, constr, SecretStr

# pydantic type that limits the range of primary keys
PrimaryKey = conint(gt=0, lt=2147483647)
NameStr = constr(regex=r"^(?!\s*$).+", strip_whitespace=True, min_length=3)
OrganizationSlug = constr(regex=r"^[\w]+(?:_[\w]+)*$", min_length=3)

def orjson_dumps(v: Any, *, default: Callable[[Any], Any] = None) -> str:
    return orjson.dumps(v, default=default).decode()

def convert_datetime_to_gmt(dt: datetime) -> str:
    if not dt.tzinfo:
        dt = dt.replace(tzinfo=ZoneInfo("UTC"))

    return dt.strftime("%Y-%m-%dT%H:%M:%S%z")


class ORJSONModel(BaseModel):
    class Config:
        json_loads = orjson.loads
        json_dumps = orjson_dumps
        json_encoders = {datetime: convert_datetime_to_gmt}
        allow_population_by_field_name = True

    @root_validator()
    def set_null_microseconds(cls, data: dict[str, Any]) -> dict[str, Any]:
        datetime_fields = {
            k: v.replace(microsecond=0)
            for k, v in data.items()
            if isinstance(k, datetime)
        }

        return {**data, **datetime_fields}

class SPResponseModel(ORJSONModel):
    code: int = status.HTTP_201_CREATED
    msg: str = None
    data: dict = {}
```

### config.py 
Config for api with env control, domain, api versioning 
``` python 
from typing import Any

from pydantic import BaseSettings, PostgresDsn, RedisDsn, root_validator

from .constants import Environment


class Config(BaseSettings):
    SITE_DOMAIN: str = "voice-api.smartpay.com.vn"
    ENVIRONMENT: Environment = Environment.PRODUCTION

    CORS_ORIGINS: list[str]
    CORS_ORIGINS_REGEX: str = None
    CORS_HEADERS: list[str]

    APP_VERSION: str = "1"


settings = Config()

app_configs: dict[str, Any] = {"title": "App API"}
if settings.ENVIRONMENT.is_deployed:
    app_configs["root_path"] = f"/v{settings.APP_VERSION}"

if not settings.ENVIRONMENT.is_debug:
    app_configs["openapi_url"] = None
```

### api.py 
Define common code for api status and routing api endpoint 

router.py: contain function for each endpoint 
``` python 
from fastapi import APIRouter, Depends
from .LoggingUtils import Logger


logger = Logger('Router API GPT')
from fastapi.logger import logger as fastapi_logger
fastapi_logger.handlers = logger.handlers

router = APIRouter()

@router.post("/order", response_model=SPResponseModel, dependencies=[Depends(valid_api_credential)])
async def get_order (orderReq: OrderModel) -> SPResponseModel:
    func_name="get_order"
    _ = func_name
    ...
    return objResponse
    
@router.post("/recommend_order", response_model=SPResponseModel, dependencies=[Depends(valid_api_credential)])
async def get_order(
        orderReq: OrderModel
    ) -> SPResponseModel:
    func_name="get_recommend_order"
    _ = func_name
    ...
    return objResponse

@router.post("/speech", response_model=SPResponseModel, dependencies=[Depends(valid_api_credential)])
async def speech_to_text (hertzReq: AudioHertzModel) -> SPResponseModel:
    func_name="speech_to_text"
    _ = func_name
    ... 
    return objResponse
```

api.py
``` python 
from typing import List, Optional
from fastapi import APIRouter, Depends
from pydantic import BaseModel
from starlette.responses import JSONResponse

from .router import router as voice_router

class ErrorMessage(BaseModel):
    msg: str

class ErrorResponse(BaseModel):
    detail: Optional[List[ErrorMessage]]

api_router = APIRouter(
    default_response_class=JSONResponse,
    responses={
        400: {"model": ErrorResponse},
        401: {"model": ErrorResponse},
        403: {"model": ErrorResponse},
        404: {"model": ErrorResponse},
        500: {"model": ErrorResponse},
    },
)

# WARNING: Don't use this unless you want unauthenticated routes
authenticated_api_router = APIRouter()

authenticated_api_router.include_router(
    voice_router, prefix="/voice", tags=["voice"]
)

@api_router.get("/healthcheck", include_in_schema=False)
def healthcheck():
    return {"status": "ok"}

api_router.include_router(
    authenticated_api_router,
)
```

