### Pycharm自用快捷键

* 自动缩进代码

    先全选，Ctrl+A

    Code -> Auto-Indent Lines
    快捷键：Ctrl + Alt + I

* 执行选中的代码

    execute selection in console：shift + enter（自己修改的）

    execute cell in console：ctrl + enter

* 编辑时覆盖后面的代码

    插入模式，按insert键取消

* 不能编辑代码

    按ctrl+alt+v

* 多行缩进代码

    - 向右

        选中多行代码后，按下Tab键，缩进四个字符

    - 向左

        选中多行代码后，同时按住shift+Tab键，左移四个字符



### Pycharm中import时无法识别自己写的程序

我们用pycharm打开自己写的代码，当多个文件之间有相互依赖的关系的时候，import无法识别自己写的文件，但是我们写的文件又确实在同一个文件夹中，这种问题可以用下面的方法解决：

* 打开File--》Setting—》打开 Console下的Python Console，把选项（Add source roots to PYTHONPAT）点击勾选上

* 右键点击自己的工作空间，找下面的Mark Directory as 选择Source Root，就可以解决上面的问题了

* from 文件夹名 import 模块名