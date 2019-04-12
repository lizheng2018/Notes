Python常用操作

---
#### 1.Python去除字符串中指定字符

1. strip()

```python
>>> s = ' hello world \n'
# s.strip()参数为空时，默认去除s字符串中头尾\r, \t, \n, 空格等字符
>>> s.strip()
'hello world'
# s.lstrip()删除s字符串开头处的指定字符，s.rstrip()删除s结尾处的指定字符
>>> s.lstrip()
'hello world \n'
>>> s.rstrip()
' hello world'
>>> t = '-----hello====='
>>> t.lstrip('-')
'hello====='
>>> t.strip('-=')
'hello'
# replace()
>>> s.replace(' ', '')
'helloworld'
```



