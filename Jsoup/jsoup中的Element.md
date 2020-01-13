## jsoup的Element类

### 一、简介

该类是Node的直接子类，同样实现了可克隆接口。类声明：public class Element extends Node

它表示由一个标签名，多个属性和子节点组成的html元素。从这个元素中，你可以提取数据，可以遍历节点树，可以操纵html。

 

### 二、构造方法

1、public Element(Tag tag, String baseUri, Attributes attributes)  创建一个新的、独立的元素。独立即没有父节点。attributes指初始属性。

2、public Element(Tag tag, String baseUri)  使用标签和base URL创建一个新的元素。

 

### 三、方法详细

1、public String nodeName()  得到节点名

2、public String tagName()  得到元素的标签名 如div

3、public Element tagName(String tagName)  改变元素的标签 。如：el.tagName("div")  把一个<span>标签改变为一个<div>标签。

4、public Tag tag() 得到元素的Tag

5、public boolean isBlock()测试元素是否是块级元素。

6、public String id() 得到元素的id属性

7、public Element attr(String attributeKey, String attributeValue) 设置元素的属性值。如果该键已存在，则替换掉以前的值；否则就新增。

8、public Map<String,String> dataset() 得到元素的HTML5自定义数据属性。元素中每个以"data-"开头的键的属性都包含在数据集范围内。

如这个元素<div data-package="jsoup" data-language="Java" class="group">... 就有如下数据集：package=jsoup, language=java.

返回的map是对元素属性的筛选后的map集合。对一个map的改变（增、删、改）会影响其他map

9、public final Element parent()  得到节点的父节点

10、public Elements parents()   得到元素的父类和祖先节点直到文档的根。返回元素最接近的一个父类的堆栈。

11、public Element child(int index)  通过索引得到元素的子元素。

注意：一个元素的子元素可以是元素和节点的混合。这个方法只检查过滤后的子元素的集合（保留那些children是elements的children，过滤掉children是Nodes的children），index也是基于过滤后的集合的索引。

12、public Elements children() 得到子元素集

13、public List<TextNode> textNodes()   得到元素的子文本节点集合。该集合不可修改但是文字节点可以被操纵。这是一个比较有效率的过滤文字节点的方法。

例如这段html:<p>One <span>Two</span> Three <br> Four</p>   用p元素来选择。
```
p.text() = "One Two Three Four"
p.ownText() = "One Three Four"
p.children() = Elements[<span>, <br>]
p.childNodes() = List<Node>["One ", <span>, " Three ", <br>, " Four"]
p.textNodes() = List<TextNode>["One ", " Three ", " Four"]
```
14、public List<DataNode> dataNodes()   得到元素的子数据节点。该集合不可修改但是数据节点可以被操纵。这是一个比较有效率的过滤数据节点的方法。

15、public Elements select(String cssQuery)  查询匹配CSS query选择器的元素集。被匹配的元素可能是它本身，也可能是它的任意子元素。 这种方法通常比使用DOM类型getelementby *的方法更强大，因为多个过滤器可以结合。如:

el.select("a[href]") - finds links (a tags with href attributes)
el.select("a[href*=example.com]") - finds links pointing to example.com (loosely)
16、public Element appendChild(Node child)  给元素增加一个子节点。要求该子节点没有已经存在父类。

17、public Element prependChild(Node child) 在该元素的子元素们的最前面增加一个子节点。要求该子节点没有已经存在父类。

18、public Element appendElement(String tagName)  使用tagName创建一个新的元素，然后把它作为该元素的最后一个子元素。如：parent.appendElement("h1").attr("id", "header").text("Welcome");

19、public Element prependElement(String tagName)  创建一个新的元素，然后把它作为该元素的第一个子元素。

20、public Element appendText(String text)   创建一个新的文字节点，然后追加到该元素中。

21、public Element prependText(String text)   创建一个新的文字节点，置于该元素子元素的最前面。

22、public Element append(String html)  增加一段html到该元素中，该html会被解析，然后每个节点都会置于元素末尾。

23、public Element prepend(String html) 增加一段html到该元素中，该html会被解析，然后每个节点都会置于元素开头。

24、public Element before(String html)  在该元素前面插入一段指定的html到DOM树中。比如用来作为前面的兄弟节点。

25、public Element before(Node node)  在该节点前面插入一个指定的节点到DOM树中。比如用来作为前面的兄弟节点。

26、public Element after(String html)   在该元素后面插入一段指定的html到DOM树中。比如用来作为后面的兄弟节点。

27、public Element after(Node node) 在该节点后面插入一个指定的节点到DOM树中。比如用来作为后面的兄弟节点。

28、public Element empty()   移除该元素的所有子节点。

29、public Element wrap(String html)  用提供的html包装该元素。

30、public Elements siblingElements()  得到元素的兄弟元素。该元素本身不包含在内。

31、public Element nextElementSibling()   得到该元素的下一个兄弟元素。

32、public Element previousElementSibling()   得到该元素的上一个兄弟元素。

33、public Element firstElementSibling()   得到该元素的第一个兄弟元素。

34、public Integer elementSiblingIndex()   得到该元素在兄弟元素集合中的索引。如果该元素是第一个，则返回0.

35、public Element lastElementSibling()   得到该元素的最后一个兄弟元素。

36、public Elements getElementsByTag(String tagName)   根据tagName查询子元素集

37、public Element getElementById(String id)   通过ID查找元素。包括元素本身和其子元素都在查询范围内。

注意：该方法寻找的是从该元素开始的第一个匹配的ID对应的元素，如果从不同位置作为起点去寻找可能得到不同的匹配该ID的元素

38、public Elements getElementsByClass(String className)  寻找包含className的class的元素集。包括元素本身和其子元素都在查询范围内。不区分大小写。元素集可能包含多个class(<div class="header round first">) 这个方法会检测每一个class，所以你可以使用el.getElementsByClass("header")找到上面这个元素。

39、public Elements getElementsByAttribute(String key)  通过元素属性的键寻找元素集。

40、public Elements getElementsByAttributeStarting(String keyPrefix)  根据属性的前缀寻找元素。HTML5属性集使用data-前缀

41、public Elements getElementsByAttributeValue(String key, String value)  寻找属性为指定值的元素。不区分大小写。

42、public Elements getElementsByAttributeValueNot(String key, String value)   寻找不包含指定属性的或者包含但有不同值的元素集。不区分大小写。

43、public Elements getElementsByAttributeValueStarting(String key, String valuePrefix)  寻找键为key，值以valuePrefix开头的元素集。

44、public Elements getElementsByAttributeValueEnding(String key, String valueSuffix)  寻找键为key，值以valueSuffix作为后缀的元素集。

45、public Elements getElementsByAttributeValueContaining(String key, String match)  寻找键为key，值包含match的元素集。

46、public Elements getElementsByAttributeValueMatching(String key, Pattern pattern)  寻找键为key，值匹配给定的正则表达式的元素集。

47、public Elements getElementsByAttributeValueMatching(String key, String regex)  寻找键为key，值匹配给定的正则表达式的元素集。

48、public Elements getElementsByIndexLessThan(int index)  寻找那些索引小于index的兄弟元素集。

49、public Elements getElementsByIndexGreaterThan(int index)  寻找那些索引大于index的兄弟元素集。

50、public Elements getElementsByIndexEquals(int index)  寻找那些索引等于index的兄弟元素集。

51、public Elements getElementsContainingText(String searchText)  寻找包含searchText字符串的元素集。不区分大小写。该文本可能直接出现在该元素中，也可能出现在其子孙元素中。

52、public Elements getElementsContainingOwnText(String searchText)  寻找包含searchText字符串的元素集。不区分大小写。该文本是出现在该元素中。而不是其子孙元素中。

53、public Elements getElementsMatchingText(Pattern pattern)  寻找文本匹配给定正则表达式的元素集。

54、public Elements getElementsMatchingText(String regex)  寻找文本匹配给定正则表达式的元素集。

55、public Elements getElementsMatchingOwnText(Pattern pattern)   寻找自身文本匹配给定正则表达式的元素集。

56、public Elements getElementsMatchingOwnText(String regex)  寻找自身文本匹配给定正则表达式的元素集。

57、public Elements getAllElements()  寻找所有元素集，包含自身和其子孙。

58、public String text()   得到该元素文本和和其子孙文本的结合。如：<p>Hello <b>there</b> now!</p>   p.text()会返回"Hello there now!"

59、public String ownText()  得到该元素自身的文本。如：<p>Hello <b>there</b> now!</p> p.ownText()会返回"Hello now!"

60、public Element text(String text)  设置元素的文本内容。之前任何存在的文本都会被清除掉。

61、public boolean hasText()  测试该元素是否还有非空格的文本内容。

62、public String data()   得到元素的数据结合。数据可以是脚本里面的。

63、public String className()    得到元素class属性的文本内容  可能包含多个class names，用空格分开。如;<div class="header gray"> 返回"header gray"。

64、public Set<String> classNames()  得到元素的所有class names，如;<div class="header gray">,返回"header", "gray"的set集合

65、public Element classNames(Set<String> classNames)  用提供的class names设置元素的class属性。

66、public boolean hasClass(String className)  测试元素是否含有一个指定的class，不区分大小写。

67、public Element addClass(String className)  给元素的class属性增加一个className值。

68、public Element removeClass(String className)  从元素的class属性中移除指定值。

69、public Element toggleClass(String className)  反转class属性的className值，有则移除，没有则新增。

70、public String val() 得到表单元素的值

71、public Element val(String value)  设置表单元素的值

72、public String html()  检索元素的内部html

73、public Element html(String html)  设置元素内部html，首先会清除存在的。


--------------------------------------
原文链接:https://blog.csdn.net/u010142437/article/details/18802873