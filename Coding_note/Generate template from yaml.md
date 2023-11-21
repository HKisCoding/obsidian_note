### Yaml design: prefix + name 
```yaml 
promptingorder_fields: ['product_name', 'store_name', 'category', 'store_address', 'store_province', 'store_district', 'store_ward', 
                      'product_color', 'product_brand', 'product_size', 'product_weight', 'product_features']

promptingproduct_fields: ['product_name', 'category', 'produc_brand', 'product_price','product_color','product_size','product_weight']
searchdefault_fields: {'category': 'FMCG'}

# YUGABYTE;
yugabytemm_fields: ['mid', 'store_name', 'store_province', 'store_district', 'store_ward', 'store_full_address', 'store_gps_lat', 'store_gps_long', 'record_created_date']
yugabyteprod_fields: ['prd_id', 'product_name', 'category_name', 'brand', 'price', 'mid']
# elssearchprimary_fields: ['product_name','category_name','brand','color','size','total_weight']

template_elssearchmapping: 'python/app/knot/config/els_store_template.yaml'
template_knnquery: 'python/app/knot/config/knn_query_template.tpl'
template_geofilter: 'python/app/knot/config/geo_filter_template.tpl'

# EMBEDDING API
embedding_column: 'product_name'
embedding_api: 'openai'
embedding_object: 'product'

# USER UNDERSTANDING API
understand_system_prompt: ''
understand_user_prompt: ''
understand_api: 'openai'

# KNN 
elssearch_query:
  limit_distance: "2km"
  k: 10
  num_candidates: 5000
  batch_size: 2000
```

### Create embedding object 
#### 1. Create Dataclass 
``` python 
@dataclass
class EmbeddingActorClass(object):
    pass 
```

#### 2. Load entities from YAML
```python 
class LoadEntityFromYAMLFile():
    
    def __init__(self, fileName:str, dataType:str ) -> None:
        assert os.path.splitext(fileName)[1] == ".yaml" , "Just load YAML file"

        self.schema = Utilities.loadYAMLConfig(fileName)
        self.dataType = dataType

    def generateSingleRecord(self,classInstance: object, data:dict) -> object:
        for item in self.schema:
            prefix = item.split("_")
            if prefix[0] == self.dataType:
                name = '_' . join(prefix[1:])
                setattr(classInstance,name,self.schema[item])
                if isinstance(self.schema[item], list):
                    for citem in self.schema[item]:
                        cname = citem
                        cname_value = data.get(citem,'' )
                        setattr(classInstance,cname,cname_value)
                elif isinstance(self.schema[item],dict): 
                    for k,v in self.schema[item].items():
                        cname = k
                        setattr(classInstance,cname,v)

        return classInstance
```

```python 
embeddingLoader = LoadEntityFromYAMLFile(init_file,'embedding')
```

#### 3. Build entities object 
```python 
embeddingObj = embeddingLoader.generateSingleRecord(embeddingIns, {})
```
