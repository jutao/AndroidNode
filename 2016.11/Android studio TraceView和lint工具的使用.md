# Lint工具的使用 #
> Android lint工具是Android studio中集成的一个代码提示工具，它主要负责对你的代码进行优化提示，包括xml和Java文件，很强大。
> 编写完代码及时进行lint测试，会让我们的代码变得非常规范而且避免代码冗余。让我们及时发现代码中隐藏的问题。
> 举个例子：我们在代码中建立全局变量，而这个变量实际并不需要全局便利，lint在检测之后会提示我们改成局部变量，这对内存优化是一个很强大的促进手段。
## 用法 ##

![Lint用法](http://i.imgur.com/f5J0kuT.png)

点击 Inspect Code

![](http://i.imgur.com/5cs3B9v.png)

点击 OK

检查结果如下：

![](http://i.imgur.com/unumZy0.png)

非常强大，会给出我们各种修改意见，有的还真能学到不少东西。就比如说我这里的建议：用 merge 标签替换 FrameLayout。（[merge标签的使用](http://www.tuicool.com/articles/jyyUV33)）


[具体步骤](http://blog.csdn.net/qq_16131393/article/details/51172488 )







