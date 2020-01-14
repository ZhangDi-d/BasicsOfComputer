## Selenium2（java）启动常用浏览器
### 默认启动firefox浏览器
```java
Webdriver driver = new FirefoxDriver();
```
 

### 启动谷歌浏览器

#### 配置chromedriver
```java
WebDriver driver;
System.setProperty("webdriver.chrome.driver", chromedriver_path);
driver = new ChromeDriver();
```

#### 修改User-Agent来伪装浏览器访问手机站点

有时候为了测试需要，可能需要使用测试手机wap这样的站点，如果用真正的手机去测试也可以实现，但是比较麻烦，我们可以通过设置chrome的user agent来伪装浏览器，达到我们的测试目的。

具体实现代码：

```java
public static void main(String[] args) {
    //设置webdriver.chrome.driver属性
    System.setProperty("webdriver,chrome.driver", "ddriver/chromedriver.exe");
    //声明chromeoptions,主要是给chrome设置参数
    ChromeOptions options = new ChromeOptions();
    //设置user agent为iphone5
    options.addArguments("--user-agent=Apple Iphone 5");
    //实例化chrome对象，并加入选项
    WebDriver driver = new ChromeDriver(options);
    //打开百度
    driver.get("https://www.baidu.com");
    try{
    Thread.sleep(5000);
    }catch(InterruptedException e) {
    e.printStackTrace();
    }
    ddriver.quit();
}
```
 

###启动IE浏览器

#### 配置iedriver

```java
WebDriver driver;
System.setProperty("webdriver.ie.driver", iedriver);
//IE的常规设置，便于执行自动化测试
DesiredCapabilities ieCapabilities = DesiredCapabilities.internetExplorer();
ieCapabilities.setCapability(InternetExplorerDriver.INTRODUCE_FLAKINESS_BY_IGNORING_SECURITY_DOMAINS, true);
driver= new InternetExplorerDriver(ieCapabilities);
```

---------------
转载自:https://www.cnblogs.com/sundalian/p/5158855.html

更多: 
How to change useragent-string in runtime chromedriver selenium:
https://stackoverflow.com/questions/50375628/how-to-change-useragent-string-in-runtime-chromedriver-selenium

Set user agent using Selenium WebDriver C# and Ruby:
https://yizeng.me/2013/08/10/set-user-agent-using-selenium-webdriver-c-and-ruby/
