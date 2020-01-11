## 反射：框架设计的灵魂
- 反射机制即将类的各个组成部分组装为其他对象
优势：可以在程序运行过程中，操作这些对象 ；可以解耦，提高程序的可扩展性；
## 获取Class对象的三种方式
- ```Class.forName("类名")```:将字节码文件加载进内存，返回Class对象
多用于配置文件，将类名定义在配置文件中，读取文件加载类
- ```类名.class```通过类名的属性class获取
多用于参数的传递
- ```对象.getClass()```:方法定义在Object类
多用于对象的获取字节码的方式
 结论：同一个字节码文件(*.class)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个
## Class对象功能
- 获取功能
1. 获取成员变量
```java
Field[] getFields()//获取所有public修饰的成员变量
Field getField(String name)
```
2. 获取构造方法
```java
Constructor[] getConstructors()
Field getConstructor(类<?> parameterTypes)
```
3. 获取成员方法
```java
Method[] getMehtods()
Method getMethod(String name,类<?> parameterTypes)
```
4. 获取类名
```java
String getName()
```
```java
//获取私有成员则使用
setAccessible(true)//暴力反射
getDeclareFields()
```
## 案例
- 要求：完成一个框架，可以创建任意类的对象，并且执行其中任意方法
```java
//将需要创建的对象的全类名和需要执行的方法定义在配置文件中
Properties pro = new Properties();
//在程序中加载读取配置文件
ClassLoader classloader = ReflectTest.class.getClassLoader();
InputStream is = classLoader.getResourceAsStream("pro.properties");
pro.load(is);
//使用反射技术来加载类文件进内存
String className = pro.getProperty("className");
String methodName = pro.getProperty("methodName");
//创建对象
Class cls = Class.forName(className);
Object obj = cls.newInstance();
//执行方法
cls.getMethod(methodName);
method.invoke(obj);
```

## 注解
- 注解，也叫元数据。一种代码级别的说明，说明

- 作用：
  编写文档：通过代码里标识的元数据生成文档
  代码分析：通过代码里标识的元数据对代码进行分析
  编译检查：通过代码里标识的元数据让编译器能够实现基本的编译检查
### JDK预定义的一些注解：
- @Override：检测被该注解标注的方法是否是继承自父类（接口）的
- @Deprecated：该注解标注的内容，表示已过时
- @SuppressWarnings：压制警告
一般传递参数all ```@SuppressWarnings(“all”)```
### 自定义注解
- 格式： 
```
元注解
public @interface 注解名称{
		属性列表;
}
```
- 本质：注解本质上就是一个接口，
```public interface MyAnno extends java.lang.annotation.Annotation{}```
- 属性：接口中的抽象方法
要求：
属性的返回值类型：基本数据类型、String、枚举、注解、以及以上类型的数组
定义了属性，在使用时需要给属性赋值：若default修饰，可省略；如果只有一个value属性，可直接定义值；数组赋值时，值使用{}
- 元注解：用于描述注解的注解
@Target：描述能够作用的位置
ElementType取值：TYPE作用于类，METHOD作用于方法，FIELD作用于成员变量上
@Retention：描述注解能够被保留的阶段
一般使用RetentionPolicy,RUNTIME，表示当前描述的注解，会保留到class字节码文件中
@Documented：描述注解是否被抽取到api文档中
@Inherited：描述注解是否被子类继承
### 解析注解
```
//获取该类的字节码对象
Class<ReflectTest> reflectTestClass =ReflectTest.class;
//获取注解对象（本质是在内存中生成了该注解接口的子类实现对象
Pro pro=reflectTestClass.getAnnotation(Pro.class);
//调用注解对象中定义的抽象方法，获取返回值
String className= pro.className();
```
### 案例测试
```java
Calculator c=new Calculator();
Class cls = c.getClass();
Method[] methods = cls.getMethods();
int number=0;
BUfferedWriter bw =new BufferedWriter(new FIleWriter("bug.txt"));
for(Method method:methods){
	if(method.isAnnotationPresent(Check.class)){
		try{
			method.invoke(c);
		}catch(Exception e){
			number++;
			bw.write(method.getName()+"出异常");
			bw.newLine();
			bw.write("名称"+e.getCause().getClass().getSimpleName());
			bw.newLine();
			bw.write("原因"+e.getCause().getMessage());
			bw.newLine();
		}
	}
}
bw.write("出现次数"+number);
bw.flush();
```
