# MySQL upsert、 获取自增id

## upsert 语法

有则更新，没有则插入

```mysql
 insert into user_third(apk_type, third_type, third_id, third_name)
                    values ('BaseWallpaper', 'fb', 'fb-test', 'test')
                    ON DUPLICATE KEY UPDATE
                    third_name=values(third_name)
                
                
 replace into user_third(apk_type, third_type, third_id, third_name)
 values ('BaseWallpaper', 'fb', 'fb-test', 'test')
```

- 两种语法都会引起自增id的增加
- replace into会直接修改记录的id
- insert ON DUPLICATE KEY UPDATE 只是修改表的AUTO_INCREMENT的起始值，记录的id不会修改
- 他们都是根据唯一主键、唯一索引判断是否存在记录

## 自增id的获取

系统变量lastrowid：```returns the value generated for an `AUTO_INCREMENT` column by the previous [`INSERT`] or [`UPDATE`] statement or `None` when there is no such value available```

系统变量IDENTITY， 此变量是[last_insert_id](https://www.docs4dev.com/docs/zh/mysql/5.7/reference/server-system-variables.html#sysvar_host_cache_size)变量的同义词

以上两种语法都可获取到更新、插入操作之后的自增id；当执行单条sql的时候，并不会有什么区别；

**但是使用事务执行多条sql时，lastrowid返回第一条改变的id，IDENTITY返回最后一条**

```python
    async def __getInsertId(self, cursor, last_id):
        """
        单条sql更新时，用lastrowid取第一条变化的id；
        多条sql时，取最后一条id`SELECT @@IDENTITY AS id`
        :param cursor:
        :param last_id: FIRST|LAST
        :return:
        """
        lastrowid = cursor.lastrowid
        if last_id == ID_FIRST:
            return lastrowid
        else:
            await cursor.execute("SELECT @@IDENTITY AS id")
            result = await cursor.fetchall()
            result = result[0][0]
            if  isinstance(result, dict):
                return result['id']
            else:
                return result


    async def excute(self, sql, param=None, last_id=ID_FIRST):
        """
        执行单条sql
        :param sql:
        :param param:
        :param last_id:
        :return:
        """
        conn, cursor = await self.getCusor()
        try:
            if param is None:
                ret = await cursor.execute(sql)
            else:
                ret = await cursor.execute(sql, param)

            await conn.commit()

            if last_id:
                id = None
                if ret > 0:
                    id = await self.__getInsertId(cursor, last_id)

                await self.relaseConn(conn)
                return id
            else:
                await self.relaseConn(conn)
                return ret

        except Exception as e:
            _LOGGER.error(sql)
            _LOGGER.exception(e)


    async def excute_transaction(self, sqls, last_id=ID_LAST):
        """
        事务，多条sql一起执行
        :param sqls:
        :param last_id:
        :return:
        """
        conn, cursor = await self.getCusor()
        try:
            for sql in sqls:
                ret = await cursor.execute(sql)

            await conn.commit()

            if last_id:
                id = None
                if ret > 0:
                    id = await self.__getInsertId(cursor, last_id)

                await self.relaseConn(conn)
                return id
            else:
                await self.relaseConn(conn)
                return ret

        except Exception as e:
            _LOGGER.error(sql)
            _LOGGER.exception(e)


```

