# 最简单的商场收银软件 #

如果要做一款收银软件，营业员根据客户所购买商品单价和数量向客户收费，这非常容易。

Demo 如下：

![](http://i.imgur.com/XBz9iW5.png)

点击确定后的代码逻辑如下:

      private void doEnter() {
      String stringTotal = tv_total.getText().toString().trim();
      double total;
      if (stringTotal != null && stringTotal != "") {
        total = Double.valueOf(stringTotal);
      } else {
        total = 0.0d;
      }

      String stringDJ = et_dj.getText().toString().trim();
      String stringSL = et_sl.getText().toString().trim();
      if (stringDJ != null && !stringDJ.equals("") && stringSL != null && !stringSL.equals("")) {
        Log.d("TAG", "123" + stringDJ + "123");
        double price = Double.valueOf(stringDJ);
        int number = Integer.valueOf(stringSL);
        double totalPrice = price * number;
        total = total + totalPrice;
        tv_total.setText(String.valueOf(total));
        String text = tv_detail.getText().toString()
            + "单价： "
            + price
            + " 数量："
            + number
            + " 合计:"
            + totalPrice
            + "\n";
        tv_detail.setText(text);
      }
    }

#增加打折功能后的收银软件 #
可是如果商场搞促销，需要打折该怎么办，不可能每次都要修改代码然后重新安装，用下拉框可能会比较方便。
Demo 如下：

![](http://i.imgur.com/sLqCANA.png)

添加的代码如下：

         switch (sp_jsfs.getSelectedItemPosition()){
          case 1:
            totalPrice*=0.8;
            break;
          case 2:
            totalPrice*=0.7;
            break;
          case 3:
            totalPrice*=0.5;
            break;
        }
这样看似解决了问题，但是需求不断增加，比如满300返50之类，这样的代码未免显得太过重复。接下来我们试着用简单工厂模式来解决问题试试。

# 简单工厂实现 #
> 面向对象的编程，并不是类越多越好，类的划分是为了封装，但分类的基础是抽象，具有相同属性和功能的对象抽象集合才是类。

打一折和九折只是形式的不同，抽象分析出来，所有打折算法都是一样的，所以打折算法应该是一个类。返现算法也是一个类。

MainActivity 改动如下
		CashSuper cSuper= CashFactor.createCashAccept(sp_jsfs.getSelectedItemPosition());
        totalPrice=cSuper.acceptCash(totalPrice);
详细代码可以去最后上传的Demo里看
简单工厂模式虽然也能解决问题，但只是解决对象创建的问题，而且由于工厂本身包括了所有的收费方式，商场是可能经常性地更改打折和返利额度，每次维护或扩展收费方式都要改动这个工厂，以至代码要重新编译部署，这是很糟糕的，所以我们需要另一种新的设计模式--策略模式。
# 策略模式 #
## 什么是策略模式 ##
> 策略模式就是定义一系列算法，把他们独立封装起来，并且这些算法之间可以相互替换。策略模式主要是管理一堆有共性的算法，客户端可以根据需要，很快切换这些算法，并且保持可扩展性。
> 策略模式的本质：分离算法，选择实现。

## 如何运用到收银系统中##
商场收银如何促销，用打折还是返利，其实都是一些算法，用工厂来生成算法对象，这没有错，但算法本身只是一种策略，最重要的是这些算法是随时都可能互相替换的，这是变化点，而封装变化点是我们面向对象的一种很重要的思维方式。

以下是策略模式 UML 图
![](http://i.imgur.com/WxFL8fa.png)

接下来我们将策略模式运用到收银系统中
首先创建一个 CashContext 代码如下：

		public class CashContext {
		  private CashSuper cs;
		
		  public CashContext(CashSuper cs) {
		    this.cs = cs;
		
		  }
		  public double GetResule(double money){
		    return cs.acceptCash(money);
		  }
		}
然后改动 MainActivity 如下：

		CashContext cc = null;
        switch (sp_jsfs.getSelectedItemPosition()) {
          case 0:
            cc = new CashContext(new CashNormal());
            break;
          case 1:
            cc = new CashContext(new CashReturn(300, 100));
            break;
          case 2:
            cc = new CashContext(new CashRebate(0.8));
            break;
          case 3:
            cc = new CashContext(new CashRebate(0.7));
            break;
          case 4:
            cc = new CashContext(new CashRebate(0.5));
            break;
        }
        totalPrice = cc.GetResule(totalPrice);
		

这时候，你会发现，我们又像原来一样在 MainActivity 中写了判断，可以试着将之前的工厂模式和策略模式结合吗？

##策略模式与简单工厂结合##
将 CashContext 类的构造方法修改如下：

		public CashContext(int type) {
		    switch (type) {
		      case 0:
		        cs=new CashNormal();
		        break;
		      case 1:
		        cs=new CashReturn(300,100);
		        break;
		      case 2:
		        cs = new CashRebate(0.8);
		        break;
		      case 3:
		        cs = new CashRebate(0.7);
		        break;
		      case 4:
		        cs = new CashRebate(0.5);
		        break;
		    }
		  }
MainActivity 代码修改如下：

		CashContext cc = new CashContext(sp_jsfs.getSelectedItemPosition());
        totalPrice = cc.GetResule(totalPrice);
        total = total + totalPrice;

这样客户端只需要认识一个类 CashContext就可以了，耦合度进一步降低了。
不过这样一旦需求变化依旧需要修改 switch ，其实想要更好的实现可以用反射方法，具体用法下次再做讨论。

# Demo #

[策略模式 Demo](https://github.com/jutao/strategymodel)

# 参考文献 #

《大话设计模式》

[安卓设计模式--策略模式](http://mobile.51cto.com/ahot-418972.htm)
