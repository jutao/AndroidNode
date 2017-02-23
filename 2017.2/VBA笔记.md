#if 语句

##  if—then—else 语句#
这个语句相当于中文里面的“如果......那么......否则......”句式。

解释：如果逻辑表达式的结果为 true，则执行 Then 下的语句，如果逻辑表达式的结果为 False， 则执行 Else 下的语句。

	  If [a1] = "" Then
	       MsgBox "A1单元格没有输入任何内容！"
	  Else
	       MsgBox "A1单元格已经输入了内容！"
	  End If

###运行效果如下

单元格没有内容：

![单元格没有内容](http://i.imgur.com/UANmhrJ.png)

单元格有内容：

![单元格有内容](http://i.imgur.com/qRcxx1J.png)

## if—then—elseif语句 
要判断A1单元格的数是否能被2、3、5其中之一整除，设计程序：

        If [a1] = "" Then
            MsgBox "A1单元格没有输入任何内容！"
        ElseIf [a1] Mod 2 = 0 Then
            MsgBox "A1单元格能被2整除！"
        ElseIf [a1] Mod 3 = 0 Then
            MsgBox "A1单元格能被3整除！"
        ElseIf [a1] Mod 5 = 0 Then
            MsgBox "A1单元格能被5整除！"
        End If
若 A1 输入 16，运行结果如下：

![值16运行结果](http://i.imgur.com/ggGY5Sg.png)

# Go to 语句 #
Go to 语句是将程序转到指定的标签的语句位置，然后继续往下执行。Go to 语句通常用来作错误处理。

下面我们用 Go to 语句来做一个1——100自然数的和：

	Dim lSum As Long,i As Long
	i=1
	x: '为go to 语句设置的标签，必须以英文状态下的冒号结尾
	 lSum=lSum+i
	 i=i+1
	If i<=100 
		Then Goto x '如果i<=100,则程序跳到标签X处
	MsgBox "1到100的自然数和为"&lSum

运行结果：
	
![1到100相加结果](http://i.imgur.com/ERGBim2.png)

# 快捷键 #
ALT+F11 ：打开Visual Basic编辑器

CRTL+G  ：打开立即窗口

选中相应代码，点击 F1 查看帮助

#常用基础

##选择多个单元格

给 A1 到 D10 的所有单元格赋值：

	Range("A1:D10")=66

或者直接：

	[A1:D10]=99

##选择单个单元格
给第一行第十列的单元格赋值：

	cells(1,10)="习大大万岁"

##工作簿的操作
输出当前表格第一个工作簿的名称：

	Msgbox Worksheets(1).name
	
输出当前表格名为 Sheet1 的工作簿的名称

	Msgbox Worksheets("Sheet1").name
##选中单元格
	[D1:F8].Select

##一维数组定义
	Dim myarr(5) As Integer

##二维数组定义
	Dim myarr(1 to 5,1 to 10) As Integer
