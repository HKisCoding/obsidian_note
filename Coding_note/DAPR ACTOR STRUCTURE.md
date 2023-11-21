## Backend function 

```python
FILE NAME: QuerySearching.py
async def getUnderstandingOrder(self, textOrder: str):
        """
        Get understanding order with text order input
        Args:
            - :param: `textOrder` (string): Text order.
        Returns:
            tuple: err, dict
        Raises:
            Exception, dict empty
        """
        err = ""
        response = dict()
        try:
            # Create proxy client
            proxy = ActorProxy.create('QueryUnderstandingActor', ActorId.create_random_id())
            payload_encode = textOrder.encode('utf-8')
            rtn_obj = await proxy.invoke_method("QueryUnderstanding", payload_encode)
            response = json.loads(rtn_obj)
        except DaprInternalError as exc:
            # Handle DaprException (includes DaprInternalError)
            err = exc
            msg = '%s.%s Error: %s - Line: %s' % (self.__class__.__name__, str(exc), stack()[0][3], exc.__traceback__.tb_lineno) # give a error message    
            logger.error(msg)
        except Exception as exc:
            # Handle other exceptions
            err = exc
            msg = '%s.%s Error: %s - Line: %s' % (self.__class__.__name__, str(exc), stack()[0][3], exc.__traceback__.tb_lineno) # give a error message
            logger.error(msg)
        finally:
            return err, response
```

## API function 

```python 
FILE NAME: actor.py

from inspect import stack

async def search(self, data) -> object:
        """Search with textOrder

        Returns:
            _type_: json
        """
       
        """An actor method which gets attr extraction from data."""
        objRespone = SPResponseModel()
        
        try:
            request = SearchingRequestModel(**data)
        except Exception as exc:
            err = '%s.%s Error: %s - Line: %s' % (self.__class__.__name__, str(exc), stack()[0][3], exc.__traceback__.tb_lineno) # give a error message
            logger.error(err)
            objRespone.code = ErrorCode.SEARCHING_BAD_REQUEST.get("code")
            objRespone.msg  = ErrorCode.SEARCHING_BAD_REQUEST.get("desc")
            objRespone.data = exc
            return json.dumps(objRespone.dict())
            
        err, dOrder = await self._searchEngine.getUnderstandingOrder(request.textOrder)
        
        if err != "":
            objRespone.code = ErrorCode.SEARCHING_QUERY_UNDERSTANDING_ERROR.get("code")
            objRespone.msg  = ErrorCode.SEARCHING_QUERY_UNDERSTANDING_ERROR.get("desc")
            objRespone.data = err
            return json.dumps(objRespone.dict())
        
        dOrder = {"store_name": "idmua"}
        err, lstStore = await self._searchEngine.query(esConnector, dOrder)
        
        if err != "":
            objRespone.code = ErrorCode.SEARCHING_QUERY_ELASTIC_ERROR.get("code")
            objRespone.msg  = ErrorCode.SEARCHING_QUERY_ELASTIC_ERROR.get("desc")
            objRespone.data = err
        else:
            objRespone.code = ErrorCode.SUCCESS.get("code")
            objRespone.msg  = ErrorCode.SUCCESS.get("desc")
            objRespone.data = lstStore
        
        return json.dumps(objRespone.dict())
```





