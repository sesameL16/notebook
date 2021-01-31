## neo4g基本操作

()表示节点，[]表示关系，{}表示节点或者关系的其他属性



添加节点
merge (n:组织 {name:"米斯特", uuid:"lkhjgntmriedfrtyhgjkfdcfnkgiedf"})
添加关系
match (a),(b) where a.uuid="lkhjgntmriedfrtyhgjkfdcfnkgiedf" and b.uuid="asdf6546asdf4556" merge (a)-[r:包含 {uuid:"asdfghjkloiuytgbnhyujmkijhgfdcse"}]->(b)



  

merge存在不会添加，create存在还会继续添加

新增节点
create (n:人员{name:"张三",uuid:"slkdf6546"})

新增关系
match (a),(b) where a.uuid='' and b.uuid='' create (a)-[r:认识]->(b)

查询 根据uuid和节点类型查询 ..2 表示查询2级内的所有关系
match path=(n:%s)-[*..2]-(b) where n.uuid='' return path

删除节点和关系
match (n) where n.uuid='%s' detach delete n

只删除关系
match(n)-[r]-(m) where n.uuid='%s' delete r

删除所有关系和节点
match(n) optional match(n)-[r]-() delete n,r



修改

match (n:人员) where n.uuid = '8993eb94669d4b8485f400448ec6bad2' set n.name = '李大伟'

查询节点

match (n:人员) where n.uuid = '8993eb94669d4b8485f400448ec6bad2' return n



match (n)-[r]->(m) where n.uuid='793c589f97024acfad2b313f21c89828' and (type(r) = '通联' or type(r) = '同行' or type(r) = '同组织') delete r



match  (n)-[r]->(m) where n.uuid = '8993eb94669d4b8485f400448ec6bad2' return n,r,m





match path=(n)-[*..2]-(b) where n.uuid='8993eb94669d4b8485f400448ec6bad2' return path



//恢复图谱

match (n)-[r]-(m) where n.uuid = '8bd6218fb0bb4b1cb6904802fef601b8' or n.uuid = '5ce55b000dfc4e45980a5bea682edb76' return n,r



match (n)-[r]-(m) where ID(n) in [4211,4212] return n,r



match (n:手机号码)-[r]-(m) where size((n)--())=1 and n.uuid='7b2272026752483cbbfa0c882e4dfd29' return n