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

### 设置主题、字体等

```python
pip install jupyterthemes

jt -t onedork -fs 14 -cellw 80% -ofs 10 -dfs 10 -T
# -cellw(占屏比或宽度) -ofs(输出段的字号) -dfs(pandas字体大小)-T(显示工具栏)
```



### 快捷工具

* 在Anaconda Prompt中运行以下命令：
```python
pip install jupyter_contrib_nbextensions && jupyter contrib nbextension install
```
* 打开 notebook，单击「edit」，然后点「nbextensions config」  
- Table of Contents：导航插件
- variable inspector：跟踪你的变量空间  
variable inspector 会显示你在 notebook 中创建的所有变量的名称，以及它们的类型、大小、形状和值
- ExecuteTime：显示单元格的运行时间和耗时
- Collapsible headings  
这个扩展在大型Notebook中非常有用，可折叠的标题能帮你收起/放下Notebook中的某些内容，使整个页面看起来更干净整洁
- autoreload：无需退出Jupyter Notebook就能动态修改代码. (不是插件)  
它的具体操作是：
```
%load_ext autoreload
%autoreload 2
```
