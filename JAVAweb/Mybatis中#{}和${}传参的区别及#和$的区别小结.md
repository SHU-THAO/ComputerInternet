# mybatis中#{}和${}传参的区别

## **#{}** 

使用#{}意味着使用的预编译的语句，即在使用jdbc时的preparedStatement，sql语句中如果存在参数则会使用?作占位符，我们知道这种方式可以防止sql注入，并且在使用#{}时形成的sql语句，已经带有引号，例，select  * from table1 where id=#{id}  在调用这个语句时我们可以通过后台看到打印出的sql为：select * from table1 where id='2'   .**也就是说在组成sql语句的时候给参数加了单引号，这样即便是传入  or 1=1# 也会作为一个字符串参数处理**

## **${}**

使用${}时的sql不会当做字符串处理，是什么就是什么，如上边的语句：select * from table1 where id=${id} 在调用这个语句时控制台打印的为：select * from table1 where id=2 ，假设传的参数值为2

从上边的介绍可以看出这两种方式的区别，我们最好是能用#{}则用它，因为它可以防止sql注入，且是预编译的，在需要原样输出时才使用${}，如，select * from ${tableName} order by ${id} 这里需要传入表名和按照哪个列进行排序 ，加入传入table1、id 则语句为：select * from table1 order by id

如果是使用#{} 则变成了select * from 'table1' order by 'id' 我们知道这样就不对了。