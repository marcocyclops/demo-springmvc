筆記

## 在 IDEA 中手動創建 Maven-based Webapp

開發環境：   
IDEA 2022   
JDK 17  
TOMCAT 9.0.73

主要經㱘的三個配置文件：  
pom.xml > web.xml > servletname-servlet.xml
```
pom.xml 是 Maven 使用的配置文件, 
用以管理 dependencies，構建及打包 Java-based 專案。
```
```
web.xml 用於 Java Web 應用程式中，
定義 servlets 及 filters 等應用程式要用的配置。
```
```
xxx-servlet.xml 用作配置控制器要使用到的一些資訊。
如註解方式註冊的控制器，需要用到的自動掃描組件，
以及視圖解析器的配置等等。
可在 web.xml 的 init-param 中自訂位置與名稱。
```

步驟如下：  

1. 在 IDEA 中創建空專案 (Empty Project)
   1. name: 專案名稱
   2. Location: 專案儲存位置


2. 於專案中創建新的 Maven-based module (不使用 Maven Archetype)
   1. name: 模組名稱
   2. Location: 專案儲存位置 (自動生成模組位置)
   3. Language: 選 Java
   4. Build system: 選 Maven
   5. JDK: 此專案選了 17
   6. Parent: None
   7. GroupId: com.xxx.yyy
   8. ArtifactId: 通常跟模組名稱


3. 設置模組的 maven pom.xml
   1. 添加 \<packaging> 標簽，并設置為 war 包
   2. 添加 \<maven.compiler.source> 與 <maven.compiler.target> 標簽  
      \<maven.compiler.source> 告訴 maven 你寫的源代碼是用哪個版本的 Java  
      \<maven.compiler.target> 告訴 maven 要將你的源代碼編譯成哪一個版本的 Java Bytecode  
      若果一開始新建的專案不是 Empty Project，而是 New Project 而又選擇了 Maven，IDEA 會自動添加以下代碼
      ```
      <properties>
          <maven.compiler.source>17</maven.compiler.source>
          <maven.compiler.target>17</maven.compiler.target>
      </properties>
      ```
      如果沒有配置，會(可能)報錯：  
      ```agsl
      java: error: release version 5 not supported

      Module springMVC-demo1 SDK 17 does not support source version 1.5.
      Possible solutions:
         
      Downgrade Project SDK in settings to 1.5 or compatible. Open project settings.
      Upgrade language version in Maven build file to 17. Update pom.xml and reload the project.
      ```
   3. 添加 \<dependencies> 標簽, 并引入依賴
      ```
      其中依賴 spring-webmvc 5.3.25 必須與 javax.servlet-api 4.0.1配套，
      具體組合可能因應版本迭代而有所改變
      試過用 spring-webmvc 6^ 配合 javax.servlet-api 4.0.1 使用，觸發錯誤：
      
      'org.springframework.web.servlet.DispatcherServlet' 
      is not assignable to 'javax.servlet.Servlet,jakarta.servlet.Servlet'
      ```
   4. ***重要：每次更新完 pom.xml 都要 Reload Maven 一下, 不然下面步驟的 Web Facets 不會出現，也不會有 Spring


4. 將新增的模組設置為 web 模組
   1. 創建新資料夾 ./src/main/webapp
   2. 通過 IDEA 新增 web.xml 配置文件 (完成後新增 ./webapp/WEB-INFO/web.xml)
      ```
      Project structure > Module > Web > Deployment descriptor > +
      若果在 Module 下沒看到 web (webapp 資料夾沒有藍點), 
      必須先在 Facets 下新增 Web Facets, 并添加給該模組
      ```
      
5. 配置 web.xml  
   配置 SpringMVC 的前端控制器 DispatcherServlet，對瀏覽器發送的請求進行統一處理  
   其中 \<url-pattern> 配置 SpringMVC 的核心控制器所能處理的請求的請求路徑  
   / 所匹配的請求可以是 /login 或 .html 或 .js 或 .css 方式的請求路徑  
   但是 / 不能匹配 .jsp 請求路徑的請求。  
   .jsp 本身就是一個 servlet，需要伺服器中特別指定的 servlet 來處理。  
   所以這裡不能用 /* 而是用 / 來匹配。

   1. 默認配置方式  
      默認配置方式的 DispatcherServlet 配置文件，位於 WEB-INF 資料夾下  
      默認文件名稱為 servletname-servlet.xml  
      例如： 以下配置必需在 WEB-INF 資料夾下創建 springMVC-servlet.xml 來做控制器配置
      ```agsl
      <servlet>
          <servlet-name>springMVC</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      </servlet>
      <servlet-mapping>
          <servlet-name>springMVC</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
      ```
   2. 擴展配置方式
      1. 使用 \<init-param> 來指定 DispatcherServlet 配置文件的位置和名稱
      2. \<param-name> 必需是 contextConfigLocation， SpringMVC 才知道你配置的是 DispatcherServlet 配置文件的位置和名稱
      3. \<param-value> 中的 classpath: = spring application context, 默認指向 ./src/main/resources
      2. 必需在以上指定的位置，創建 DispatcherServlet 配置文件 springMVC.xml，文件名可自訂但要與 \<param-value> 中的一樣
      4. 創建時，可於 ./src/main/resources 右鍵 > new > XML Configuration File > Spring Config
      ```agsl
      <servlet>
          <servlet-name>springMVC</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springMVC.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>  <!-- 將 DispatcherServlet 的初始化時間提前到服務器啟動時 -->
      </servlet>
      <servlet-mapping>
          <servlet-name>springMVC</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
      ```


6. 創建及實作控制器 controller class
   1. 通過 IDEA 新建 class: ./src/main/java/com/xxx/xxx/controllers/XxxController
      ```agsl
      package com.marco.mvc.controller;
   
      import org.springframework.stereotype.Controller;
      import org.springframework.web.bind.annotation.RequestMapping;
   
      @Controller
      public class HelloController {
   
         @RequestMapping("/")
         public String index() {
            return "index";
         }
      }
      ```
   2. SpringMVC 并不知道 controller class 的存在，需要做關聯，有兩種方法：
      1. 在 springMVC.xml 控制器配置文件中用 bean 標簽注冊
      2. 直接在 controller class 前面添加 comment directive @controller 註解方式， 將控制器添加到 IOC 容器成為其組件
         - 添加 comment directive 註解後，還需要進行掃描 SpringMVC 才會知道控制器的存在
         - 需要掃描，就必需在 springMVC 中註冊


7. 配置 DispatcherServlet (springMVC.xml)  
   1. 註冊自動掃描組件，需要用到 context:component-scan 標簽  
      要使用此標簽，必需在 HTML 中添加以下命名空間
      ```agsl
      xmlns:context="http://www.springframework.org/schema/context"
      ```
   2. 另外，原本經 IDEA 生成 HTML 的 xsi:schemaLocation 只有前兩項，  
      要將 context 以及 spring-context 添加到 xsi:schemaLocation, 否則無法運作
      ```agsl
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       https://www.springframework.org/schema/context/spring-context.xsd
       ">
      ```
   完整 springMVC.xml 例子：
   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans 
                          http://www.springframework.org/schema/beans/spring-beans.xsd 
                          http://www.springframework.org/schema/context 
                          https://www.springframework.org/schema/context/spring-context.xsd"
   >

      <!-- 自動掃描組件（以註解方式註冊的控制器） -->
      <context:component-scan base-package="com.marco.mvc.controller"></context:component-scan>

      <!-- 配置 Thymeleaf 視圖解析器-->
      <bean id="viewResolver" class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
         <property name="order" value="1" />
         <property name="characterEncoding" value="UTF-8" />
         <property name="templateEngine">
            <bean class="org.thymeleaf.spring4.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">
                        <property name="prefix" value="/WEB-INF/templates/" />
                        <property name="suffix" value=".html" />
                        <property name="templateMode" value="HTML5" />
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
         </property>
      </bean>
   </beans>
   ```


8. 配置視圖解析器
   1. 當請求經控制器處理完，準備回應給瀏覽器時，需要將回應內容轉為視圖，就必需經視圖解析器解析
   2. 同樣，解析器需要在 springMVC.xml 中配置 (如上 7)


9. 根據視圖解析器裡 prefix 設置的模板路徑創建模板，如： /WEB-INF/templates/
   1. Thymeleaf 模板需要在 html 標簽裡添加命名空間屬性，Thymeleaf xmlns:th="http://www.thymeleaf.org"
      ```agsl
      <html lang="en" xmlns:th="http://www.thymeleaf.org">
      <body></body>
      </html>
      ```


10. 創建完模板後，要在控制器的方法裡返回視圖名稱才能正常工作
    1. 一個控制器可以有多個方法
    2. 視圖名稱其實就是模板路徑名稱，去掉視圖解析器配置裡的前置與後置後的字符串
    2. 方法名稱不需要和請求路徑相同
    3. 方法返回視圖名稱即可
    4. 方法前面需要使用註解與請求路徑匹配 @RequestMapping("/")，由此決定請求路徑使用哪個控制器方法


11. 要在瀏覽器調試必需先配置 Tomcat 伺服器
    1. Edit Configurations > + Tomcat Server > Local 添加伺服器
    2. Server > configuration 配置伺服器
    3. Deployment > select an artifact  配置佈署實體 (模組)

    
## 遇到的問題  
1. pom.xml dependency spring-webmvc 設為 6.0.5 時，報錯： 
   ```
   'org.springframework.web.servlet.DispatcherServlet' 
   is not assignable to 'javax.servlet.Servlet,jakarta.servlet.Servlet'
   ```
   Solved: 使用穩定版本 5.3.15 或 5.3.25, 不要用 6^


2. 運行報錯：
   ```   
   java: error: release version 5 not supported
   
   Module springMVC-demo1 SDK 17 does not support source version 1.5.
   Possible solutions:
   
   Downgrade Project SDK in settings to 1.5 or compatible. 
   Open project settings.
   Upgrade language version in Maven build file to 17. 
   Update pom.xml and reload the project.
   ```
   Solved: 在 pom.xml 中加入以下代碼
   ```agsl
   <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
   ```

