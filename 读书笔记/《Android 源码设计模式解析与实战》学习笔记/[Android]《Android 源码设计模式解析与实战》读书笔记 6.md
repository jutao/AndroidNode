# 工厂方法模式介绍
创建型设计模式之一，使用非常广泛，Android 中的 Activity 生命周期 中的 oncreate 就可以被看作是一个工厂方法。在任何需要生成复杂对象的地方，都可以使用工厂方法模式。

# 实战代码
**写一个抽象类**

    public abstract class IOHandler {
      /**
       * 添加一条个人信息
       * @param id 身份证号码
       * @param name  名字
       */
      public abstract void add(String id,String name);

      /**
       * 删除一条个人信息
       * @param id 身份证号码
       */
      public abstract void remove(String id);

      /**
       * 更新一条个人信息
       * @param id 身份证号码
       * @param name  名字
       */
      public abstract void update(String id,String name);

      /**
       * 查询身份证对应的人名
       * @param id 身份证号码
       */
      public abstract String query(String id);

    }

**三个实现类**

    public class FileHandler extends IOHandler{
      @Override
      public void add(String id, String name) {

      }

      @Override
      public void remove(String id) {

      }

      @Override
      public void update(String id, String name) {

      }

      @Override
      public String query(String id) {
          return "FILE";
      }
      }

    public class DBHandler extends IOHandler{
        @Override
        public void add(String id, String name) {

        }

        @Override
        public void remove(String id) {

        }

        @Override
        public void update(String id, String name) {

        }

        @Override
        public String query(String id) {
            return "DB";
        }
    }


    public class XMLHandler extends IOHandler {
      @Override
      public void add(String id, String name) {

      }

      @Override
      public void remove(String id) {

      }

      @Override
      public void update(String id, String name) {

      }

      @Override
      public String query(String id) {
          return "XML";
      }
    }

**工厂类**

    public class IOFactory {
        public static <T extends IOHandler> T getIOHandler(Class<T> clz) {
            IOHandler handler = null;
            try {
                handler = (IOHandler) Class.forName(clz.getName()).newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return (T) handler;
        }
    }

**使用**

    public class Main {
        public static void main(String[] args) {
            IOHandler ioHandler1 = IOFactory.getIOHandler(FileHandler.class);
            System.out.println(ioHandler1.query("123"));
                IOHandler ioHandler2 = IOFactory.getIOHandler(XMLHandler.class);
            System.out.println(ioHandler2.query("123"));
            IOHandler ioHandler3 = IOFactory.getIOHandler(DBHandler.class);
            System.out.println(ioHandler3.query("123"));
        }
    }
