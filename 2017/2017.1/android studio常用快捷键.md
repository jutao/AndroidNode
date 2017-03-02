#最重要的快捷键
    1. ctrl+shift+A:万能命令行
    2. shift两次:查看资源文件
    3. ctrl+alt+s

#新建工程第一步操作
    1. module设置把空包分层去掉,compact empty middle package
    2. 设置当前的工程是utf-8,设置的Editor-->File Encodings-->全部改成utf-8,
#注释
    1. ctrl+/:单行注释
    2. ctrl+shift+/:多行注释



#光标操作
    1. ctrl+alt+enter:向上插入
    2. shift+enter:向下插入
    3. end:光标
#操作代码
    1. ctrl+d:复制粘贴一行
    2. ctrl+y:删除一行
#    shift+F6:重命名 #
    4. ctrl+alt+shift+c:类全名复制
    5. ctrl+O:复写代码

#格式代码及其他功能
    1. ctrl+alt+L:格式代码
    2. 在代码中使用alt+insert:Generate,可以get/set等操作
    3. ctrl+alt+T:添加try/catch
    4. ctrl+alt+M:抽取代码
    5. ctrl+alt+F:变量抽取全局变量
        1. 还需要设置前缀:Editor-->code style-->java-->code Genertion-->设置Field的前缘为m添加
    6. ctrl+alt+v:方法体内值抽取成变量
    7. ctrl+alt+p:把方法体内的变量抽取成方法内的参数
    8. 保存成模板:ctrl+shift+L,这个是自定义的(save as live Template)
    9. 选中内容:tab进行退格
    10. shift+tab:反向退格
    11. alt+shift+上下键:选中代码移动
    12. ctrl+shift+上下键:可以移动当前方法体,如果移动一行代码只能在代码体内移动
    13. ctrl+shift+U:代码大小写
    14. ctrl+alt+c:抽取成全局静态常量
    15. ctrl+shift+enter:补全代码(一行尾添加分号,如果是if等添加括号)

#进入代码
    1. ctrl+鼠标:进入代码
    2. ctrl+B:进入代码
    3. ctrl+alt+home:查看布局与对应的类
#图形操作
    1. xml显示布局时双击添加id
    2. 在xml布局时使用preview视图
    3. 在代码编辑时使用:
        1. ctrl+shift+a:输入split选择分屏

    4. 在xml布局中使用 : alt+shift+左右:切换布局视图
    5. ctrl+shift+12:最大化窗口
#替换查找
    1. ctrl+r:替换
    2. ctrl+F:查找
    3. ctrl+shift+F:全局查找
    4. ctrl+shift+R:全局替换
    5. ctrl+shift+i:快捷查看方法实现的内容
    6. ctrl+p:查看参数
    7. ctrl+Q:查看文档描述
    8. shift+F1:查看api文档
    8. ctrl+F12:查看类的方法
    9. ctrl+H:查看类的继承关系
    10. 查看变量的赋值情况:
        1. shift+ctrl+a:输入analyze data flow to Here
    11. ctrl+alt+H:查看方法在那里被调用了
    12. ctrl+{}:可以定位方法体的括号
    13. F3:查看选中的内容
    14. shift+F3:反向查看内容
    15. ctrl+alt+B:查询那些类实现了光标所在的接口
    16. ctrl+U:查看父类
    17. ctrl+E:最近编辑的文件列表
    18. ctrl+alt+home:查看布局与对应的类
    19. ctrl+alt+H:查看当前方法在那里进行调用
#运行编译
    1. ctrl+F9:构建
    2. shift+F10:运行

#工程目录操作
    1. 新建文件及工程:选中要创建目录使用alt+insert
    2. ctrl+shift+a:输入show in explorer-->打开相应目录
    3. ctrl+alt+s:打开软件设置
    4. ctrl+alt+shift+s:打开module设置
    5. alt+1:当前目录区
    6. alt+7:当前类的方法列表查看
    7. ctrl+tab:切换目录及视图
    8. alt+shift+c:查看工程最近更改的地方
    9. ctrl+J:livetemp模板查看
#代码快捷操作
    1. 没有操作完成操作可以先写todo进行,就可以在todo的窗口进行查看
    2. F11定义书签
    3. shift+F11:查看书签
    4. ctrl+J:快捷调出模板
    5. alt+点击断点:禁用断点
    6. 调试状态下按下:alt查看变量能审查表达式的值
  
#组合快捷键
    1. F2:定位错误
    2. alt+enter:修正错误
    
    3. alt+鼠标:进入列编辑模式
    4. ctrl+w:选中单词
    5. 或其他组合操作 

#编辑的位置
    ctrl+alt+左右键:这个是定位到编辑的位置
# ctrl+H 查看继承关系#

# ctrl+shift+R 全局搜索替换#
# ctrl+shift+F 全局搜索#