# Tools

## Jupyter Notebbok
### 设置chrome为默认浏览器

* 进入C:\Users\Administrator\.jupyter，打开jupyter_notebook_config.py文件
* 找到代码位置 # c.NotebookApp.browser = ''

* 在上述代码后添加以下代码：

```
import webbrowser

webbrowser.register("chrome",None,webbrowser.GenericBrowser(u"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"))

c.NotebookApp.browser = 'chrome'
```
