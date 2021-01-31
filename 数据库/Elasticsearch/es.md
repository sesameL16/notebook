##### 1.  删除单个索引数据

post 请求

~~~
http://10.1.12.15:9200/person_idx/person_type/_delete_by_query?pretty

{
	"query":{
    	"match_all":{}
 	}
}
~~~

##### 2. 删除全部索引数据

post 请求

```
http://10.1.12.15:9200/_all/_delete_by_query?pretty

{
	"query":{
    	"match_all":{}
 	}
}
```

http://10.1.4.31:9200/

##### 3. 新建索引

```
http://10.1.12.15:9200
```

