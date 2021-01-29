发现每次用到Selenium的时候都会忘记用法，迫不得已需要再学一遍，因此这次准备好好做一下笔记，至少以后用到有个查的地方



```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

browser = webdriver.Chrome()

url = "http://baidu.com"
browser.get(url)
search_box = browser.find_element_by_id("kw")
search_box.send_keys('selenium'+Keys.Space+'python')

search_box.click()
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
# 通过这个接口可以执行JavaScript的代码

windows = browser.window_handles
# 标签页列表

```

