
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
# constants.py 

class Environment(str, Enum):
    LOCAL = "LOCAL"
    STAGING = "STAGING"
    TESTING = "TESTING"
    PRODUCTION = "PRODUCTION"

    @property
    def is_debug(self):
        return self in (self.LOCAL, self.STAGING, self.TESTING)

    @property
    def is_testing(self):
        return self == self.TESTING

    @property
    def is_deployed(self) -> bool:
        return self in (self.STAGING, self.PRODUCTION)
```

``` python 
# settings.py

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
# router.py

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
# api.py

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

### main.py
#### Create fastapi app 
Create the ASGI for the app 
``` python
# main.py 

from fastapi import FastAPI, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from fastapi.logger import logger as fastapi_logger

log = logging.getLogger(__name__)
fastapi_logger.handlers = log.handlers

exception_handlers = {404: not_found}
app = FastAPI(exception_handlers=exception_handlers, openapi_url="",
              max_body_size=200000000) 
```

#### Adding CORS 
> [!note]
> **CORS**: Cross-origin resource sharing 
> The situations when a frontend running in a browser has JavaScript code that communicates with a backend, and the backend is in a different "origin" than the frontend.
> ```
>Example: Same localhost but use differnet protocol and ports are different origin
> ```

``` python 
# main.py
from starlette.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_origin_regex=settings.CORS_ORIGINS_REGEX,
    allow_credentials=True,
    allow_methods=("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"),
    allow_headers=settings.CORS_HEADERS,
)

@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["Strict-Transport-Security"] = "max-age=31536000 ; includeSubDomains"
    return response
```

#### Create web api 
``` python 
# main.py 

api = FastAPI(
    title="Voice API",
    description="Welcome to Voice's API documentation! Here you will able to discover all of the ways you can interact with the Voice API.",
    root_path="/api/v1",
    docs_url=None,
    openapi_url="/docs/openapi.json",
    redoc_url="/docs",
    max_body_size=200000000,
)

# we add all API routes to the Web API framework
api.include_router(api_router)

# we mount api backend
app.mount("/api/v1", app=api)
```

#### Add api middleware
>[!note]
>**Middleware**: function that works with every **request** before it is processed by any specific _path operation_. And also with every **response** before returning it.

``` python 
# main.py

from starlette.middleware.base import BaseHTTPMiddleware, RequestResponseEndpoint
from starlette.requests import Request
from starlette.routing import compile_path

from starlette.responses import Response, StreamingResponse, FileResponse
from starlette.staticfiles import StaticFiles


def get_path_template(request: Request) -> str:
    if hasattr(request, "path"):
        return ",".join(request.path.split("/")[1:])
    return ".".join(request.url.path.split("/")[1:])

class MetricsMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
        path_template = get_path_template(request)

        method = request.method
        tags = {"method": method, "endpoint": path_template}

        try:
            start = time.perf_counter()
            response = await call_next(request)
            elapsed_time = time.perf_counter() - start
            tags.update({"status_code": response.status_code})
            fastapi_logger.debug(f"server.call.elapsed.{path_template}: {elapsed_time}")
        except Exception as e:
            raise e from None
        return response
    
class ExceptionMiddleware(BaseHTTPMiddleware):
    async def dispatch(
        self, request: Request, call_next: RequestResponseEndpoint
    ) -> StreamingResponse:
        try:
            response = await call_next(request)
        except ValidationError as e:
            fastapi_logger.exception(e)
            response = JSONResponse(
                status_code=status.HTTP_422_UNPROCESSABLE_ENTITY, content=custom_errors_handler(e),
            )
        except ValueError as e:
            fastapi_logger.exception(e)
            response = JSONResponse(
                status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
                content=custom_errors_handler(e),
            )
        except Exception as e:
            fastapi_logger.exception(e)
            response = JSONResponse(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                content=custom_errors_handler(e),
            )

        return response

# we add a middleware class for capturing metrics using Dispatch's metrics provider
api.add_middleware(MetricsMiddleware)
api.add_middleware(ExceptionMiddleware)
```

#### Adding handler exception
``` python 
# exceptions.py

from typing import Any

from fastapi import HTTPException, status
from pydantic.errors import PydanticValueError

import json 
from http import HTTPStatus
from pydantic.error_wrappers import ValidationError
from fastapi.responses import JSONResponse
from .models import SPResponseModel

from starlette.requests import Request

def custom_errors_handler(e: Exception) -> SPResponseModel:
    objResp = SPResponseModel()
    
    if isinstance(e, HTTPException):
        objResp.msg     = e.detail
        objResp.code    = e.status_code
    elif isinstance(e, ValidationError):
        objResp.msg = json.loads(e.json())
        objResp.code = HTTPStatus.BAD_REQUEST
    else:
        objResp.msg = ""
        objResp.code = HTTPStatus.INTERNAL_SERVER_ERROR
    return objResp.dict()

def custom_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(status_code=exc.status_code, content=custom_errors_handler(exc))
    
class DetailedHTTPException(HTTPException):
    STATUS_CODE = status.HTTP_500_INTERNAL_SERVER_ERROR
    DETAIL = "Server error"

    def __init__(self, **kwargs: dict[str, Any]) -> None:
        super().__init__(status_code=self.STATUS_CODE, detail=self.DETAIL, **kwargs)

class PermissionDenied(DetailedHTTPException):
    STATUS_CODE = status.HTTP_403_FORBIDDEN
    DETAIL = "Permission denied"


class NotFound(DetailedHTTPException):
    STATUS_CODE = status.HTTP_404_NOT_FOUND


class BadRequest(DetailedHTTPException):
    STATUS_CODE = status.HTTP_400_BAD_REQUEST
    DETAIL = "Bad Request"


class NotAuthenticated(DetailedHTTPException):
    STATUS_CODE = status.HTTP_401_UNAUTHORIZED
    DETAIL = "User not authenticated"

    def __init__(self) -> None:
        super().__init__(headers={"WWW-Authenticate": "Bearer"})

```

Authenticate exception
``` python 
# auth_exceptions.py

from python.sdk.api.src.auth.constants import ErrorCode
from python.sdk.api.src.exceptions import BadRequest, NotAuthenticated, PermissionDenied


class AuthRequired(NotAuthenticated):
    DETAIL = ErrorCode.AUTHENTICATION_REQUIRED

class AuthorizationFailed(PermissionDenied):
    DETAIL = ErrorCode.AUTHORIZATION_FAILED

class InvalidToken(NotAuthenticated):
    DETAIL = ErrorCode.INVALID_TOKEN

class RefreshTokenNotValid(NotAuthenticated):
    DETAIL = ErrorCode.REFRESH_TOKEN_NOT_VALID

class InvalidCredentials(BadRequest):
    DETAIL = ErrorCode.INVALID_CREDENTIALS

class InvalidSecretId(BadRequest):
    DETAIL = ErrorCode.SECRET_ID_INVALID

class InvalidSecretKey(BadRequest):
    DETAIL = ErrorCode.SECRET_KEY_INVALID
```

``` python 
# main.py

from python.sdk.api.src.auth.exceptions import (
    InvalidCredentials,
    InvalidSecretId,
    InvalidSecretKey,
    custom_errors_handler, 
    custom_exception_handler,
    DetailedHTTPException,
    PermissionDenied,
    NotFound,
    NotAuthenticated,
)

api.add_exception_handler(InvalidCredentials, custom_exception_handler)
api.add_exception_handler(InvalidSecretId, custom_exception_handler)
api.add_exception_handler(InvalidSecretKey, custom_exception_handler)
api.add_exception_handler(DetailedHTTPException, custom_exception_handler)
api.add_exception_handler(PermissionDenied, custom_exception_handler)
api.add_exception_handler(NotFound, custom_exception_handler)
api.add_exception_handler(NotAuthenticated, custom_exception_handler)

@api.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({"code": status.HTTP_422_UNPROCESSABLE_ENTITY, "msg": exc.errors(), "data": exc.body}),
    )
```
