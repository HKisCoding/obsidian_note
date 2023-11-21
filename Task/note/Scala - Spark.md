![[Pasted image 20230207100027.png]]

## Scala basis
### Scala collection 
1. Array 

| Array Operation | Purpose                        | Performance      |
| --------------- | ------------------------------ | ---------------- |
| head            | Get first element              | O(1) TIME        |
| tail            | Get elements other than head   | O(n) linear time |
| apply           | Get element in specified index | O(1) Time        |
| update          | Replace element at index       | O(1) Time        |
| prepend         | Add new element on head        | Not supported    |
| append          | Add new element at the end     | Not supported    |
| insert          | Add new element at index       | Not supported    |

2. List 

| List Operation | Purpose                        | Performance      |
| -------------- | ------------------------------ | ---------------- |
| head           | Get first element              | O(1) TIME        |
| tail           | Get elements other than head   | O(1) Time        |
| apply          | Get element in specified index | O(n) Time        |
| update         | Replace element at index       | O(n) Time        |
| prepend        | Add new element on head        | O(1) Time        |
| append         | Add new element at the end     | O(n) linear Time |
| insert         | Add new element at index       | Not supported    |


3. Map
Map provides a mapping from a key to the associated value. Lookups into a Map based on a key have a generally constant time of O(1).

| Map type      | Description               | lookup             | add                | remove             | min    |
| ------------- | ------------------------- | ------------------ | ------------------ | ------------------ | ------ |
| Hashmap       | Backed by a hash table    | Near constant time | Near constant time | Near constant time | linear |
| Treemap       | Backed by a sorted tree   | logn               | logn               | logn               | logn   |
| linkedHashMap | Insert order is preserved | Linear             | Constant time      | Linear             | Linear |


