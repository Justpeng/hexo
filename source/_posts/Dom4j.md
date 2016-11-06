---
title: Dom4j-使用指导
date: 2016-10-22 17:58:27
description: "Dom4j的基础读写处理"
categories: [Dom4j,XML]
tags: [Dom4j]
top: 7
---

## what is dom4j?
dom4j 是一个为Java服务的开源的XML框架，可以对xml文档进行读写,操作,创建和修改，集成了DOM和SAX，同样支持xPath

接下来我们开始了解dom4j的基本用法

## 解析xml文档

### 获得Document对象

解析xml文档，首先需要通过 SAXReader 创建一个 document对象
> SAXReader.read() 方法可以通过多种方式进行读取，例如：URL、InputStream、File、Reader等

- URL路径方式

```
 public Document parse(URL url) throws DocumentException {
        SAXReader reader = new SAXReader();
        Document document = reader.read(url);
        return document;
    }
```
- InputStream方式


```
    SAXReader reader = new SAXReader();
    //获取类加载器
    ClassLoader loader = Thread.currentThread().getContextClassLoader();
    //获取文件流
    java.io.InputStream inputStream = loader.getResourceAsStream("out.xml");
    //创建document对象
    Document doc = reader.read(inputStream);
```

- File方式

```
    SAXReader saxReader = new SAXReader();
    File file = new File("out.xml");
    Document document = saxReader.read(file);
```

那么，接下来就是对xml中的Element(Node)，进行获取了。

### 获取root元素
方法

```
Element root = document.getRootElement();

```

### 使用Iterator获取元素
dom4j通过内置的方法，可以返回标准的java迭代，方便我们获取元素

```
   public void bar(Document document){
        Element element = document.getRootElement();

        //root 元素的子元素进行迭代
        for(Iterator i = element.elementIterator();i.hasNext();) {
           Element element1 = (Element) i.next();
            // do something
        }
        //iterate through child elements of root with element name "foo"
        //获取子元素name为 foo的迭代
        for(Iterator i=element.elementIterator("foo");i.hasNext();) {
            Element foo = (Element) i.next();
            //do something
        }

        // iterate through attribute of root
        // 节点属性
        for(Iterator i = element.attributeIterator();i.hasNext();) {
            Attribute attribute = (Attribute) i.next();
            // do something
        }
    }
```

例如，模拟SpringIOC 依赖注入机制(通过反射实现)，有如下spring_core.xml配置，
```
<?xml version="1.0" encoding="UTF-8"?>

<beans>
  <bean id="testBean" class="com.just.Test"/>
</beans>

```
我们需要根据spring_core.xml配置来获取bean id为 "testBean" 的 class,即类的全路径，获取类的全路径后，就可以随心所欲的进行反射调用了~

```
   for (Iterator i = root.elementIterator("bean"); i.hasNext();){
        foo = (Element) i.next();
        //针对每一个bean实例获取id和name属性
        Attribute id = foo.attribute("id");
        Attribute clazz = foo.attribute("class");

        //利用反射，通过class名称获取class对象
        Class bean = Class.forName(clazz.getText());
        // other 省略无关的反射操作
  }
```

如果需要处理的XML文件非常大，每次循环都使用Iterator，那么性能会降低，于是dom4j还提供了更加快速的获取元素的方法 element.nodeCount

```
public class ReadXml {
    public static void init(){
        try {
            SAXReader saxReader = new SAXReader();
            File file = new File("spring_core.xml");
            Document document = saxReader.read(file);
            Element rootElement = document.getRootElement();
            treeWalk(rootElement);
        } catch (DocumentException e) {
            e.printStackTrace();
        }
    }
    public static  void treeWalk(Element element) {
        for ( int i = 0, size = element.nodeCount(); i < size; i++ ) {
            Node node = element.node(i);
            if(node instanceof  Element){
                Element e = (Element) node;
                System.out.println(e.attribute("id").getText());
                System.out.println(e.attribute("class").getText());
            }
        }
    }
    public static void main(String args[]){
        init();
    }
}
```

## 创建一个document
使用对象 DocumentHelper对象

```
 public Document createDocument() {
        Document document = DocumentHelper.createDocument();
        Element root = document.addElement("root");
        Element author1 = root.addElement("author")
                .addAttribute("name", "lip")
                .addText("1323");
        Element author2 = root.addElement("author")
                .addAttribute("name", "lin")
                .addText("4444");
        return  document;
    }
```

## 写入文件

通过创建XMLWriter对象进行写操作

```
public class WriteXml {
    public static void init() throws IOException {
        Document document = DocumentHelper.createDocument();
        Element root = document.addElement("beans");
        Element bean = root.addElement("bean")
                .addAttribute("id", "testBean")
                .addAttribute("class","com.just.Test");
        write(document);
    }

    public static void write(Document document) throws IOException {
        //输出到文件 out.xml
        XMLWriter writer = new XMLWriter(new FileWriter("out.xml"));
        writer.write(document);
        writer.flush();
        writer.close();
        System.out.println("------------");
        /**
         * 通过 createPrettyPrint美化输出到控制台
         */
        OutputFormat outputFormat = OutputFormat.createPrettyPrint();
        writer = new XMLWriter(System.out, outputFormat);
        writer.write(document);
        System.out.println("----------------------------------");
        /**
         * 实现压缩
         */
        outputFormat = OutputFormat.createCompactFormat();
        writer = new XMLWriter(System.out, outputFormat);
        writer.write(document);

    }
    public static void main(String args[]) throws IOException {
        init();
    }
}
```
## String与Xml转换
- String 转为document对象或其他Node，使用 DocumentHelper.parseText()
- document或其他node节点，转为String，直接调用 .asXML()
```
   /**
     * convert from string
     */
    public Document convert(String xml) throws DocumentException {
        Document document = DocumentHelper.parseText(xml);
        return  document;
    }

    /**
     * convert to string
     */
    public String convert(Document document) throws DocumentException {
        String str = document.asXML();
        return str;
    }
```
## xPath路径处理
对于xPath,在浏览器debug模式中，查看元素，对于元素进行复制时可以保存为xPath，那么可以理解为一种路径表示方式，在自动化测试中，xPath是一种非常重要的获取页面元素的手段。

使用xPath需要引入依赖包，顺便贴上maven依赖
```
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.1</version>
</dependency>
```


- xPath的基本语法

expression | description
---|---
nodename | 当前节点的所有子节点
/ | 根节点选取
// |匹配文档中节点，不考虑位置
. | 当前节点
.. | 当前节点的父节点
@ |选取属性

for Example：

```
SAXReader saxReader = new SAXReader();
File file = new File("out.xml");
Document document = saxReader.read(file);
Element rootElement = document.getRootElement();
List<Object> list = document.selectNodes("//bean");
for (Object n : list) {
    if(n instanceof Element){
        Element e = (Element) n;
        System.out.println(e.getName());
        List<Object> aList  = e.selectNodes("@class");
        for(Object a :aList)
            if(a instanceof Attribute){
                Attribute attribute = (Attribute) a;
                System.out.println(attribute.getText());

            }
    }
}
```
输出：

```
bean
com.just.Test

```

## 处理XSLT
对于XSLT，不甚了解，贴上官方例子

```
public class Foo {

    public Document styleDocument(
        Document document,
        String stylesheet
    ) throws Exception {

        // load the transformer using JAXP
        TransformerFactory factory = TransformerFactory.newInstance();
        Transformer transformer = factory.newTransformer(
            new StreamSource( stylesheet )
        );

        // now lets style the given document
        DocumentSource source = new DocumentSource( document );
        DocumentResult result = new DocumentResult();
        transformer.transform( source, result );

        // return the transformed document
        Document transformedDoc = result.getDocument();
        return transformedDoc;
    }
}
```

## 写在后面
以上只是基于官方guide文档学习时所记录，更多移步[http://www.dom4j.org/](http://www.dom4j.org//)

.END