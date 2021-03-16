需求是处理word文件里面的表格，处理之后写到excel里面；

处理word文件使用到的是这个库，docx；

安装

```bash
pip install python_docx
```

简单的demo

```python
from docx import Document

doc = Document(filename)
tb1 = doc.tables[0]
# 读取其中所有的表格

tb1.rows
# 表格1的行

cells = tb1.rows[0].cells
for cell in cells:
  print(cell.text)
  # 输出第一行的所有单元格
```

由于这个需求只需要读表格中的内容，所以比较简单，没有涉及到其他写、格式等等内容；



