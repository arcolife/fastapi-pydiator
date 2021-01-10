![example event parameter](https://github.com/ozgurkara/fastapi-pydiator/workflows/CI/badge.svg) [![Coverage Status](https://coveralls.io/repos/github/ozgurkara/fastapi-pydiator/badge.svg)](https://coveralls.io/github/ozgurkara/fastapi-pydiator)

# What is the purpose of this repository
This project is an example that how to implement FastAPI and the pydiator-core. You can see the detail of the pydiator-core on this link https://github.com/ozgurkara/pydiator-core 

# How to run app
`uvicorn main:app --reload`
or `docker-compose up`

# How to run Tests
`coverage run --source app/ -m pytest`

`coverage report -m`

`coverage html`


# What is the pydiator?
You can see details here https://github.com/ozgurkara/pydiator-core
![pydiator](https://raw.githubusercontent.com/ozgurkara/pydiator-core/master/assets/pydiator_flow.png)

This architecture;
* Testable
* Use case oriented
* Has aspect programming (Authorization, Validation, Cache, Logging, Tracer etc.) support
* Clean architecture
* SOLID principles
* Has publisher subscriber infrastructure

There are ready implementations;
* Redis cache
* Swagger (http://0.0.0.0:8080)
* Opentracing via Jaeger (http://0.0.0.0:16686/)
 

# How to add the new use case? 

**Add New Use Case** 
   ```python
    #resources/sample/get_sample_by_id.py

    class GetSampleByIdRequest(BaseRequest):
        def __init__(self, id: int):
            self.id = id

    class GetSampleByIdResponse(BaseResponse):
        def __init__(self, id: int, title: str):
            self.id = id
            self.title = title 

   class GetSampleByIdUseCase(BaseHandler):
        async def handle(self, req: GetSampleByIdRequest):
            # related codes are here such as business
            return GetSampleByIdResponse(id=req.id, title="hello pydiatr")    
   ```

**Register Use Case**
   ```python
    # utils/pydiator/pydiator_core_config.py set_up_pydiator 
    container.register_request(GetSampleByIdRequest, GetSampleByIdUseCase())
   ```
 
Calling Use Case;
   ```python
    await pydiator.send(GetSampleByIdRequest(id=1))
   ```
<hr>

# How to use the cache? 
The cache pipeline decides that put to cache or not via the request model. If the request model inherits from the BaseCacheable object, this use case response can be cacheable. 
<br>
If the cache already exists, the cache pipeline returns with cache data so, the use case is not called. Otherwise, the use case is called and the response of the use case is added to cache on the cache pipeline.

   ```python
    class GetTodoAllRequest(BaseModel, BaseRequest, BaseCacheable):
        # cache key.
        def get_cache_key(self) -> str:
            return type(self).__name__ # it is cache key
    
        # cache duration value as second
        def get_cache_duration(self) -> int: 
            return 600

        # cache location type
        def get_cache_type(self) -> CacheType:
            return CacheType.DISTRIBUTED
   ```

Requirements;

1- Must have a redis and should be set the below environment variables
    
    REDIS_HOST = 'redis ip'

2- Must be activated the below environment variables on the config for using the cache;
    
    DISTRIBUTED_CACHE_IS_ENABLED=True
    CACHE_PIPELINE_IS_ENABLED=True

# Tracing via Jaeger
Requirements;

1- Must have a jaeger server and should be set the below environment variables
    
    JAEGER_HOST = 'jaeger ip'
    JAEGER_PORT = 'jaeger port'

2- Must be activated the below environment variables on the config for using the jaeger;
    
    TRACER_IS_ENABLED=True

![pydiator](https://raw.githubusercontent.com/ozgurkara/fastapi-pydiator/master/docs/assets/jaeger_is_not_enabled.png)

3- If want to trace the handlers, should be activated the below environment variables on the config for using the jaeger. Otherwise, can just see the endpoint trace details.   

    CACHE_PIPELINE_IS_ENABLED=True 

![pydiator](https://raw.githubusercontent.com/ozgurkara/fastapi-pydiator/master/docs/assets/jaeger.png)


