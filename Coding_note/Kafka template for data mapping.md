
### General step: 
- Define Data Entity Object for source and destination 
- Define Data transform function class 
- Create produce/consume based on template
### 1. Data transform
Object class define transform/mapping function 
``` python 
class EntityHookFunction(object):
    def __init__(self,orgin_mapper, logic_mapper) :
        self.orgin_mapper = orgin_mapper
        self.logic_mapper = logic_mapper
    def __call__(self, datadict ) :
        """
            datadict: data in dict format
            orgin_mapper: {'target_fields' : 'source_fields' }
            logic_mapper: {}
        """
        return datadict
```

