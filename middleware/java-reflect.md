# 利用反射机制遍历String的字段和方法、构造器

利用java反射机制能获取某个类或某个对象的所属类、拥有的方法、成员变量等信息。很多java反编译工具就是利用这个机制。

## 类名的获取

java所有的类/枚举/接口/注解，都属于Class 类型，这个Class和class是不一样的，class属于java关键字、Class是类名，如同String一样，且Class是个泛型类。

### 获取具体的某个类的Class常见的有三种方式：

- 在实例方法中 直接getClass，就能获取实例对象的Class类型

  知道类名可以通过以下两种方式获取Class

  - Class.forName("具体类名，包含包名如：java.lang.Object")

  - 直接用.class文件名称，如 Class<String> cls = String.class

    ## 方法的获取

    通过Class类型对象.getMethods获取所有公开方法、getDeclaredMethods获取所有方法 (不管是private还是public修饰的方法)  
    getMethod(String name, Class<?>... parameterTypes)  获取指定名称方法（必须是public类型方法）、parameterTypes表示该方法有哪些参数，parameterTypes长度可变，如果为空，表示这个方法没有参数。

    同样getDeclaredMethod(String name, Class<?>... parameterTypes) 就是获取指定方法（不管是public还是private修饰的方法），parameterTypes参数就不作解释说明了。

    它们的返回值类型都是Method，Method 有哪些详细的方法，可以直接看源码或者jdk-api； 得到Method后还可以后续得到方法里面的参数、注解，一直延伸下去，直至得到自己需要的内容。

    ## 变量（filed）、构造器以及注解的获取

    它们获取方式和方法非常雷同，如getFileds、getDeclaredFields 等、就不再赘述。

    - 说了这么多不如实战

    ## 代码实现

    ```java
     public static void showFiledsAndMethods(String classname){
            Class<?> cls = null;
            try{
                cls = Class.forName(classname);
            }catch (ClassNotFoundException e){
                e.printStackTrace();
            }
            System.out.println("---------------成员变量------------------");
            Stream.of(cls.getDeclaredFields()).forEach(System.out::println);
            System.out.println("---------------方法------------------");
            Stream.of(cls.getDeclaredMethods()).forEach(System.out::println);
            System.out.println("---------------构造器------------------");
            Stream.of(cls.getDeclaredConstructors()).forEach(System.out::println);
        }
        public static void main(String[] args) {
            showFiledsAndMethods("java.lang.String");
        }
    ```

    代码中使用的Stream流操作，用法可以参考：[JDK8新特性Stream实战](middleware/java8-practice.md)

    输出结果：

    ```java
    ---------------成员变量------------------
    private final char[] java.lang.String.value
    private int java.lang.String.hash
    private static final long java.lang.String.serialVersionUID
    private static final java.io.ObjectStreamField[] java.lang.String.serialPersistentFields
    public static final java.util.Comparator java.lang.String.CASE_INSENSITIVE_ORDER
    ---------------方法------------------
    public boolean java.lang.String.equals(java.lang.Object)
    public java.lang.String java.lang.String.toString()
    public int java.lang.String.hashCode()
    public int java.lang.String.compareTo(java.lang.String)
    public int java.lang.String.compareTo(java.lang.Object)
    public int java.lang.String.indexOf(java.lang.String,int)
    public int java.lang.String.indexOf(java.lang.String)
    public int java.lang.String.indexOf(int,int)
    public int java.lang.String.indexOf(int)
    static int java.lang.String.indexOf(char[],int,int,char[],int,int,int)
    static int java.lang.String.indexOf(char[],int,int,java.lang.String,int)
    public static java.lang.String java.lang.String.valueOf(int)
    public static java.lang.String java.lang.String.valueOf(long)
    public static java.lang.String java.lang.String.valueOf(float)
    public static java.lang.String java.lang.String.valueOf(boolean)
    public static java.lang.String java.lang.String.valueOf(char[])
    public static java.lang.String java.lang.String.valueOf(char[],int,int)
    public static java.lang.String java.lang.String.valueOf(java.lang.Object)
    public static java.lang.String java.lang.String.valueOf(char)
    public static java.lang.String java.lang.String.valueOf(double)
    public char java.lang.String.charAt(int)
    private static void java.lang.String.checkBounds(byte[],int,int)
    public int java.lang.String.codePointAt(int)
    public int java.lang.String.codePointBefore(int)
    public int java.lang.String.codePointCount(int,int)
    public int java.lang.String.compareToIgnoreCase(java.lang.String)
    public java.lang.String java.lang.String.concat(java.lang.String)
    public boolean java.lang.String.contains(java.lang.CharSequence)
    public boolean java.lang.String.contentEquals(java.lang.CharSequence)
    public boolean java.lang.String.contentEquals(java.lang.StringBuffer)
    public static java.lang.String java.lang.String.copyValueOf(char[])
    public static java.lang.String java.lang.String.copyValueOf(char[],int,int)
    public boolean java.lang.String.endsWith(java.lang.String)
    public boolean java.lang.String.equalsIgnoreCase(java.lang.String)
    public static java.lang.String java.lang.String.format(java.util.Locale,java.lang.String,java.lang.Object[])
    public static java.lang.String java.lang.String.format(java.lang.String,java.lang.Object[])
    public void java.lang.String.getBytes(int,int,byte[],int)
    public byte[] java.lang.String.getBytes(java.nio.charset.Charset)
    public byte[] java.lang.String.getBytes(java.lang.String) throws java.io.UnsupportedEncodingException
    public byte[] java.lang.String.getBytes()
    public void java.lang.String.getChars(int,int,char[],int)
    void java.lang.String.getChars(char[],int)
    private int java.lang.String.indexOfSupplementary(int,int)
    public native java.lang.String java.lang.String.intern()
    public boolean java.lang.String.isEmpty()
    public static java.lang.String java.lang.String.join(java.lang.CharSequence,java.lang.CharSequence[])
    public static java.lang.String java.lang.String.join(java.lang.CharSequence,java.lang.Iterable)
    public int java.lang.String.lastIndexOf(int)
    public int java.lang.String.lastIndexOf(java.lang.String)
    static int java.lang.String.lastIndexOf(char[],int,int,java.lang.String,int)
    public int java.lang.String.lastIndexOf(java.lang.String,int)
    public int java.lang.String.lastIndexOf(int,int)
    static int java.lang.String.lastIndexOf(char[],int,int,char[],int,int,int)
    private int java.lang.String.lastIndexOfSupplementary(int,int)
    public int java.lang.String.length()
    public boolean java.lang.String.matches(java.lang.String)
    private boolean java.lang.String.nonSyncContentEquals(java.lang.AbstractStringBuilder)
    public int java.lang.String.offsetByCodePoints(int,int)
    public boolean java.lang.String.regionMatches(int,java.lang.String,int,int)
    public boolean java.lang.String.regionMatches(boolean,int,java.lang.String,int,int)
    public java.lang.String java.lang.String.replace(char,char)
    public java.lang.String java.lang.String.replace(java.lang.CharSequence,java.lang.CharSequence)
    public java.lang.String java.lang.String.replaceAll(java.lang.String,java.lang.String)
    public java.lang.String java.lang.String.replaceFirst(java.lang.String,java.lang.String)
    public java.lang.String[] java.lang.String.split(java.lang.String)
    public java.lang.String[] java.lang.String.split(java.lang.String,int)
    public boolean java.lang.String.startsWith(java.lang.String,int)
    public boolean java.lang.String.startsWith(java.lang.String)
    public java.lang.CharSequence java.lang.String.subSequence(int,int)
    public java.lang.String java.lang.String.substring(int)
    public java.lang.String java.lang.String.substring(int,int)
    public char[] java.lang.String.toCharArray()
    public java.lang.String java.lang.String.toLowerCase(java.util.Locale)
    public java.lang.String java.lang.String.toLowerCase()
    public java.lang.String java.lang.String.toUpperCase()
    public java.lang.String java.lang.String.toUpperCase(java.util.Locale)
    public java.lang.String java.lang.String.trim()
    ---------------构造器------------------
    public java.lang.String(byte[],int,int)
    public java.lang.String(byte[],java.nio.charset.Charset)
    public java.lang.String(byte[],java.lang.String) throws java.io.UnsupportedEncodingException
    public java.lang.String(byte[],int,int,java.nio.charset.Charset)
    public java.lang.String(byte[],int,int,java.lang.String) throws java.io.UnsupportedEncodingException
    java.lang.String(char[],boolean)
    public java.lang.String(java.lang.StringBuilder)
    public java.lang.String(java.lang.StringBuffer)
    public java.lang.String(byte[])
    public java.lang.String(int[],int,int)
    public java.lang.String()
    public java.lang.String(char[])
    public java.lang.String(java.lang.String)
    public java.lang.String(char[],int,int)
    public java.lang.String(byte[],int)
    public java.lang.String(byte[],int,int,int)
    ```

    

