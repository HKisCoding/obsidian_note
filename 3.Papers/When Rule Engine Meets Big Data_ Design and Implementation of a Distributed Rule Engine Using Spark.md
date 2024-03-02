
- **PublishYear**:: 2017 
- **Author**:: Jindou Zhang, Jinxing Yang, Jing Li
- **Link**:: http://ieeexplore.ieee.org/document/7944919/
- **Tags**:: #paper
- **Cite Key**:: [@zhangWhenRuleEngine2017]

### Abstract
```
Rule Engines have been widely used both in industry and academia since they can separate rule knowledge from implementation logic conveniently and flexibly. However, traditional rule engine systems can not deal with big data, because of the limitations of memory and computing capacity of one single computer. Consequently, some researchers have proposed distributed rule engine to meet this challenge. But these solutions still do not work well when enormous amounts of facts are involved, for the reasons such as high cost of moving data, data imbalance and so on. We present the SparkRE system, a distributed rule engine based on Spark, to support big data reasoning. In particular, SparkRE uses DataFrame representing distributed working memory which holds enormous amounts of facts, and implements the rule condition-testing mechanism by the query execution engine of Spark SQL. In addition, we design a rule language tailored for SparkRE and an efficient inference engine. An experimental evaluation shows SparkRE’s capability of matching 26 million facts, and reveals its scalability and performance.
```

### Notes

Main components of Rule Engine:
- Working mem holding a set of Object representing facts: 
	- One object contains some attribute-value pairs (working memory element)
- Production mem contain set of rules
- Inference engine

Rete Algorithms: 
- Build a discrimination network by compiling rules in production memory 
- Flows the fact into the network to matching rule 

One cycle of rule engine:
- Matches the current content of WM against the LHS of all rules in product memory to find out satisfied rules
- Select one rule (conflict resolution)
- fire the selected rule (action)
- repeated until a fixed point is reach: No WM change or a halt action is fired 

Conditions of Rule (Left-hand side): structured as a conjunction of condition elements
Actions of Rule (Right-hand side): specified the consequent actions -> update, delete, add data for the Working memory.

Step in Inference Engine: 
- Matches the current contents of WM agaisnts the LHS of all rules in PM 
- Select one rule from satisfied rules
- Fire selected rule
![[Pasted image 20240104103810.png]]

Representing Fact in DataFrame:
- DataFrame is viewed as a distributed collection of rows with the same schema and can be stored in distributed mem evenly 
- DataFrame can be easily constructed from external source 
- DataFrame can be stored and processed in columnar way 

Representing Rule in SparkRE rule language:
Rule syntax: 
![[Pasted image 20240104111644.png]]
- **Rule attributes:** 
	- tell the inference engine how to process the rule -> **priority** attribute to resolving confliction 
	- Easy to extend new demand by adding new attribute in rule attributes
- **if statement:**
	- Each condition in conditions is matched with working memory elements and binds the elements matched to variables
	- Patterns of a condition are wrapped up sent to SparkQL for analysis 
	- Can provide customized matching operations through user defined function in SparkQL
- **Rule actions:**
	- Insert: take new objects and FactType class as input and convert the objects to a new DataFrame object  
	- Update: 
		- Input: binding variables defined in conditions, name of attributes, new value of the attributes 
		- Remove the value from existing elements and add new value to existing element 
	- Delete 
	- Save 
	- Abort 

Conditions can be present in a diagram:
![[Pasted image 20240104111726.png]]

Inference Algorithms:
- General approach:
	- Conflict resolution: sort the rule -> split the sorted into arrays by the criteria whether the rules has changeable action -> last rule of each array should have changeable action -> store in matching array sequentially 
	- Match: Get the element from the matching array sequentially -> evaluate the LHS (wrap the patterns of each condition into SparkQL query and bind to variables) -> store the matched rules in a matched set -> return the set to be executed 
	- Action: Consume rules in the matched sets one by one 
	- Cycle 
- Optimize approach:
	- Record the fact types of changed WME while firing rules -> compare the fact types of changed WME with rule's related fact types before evaluating the rule 
	- Evaluate rules concurrently by multi threading: Each rule in one element of the matching array spawns one thread to run the procedure of LHS matching 

Compare with Drools:
- SparkRE can go through big dataset and complex rule sets while Drools crash due to out of memory 
- SparkRE is slower 

---

