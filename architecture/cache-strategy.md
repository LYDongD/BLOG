## cache strategy

### category

> read & write strategy 

[reference](https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/)

```
| cache-aside 
	| cache process
		| read cache  
		| cache miss ->applicaiton read database -> update cache -> read
	| cache scene
		| read-heavy workloads
| Read-through
	| cache process
		| read cache
		| cache miss -> cache provider read database -> update cache -> return  
	| cache scene
		| read-heavy workloads
| write
	| write-through
	| write-around
	| write-back

```
