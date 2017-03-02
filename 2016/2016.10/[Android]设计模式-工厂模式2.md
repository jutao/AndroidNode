<h1 id="">工厂模式</h1>

<p>关于工厂模式的基本概念和使用方法，<a href="http://blog.qiji.tech/archives/6357">工厂模式</a> 中已经有了不错的介绍了。</p>

<h1 id="">反射和工厂模式的结合</h1>

<p>@黛千秋 在她介绍工厂模式的例子中写了一个关于工厂模式的例子，在这借用一下。</p>

<p>以下代码为工厂模式结合反射实例，这样可以不用帮每种饮料都创建一个工厂类：
其他基本代码都一样这里不贴，先是创建一个饮料工厂抽象类</p>

<pre><code>    public abstract class DrinkFactory {
        public abstract <T extends Drink>T createDrink(Class<T> clz);   
    }
</code></pre>

<p>接下来是抽象方法的实现类：</p>

<pre><code>    public class DrinkFactoryimpl extends DrinkFactory {

        @Override
        public <T extends Drink> T createDrink(Class<T> clz) {
            Drink drink = null;
            try {
                drink = (Drink) Class.forName(clz.getName()).newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }

            return (T) drink;
        }

    }
</code></pre>

<p>最后是客户端方法：</p>

<pre><code>    public class Console {
        public static void main(String[] args) {
            DrinkFactory factory=new DrinkFactoryimpl();

            Drink d1=factory.createDrink(Cola.class);
            d1.kind();

            Drink d2=factory.createDrink(Coffee.class);
            d2.kind();

            Drink d3=factory.createDrink(Fanta.class);
            d3.kind();

        }
    }
</code></pre>

<p>运行结果：</p>

<p><img src="http://i.imgur.com/wSZXRcm.png" alt="" title=""></p>

<h1 id="java">Java 中的工厂模式实现</h1>

<p>List 和 Set都继承于 Collection 接口，而 Collection 接口继承于 Iterable 接口.</p>

<pre><code>    public interface Iterable<T> {
        Iterator<T> iterator();
    }
</code></pre>

<p>以上是 Iterable 接口内的一个方法，也就是迭代器。我们使用 ArrayList 的时候常常这样使用：</p>

<pre><code>    ArrayList.iterator()；
</code></pre>

<p>方法来返回一个迭代器对象。我们来看一下 ArrayList 的源码：</p>

<pre><code>    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
          }
</code></pre>

<p>ArrayList 中的 itertor 方法其实就相当于一个工厂方法，专门为 new 对象而生。</p>

<h1 id="android">Android 中的工厂方法模式实现</h1>

<pre><code>    public class AActivity extends Activity {
      @Override protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(new LinearLayout(this));
      }
    }
</code></pre>

<p>构造一个线性布局 LinerarLayout 对象并设置为当前 Activity 的根布局，Oncreate 方法就相当于一个工厂方法，为什么呢，我们知道，所有控件都是View的子类，上面代码中，AActivity 的 onCreate 方法构造一个 View 对象，并设置为当前界面的 ContentView 返回给 framework 处理，如果现在又有一个 BActivity，这时我们又在其 onCreate 方法中通过 setContentView 方法设置另外不同的View，这是不是就是一个工厂模式的结构呢？</p>

<h1 id="">参考资料</h1>

<p>《Android 源码设计模式解析与实战》</p>