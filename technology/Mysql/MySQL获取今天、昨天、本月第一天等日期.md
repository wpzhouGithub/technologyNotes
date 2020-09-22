```mysql
select curdate();                       -- 获取当前日期
select DATE_SUB(curdate(),INTERVAL 1 DAY);     -- 获取昨天日期
select last_day(curdate());                    -- 获取当月最后一天。
select DATE_ADD(curdate(),interval -day(curdate())+1 day);   -- 获取本月第一天
select date_add(curdate()-day(curdate())+1,interval 1 month); -- 获取下个月的第一天
select DATEDIFF(date_add(curdate()-day(curdate())+1,interval 1 month ),DATE_ADD(curdate(),interval -day(curdate())+1 day)) from dual; -- 获取当前月的天数
select date_add(curdate()-day(curdate())+1,interval -1 month); -- 获取上月第一天
select last_day(date_add(curdate()-day(curdate())+1,interval -1 month)) -- 获取上月最后一天; 
```

