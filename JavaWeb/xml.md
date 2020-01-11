# XML & Tomcat
## XML
> extendsible markup language 可扩展的标记语言
### 简介
#### 文档结构 
- 树形结构
#### 文档声明
- ```<?xml version="1.0" encoding="UTF-8" standalone="no"?>```
#### encoding
- 正常显示中文使用GBK或GB2312进行编码
#### 元素定义（标签）
- ```<>```内的即为元素，成对，第一个元素为根元素
#### 属性定义
- 定义在元素内部，```<元素名称 属性名称="属性的值"></元素名称>```
#### 注释
- 同HTML ```<!-- -->```
#### CDATA区
- 非法字符
	< &lt; & &amp; > &gt;
- 非法字符区（通常用于服务器给客户端返回数据
> <des><![CDATA[<a href="http://ww.baidu.com">百度</a>]]></des>
### 解析
- DOM Document Object model 把整个xml读入内存当中，形成树状结构。整个文档称之为Document对象，所有的元素节点对应Element对象，文本也可以称之为Text对象，以上所有对象都可以称之为Node节点。如果XML特别大，将可能造成内存溢出
- SAX Simpel API for XML 基于事件驱动，读取一行，解析一行，不会造成事件溢出
- 针对以上两种解析方式给出的解决方案 使用比较广泛的 dom4j
#### dom4j基本用法
- 创建SaxReader对象

- 指定解析的xml

- 获取根元素。

- 根据根元素获取子元素或者下面的子孙元素

		try {
			//1. 创建sax读取对象
			SAXReader reader = new SAXReader(); //jdbc -- classloader
			//2. 指定解析的xml源
			Document  document  = reader.read(new File("src/xml/stus.xml"));	

```
//3. 得到元素、
//得到根元素
Element rootElement= document.getRootElement();

//获取根元素下面的子元素 age
//rootElement.element("age") 
//System.out.println(rootElement.element("stu").element("age").getText());
```


			//获取根元素下面的所有子元素 。 stu元素
			List<Element> elements = rootElement.elements();
			//遍历所有的stu元素
			for (Element element : elements) {
				//获取stu元素下面的name元素
				String name = element.element("name").getText();
				String age = element.element("age").getText();
				String address = element.element("address").getText();
				System.out.println("name="+name+"==age+"+age+"==address="+address);
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		}
- XPaths使用
		```
		//要想使用Xpath， 还得添加支持的jar 获取的是第一个 只返回一个。
		Element nameElement = (Element) rootElement.selectSingleNode("//name");
		System.out.println(nameElement.getText());

		//获取文档里面的所有name元素 
		List<Element> list = rootElement.selectNodes("//name");
		for (Element element : list) {
			System.out.println(element.getText());
		}
	```
- 约束
	* DTD
	语法自成一派， 早起就出现的。 可读性比较差。 
		+ 引入网络上的DTD
		```
		   	<!-- 引入dtd 来约束这个xml -->		
		   	<!--    文档类型  根标签名字 网络上的dtd   dtd的名称   dtd的路径
		   	<!DOCTYPE stus PUBLIC "//UNKNOWN/" "unknown.dtd"> -->
		```
		+ 引入本地的DTD
		```
			<!-- 引入本地的DTD  ： 根标签名字 引入本地的DTD  dtd的位置 -->
			<!-- <!DOCTYPE stus SYSTEM "stus.dtd"> -->
		```
		+ 直接在XML里嵌入DTD的约束规则
		```
		<!-- xml文档里面直接嵌入DTD的约束法则 -->
		   	<!DOCTYPE stus [
		   		<!ELEMENT stus (stu)>
		   		<!ELEMENT stu (name,age)>
		   		<!ELEMENT name (#PCDATA)>
		   		<!ELEMENT age (#PCDATA)>
		   	]>		   	
		   	<stus>
		   		<stu>
		   			<name>张三</name>
		   			<age>18</age>
		   		</stu>
		   	</stus>
		
		
				<!ELEMENT stus (stu)>  : stus 下面有一个元素 stu  ， 但是只有一个
				<!ELEMENT stu (name , age)>  stu下面有两个元素 name  ,age  顺序必须name-age
				<!ELEMENT name (#PCDATA)> 
				<!ELEMENT age (#PCDATA)>
				<!ATTLIST stu id CDATA #IMPLIED> stu有一个属性 文本类型， 该属性可有可无
		```
				元素的个数：
			
					＋　一个或多个
					*  零个或多个
					? 零个或一个
			
				属性的类型定义 
			
					CDATA : 属性是普通文字
					ID : 属性的值必须唯一
		
		
				```<!ELEMENT stu (name , age)>```		按照顺序来 
			
				```<!ELEMENT stu (name | age)>```   两个中只能包含一个子元素
	* Schema
	其实就是一个xml ， 使用xml的语法规则， xml解析器解析起来比较方便 ， 是为了替代DTD 。但是Schema 约束文本内容比DTD的内容还要多。 所以目前也没有真正意义上的替代DTD
		约束文档：
		```
		<!-- xmlns  :  xml namespace : 名称空间 /  命名空间
		targetNamespace :  目标名称空间 。 下面定义的那些元素都与这个名称空间绑定上。 
		elementFormDefault ： 元素的格式化情况。  -->
		<schema xmlns="http://www.w3.org/2001/XMLSchema" 
			targetNamespace="http://www.itheima.com/teacher" 
			elementFormDefault="qualified">
			
			<element name="teachers">
				<complexType>
					<sequence maxOccurs="unbounded">
						<!-- 这是一个复杂元素 -->
						<element name="teacher">
							<complexType>
								<sequence>
									<!-- 以下两个是简单元素 -->
									<element name="name" type="string"></element>
									<element name="age" type="int"></element>
								</sequence>
							</complexType>
						</element>
					</sequence>
				</complexType>
			</element>
		</schema>
		```
		实例文档：
		```
		<?xml version="1.0" encoding="UTF-8"?>
			<!-- xmlns:xsi : 这里必须是这样的写法，也就是这个值已经固定了。
			xmlns : 这里是名称空间，也固定了，写的是schema里面的顶部目标名称空间
			xsi:schemaLocation : 有两段： 前半段是名称空间，也是目标空间的值 ， 后面是约束文档的路径。
			 -->
			<teachers
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xmlns="http://www.itheima.com/teacher"
				xsi:schemaLocation="http://www.itheima.com/teacher teacher.xsd"
			>
				<teacher>
					<name>zhangsan</name>
					<age>19</age>
				</teacher>
				<teacher>
					<name>lisi</name>
					<age>29</age>
				</teacher>
				<teacher>
					<name>lisi</name>
					<age>29</age>
				</teacher>
			</teachers>
		```
- 名称空间的作用
一个xml如果想指定它的约束规则， 假设使用的是DTD ，那么这个xml只能指定一个DTD  ，  不能指定多个DTD 。 但是如果一个xml的约束是定义在schema里面，并且是多个schema，那么是可以的。简单的说： 一个xml 可以引用多个schema约束。 但是只能引用一个DTD约束。

### 程序架构

- C/S(client/server):QQ 微信/lol
	
	* 部分代码放在客户端，用户体验好；服务器更新，客户端随之更新，占用资源大
	
- B/S（browser/server）webQQ，页游
	* 占用资源小，无需更新；用户体验不佳

## Tomcat

- bin 包含了一些jar ,  bat文件 。  startup.bat

- conf tomcat的配置 	server.xml  web.xml

- lib tomcat运行所需的jar文件

- logs 运行的日志文件

- temp 临时文件

- webapps 发布到tomcat服务器上的项目，就存放在这个目录。	

- work jsp翻译成class文件存放地
  ####发布

- 拷贝这个文件到webapps/ROOT底下，在浏览器里面访问

- 虚拟路径配置方式1
	* 使用localhost：8080 打开tomcat首页， 在左侧找到tomcat的文档入口， 点击进去后， 在左侧接着找到 Context入口，点击进入。http://localhost:8080/docs/config/context.html
	* 在conf/server.xml 找到host元素节点。
	* 加入以下内容。
	```<!-- docBase ：  项目的路径地址 如： D:\xml02\person.xml
		path : 对应的虚拟路径 一定要以/打头。
		对应的访问方式为： http://localhost:8080/a/person.xml -->
		<Context docBase="D:\xml02" path="/a"></Context>
	```
	* 在浏览器地址栏上输入： http://localhost:8080/a/person.xml
	
- 虚拟路径配置方式2
	* 在tomcat/conf/catalina/localhost/ 文件夹下新建一个xml文件，名字可以自己定义。 person.xml
	* 在这个文件里面写入以下内容
	```
	<?xml version='1.0' encoding='utf-8'?>
	<Context docBase="D:\xml02"></Context>
	```
	* 在浏览器上面访问 http://localhost:8080/person/xml的名字即可