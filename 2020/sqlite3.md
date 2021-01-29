## 简介

Sqlite工作主要的流程是

```python
import sqlite3

conn = sqlite3.connect('database.db')
# 创建连接
c = conn.cursor()
# 建立游标

c.excute('SQL语句')

conn.commit()
# 操作只有commit了才会被执行
conn.close()
# 断开连接
```



其中最主要的两个类就是Connection和Cursor

通过Connection和文件建立连接，Cursor作为一个游标来操作



这个模块中的其他比较有价值的函数

```python
sqlite3.complete_statement(sql)
```

这个函数可以检测输入的sql语句是否是一个完整的sql语句，但是这个方法并不会判断这个语句的语法是否正确，除了是否是用分号结尾的或是引号不闭合这样的情况

利用这个函数可以构造出一个shell来交互sqlite的文件

```python
# A minimal SQLite shell for experiments

import sqlite3

con = sqlite3.connect(":memory:")
con.isolation_level = None
cur = con.cursor()

buffer = ""

print "Enter your SQL commands to execute in sqlite3."
print "Enter a blank line to exit."

while True:
    line = raw_input()
    if line == "":
        break
    buffer += line
    if sqlite3.complete_statement(buffer):
        try:
            buffer = buffer.strip()
            cur.execute(buffer)

            if buffer.lstrip().upper().startswith("SELECT"):
                print cur.fetchall()
        except sqlite3.Error as e:
            print "An error occurred:", e.args[0]
        buffer = ""

con.close()
```

以上代码来自Sqlite的手册



