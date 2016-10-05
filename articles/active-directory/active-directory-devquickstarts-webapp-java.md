<properties
	pageTitle="開始使用 Azure AD Java | Microsoft Azure"
	description="如何建置可使用工作或學校帳戶登入使用者的 Java Web 應用程式。"
	services="active-directory"
	documentationCenter="java"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
  ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="brandwe"/>


# 使用 Azure AD 進行 Java Web 應用程式登入與登出

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Azure AD 讓您外包 Web 應用程式的身分識別管理變得既簡單又直接，只需幾行的程式碼便可提供單一登入和登出。在 Java Web 應用程式中，您可以使用 Microsoft 的 ADAL4J 社群導向實作來完成這項作業。

  我們將在此處使用 ADAL4J 來執行下列動作：
- 使用 Azure AD 做為身分識別提供者，將使用者登入應用程式。
- 顯示使用者的一些相關資訊。
- 讓使用者登出 App。

為執行此作業，您必須執行下列動作：

1. 向 Azure AD 註冊應用程式
2. 設定您的應用程式以使用 ADAL4J 程式庫。
3. 使用 ADAL4J 程式庫向 Azure AD 發出登入和登出要求。
4. 列印出使用者的相關資料。

若要開始使用，請[下載應用程式基本架構](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/skeleton.zip)或[下載完整的範例](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect\/archive/complete.zip)。您還需要一個可以註冊應用程式的 Azure AD 租用戶。如果您還沒有租用戶，[了解如何取得租用戶](active-directory-howto-tenant.md)。

## 1\.向 Azure AD 註冊應用程式
若要啟用應用程式來驗證使用者，您必須先要在您的租用戶中註冊這個新的應用程式。

- 登入 Azure 管理入口網站。
- 在左側導覽中按一下 **Active Directory**。
- 選取您要註冊應用程式的租用戶。
- 按一下 [**應用程式**] 索引標籤，然後按一下最下面抽屜的 [新增]。
- 遵照提示進行，並建立新的 **Web 應用程式和/或 WebAPI**。
    - 應用程式的 [**名稱**] 將對使用者說明您的應用程式
    - [**登入 URL**] 是指應用程式的基底 URL。基本架構的預設值是 `http://localhost:8080/adal4jsample/`。
    - [**應用程式識別碼 URI**] 是指應用程式的唯一識別碼。慣例會使用 `https://<tenant-domain>/<app-name>`，例如：`http://localhost:8080/adal4jsample/`
- 完成註冊後，AAD 會為您的應用程式指派一個唯一用戶端識別碼。您在後續章節中將會用到這個值，所以請從 [設定] 索引標籤中複製此值。

進入入口網站後，為您的應用程式建立「應用程式密碼」並複製下來。稍後您將會用到此資訊。


## 2\.使用 Maven 設定您的應用程式以使用 ADAL4J 程式庫和必要條件
在這裡，我們將設定 ADAL4J 以使用 OpenID Connect 驗證通訊協定。ADAL4J 將用來發出登入和登出要求、管理使用者的工作階段，以及取得使用者相關資訊等其他作業。

-	在您的專案根目錄中，開啟/建立 `pom.xml` 並找到 `// TODO: provide dependencies for Maven`，然後以下列取代：

```Java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.microsoft.azure</groupId>
	<artifactId>adal4jsample</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>adal4jsample</name>
	<url>http://maven.apache.org</url>
	<properties>
		<spring.version>3.0.5.RELEASE</spring.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>com.microsoft.azure</groupId>
			<artifactId>adal4j</artifactId>
			<version>1.1.1</version>
		</dependency>
		<dependency>
			<groupId>com.nimbusds</groupId>
			<artifactId>oauth2-oidc-sdk</artifactId>
			<version>4.5</version>
		</dependency>
		<dependency>
			<groupId>org.json</groupId>
			<artifactId>json</artifactId>
			<version>20090211</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.5</version>
		</dependency>
		<!-- Spring 3 dependencies -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
	</dependencies>

	<build>
		<finalName>sample-for-adal4j</finalName>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.4</version>
				<configuration>
					<warName>${project.artifactId}</warName>
					<source>${project.basedir}\src</source>
					<target>${maven.compiler.target}</target>
					<encoding>utf-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<executions>
					<execution>
						<id>install</id>
						<phase>install</phase>
						<goals>
							<goal>sources</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.5</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>

```


## 3\.建立 Java Web 應用程式檔案 (WEB-INF)

在這裡，我們將設定 Java Web 應用程式使用 OpenID Connect 驗證通訊協定。ADAL4J 程式庫將用來發出登入和登出要求、管理使用者的工作階段，以及取得使用者相關資訊等其他作業。

-	若要開始，請開啟 `\webapp\WEB-INF` 下的 `web.xml` 檔案，並在 XML 中輸入應用程式的組態值。

檔案看起來應該如下所示：

```xml
<?xml version="1.0"?>
<web-app id="WebApp_ID" version="2.4"
	xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
	<display-name>Archetype Created Web Application</display-name>
	<context-param>
		<param-name>authority</param-name>
		<param-value>https://login.windows.net/</param-value>
	</context-param>
	<context-param>
		<param-name>tenant</param-name>
		<param-value>YOUR_TENANT_NAME</param-value>
	</context-param>

	<filter>
		<filter-name>BasicFilter</filter-name>
		<filter-class>com.microsoft.aad.adal4jsample.BasicFilter</filter-class>
		<init-param>
			<param-name>client_id</param-name>
			<param-value>YOUR_CLIENT_ID</param-value>
		</init-param>
		<init-param>
			<param-name>secret_key</param-name>
			<param-value>YOUR_CLIENT_SECRET</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>BasicFilter</filter-name>
		<url-pattern>/secure/*</url-pattern>
	</filter-mapping>

	<servlet>
		<servlet-name>mvc-dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>mvc-dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
</web-app>

```


    -	The `YOUR_CLIENT_ID` is the **Application Id** assigned to your app in the registration portal.
    -	The `YOUR_CLIENT_SECRET` is the **Application Secret**  you created in the portal.
    - The `YOUR_TENANT_NAME` is the **tenant name** of your app, e.g. contoso.onmicrosoft.com

接受其餘的組態參數。

> [AZURE.NOTE]
您可以從 XML 檔案中看到，我們要編寫一個名為 `mvc-dispatcher` 的 JSP/Servlet Web 應用程式，該應用程式會在我們瀏覽 /secure URL 時使用 `BasicFilter`。您會在我們編寫的其餘相同程式碼中看到我們使用 /secure 做為受保護內容的所在的位置，並且將會強制向 Azure Active Directory 進行驗證。

-	接下來，在 `\webapp\WEB-INF` 下建立 `mvc-dispatcher-servlet.xml` 檔案，並輸入下列：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans     
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

	<context:component-scan base-package="com.microsoft.aad.adal4jsample" />

	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix">
			<value>/</value>
		</property>
		<property name="suffix">
			<value>.jsp</value>
		</property>
	</bean>

</beans>
```

這會告知 Web 應用程式使用 Spring 以及到哪裡尋找我們在以下撰寫的 .jsp 檔案。

## 4\.建立 Java JSP 檢視檔案 (適用於 BasicFilter MVC)

我們在 WEB-INF 中 Web 應用程式的設定目前只完成了一半。接下來，我們必須建立 Web 應用程式將會執行的實際 Java Server Pages 檔案 (已經在組態中提示)。

如果您記得，我們已在 XML 組態檔案中告知 Java，我們有個應該載入 .jsp 檔案的 `/` 資源，以及應該通過稱為 `BasicFilter` 之篩選器的 `/secure` 資源。

我們現在來製作那些項目。

-	首先，請在 `\webapp` 下建立 `index.jsp` 檔案，並剪下/貼上下列內容：

```jsp
<html>
<body>
	<h2>Hello World!</h2>
	<ul>
	<li><a href="secure/aad">Secure Page</a></li>
	</ul>
</body>
</html>

```

這只會重新導向至我們的篩選條件保護的安全的頁面。

- 接下來，在相同的目錄中建立 `error.jsp` 檔案，以攔截可能會發生的任何錯誤：

```jsp
<html>
<body>
	<h2>ERROR PAGE!</h2>
	<p>
		Exception -
		<%=request.getAttribute("error")%></p>
	<ul>
		<li><a href="<%=request.getContextPath()%>/index.jsp">Go Home</a></li>
	</ul>
</body>
</html>
```

- 最後，在 `\webapp` 下建立稱為 `\secure` 的資料夾，以建立我們需要的安全網頁，如此一來，目錄即為 `\webapp\secure`。

- 在此目錄內，接著建立 `aad.jsp` 檔案，並剪下/貼上下列內容：

```jsp
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>AAD Secure Page</title>
</head>
<body>

	<h1>Directory - Users List</h1>
	<p>${users}</p>

	<ul>
		<li><a href="<%=request.getContextPath()%>/secure/aad?cc=1">Get
				new Access Token via Client Credentials</a></li>
	</ul>
	<ul>
		<li><a href="<%=request.getContextPath()%>/secure/aad?refresh=1">Get
				new Access Token via Refresh Token</a></li>
	</ul>
	<ul>
		<li><a href="<%=request.getContextPath()%>/index.jsp">Go Home</a></li>
	</ul>
</body>
</html>
```

您會看到此頁面會重新導向至我們的 BasicFilter Servlet 將會讀取的特定要求，並接著使用 `ADAJ4J` 程式庫執行。非常簡單吧？

當然，我們現在需要設定我們的 Java 檔案，以便 Servlet 可以執行其工作。

## 5\.建立一些 Java 協助程式檔案 (適用於 BasicFilter MVC)

我們的目標是要建立將一些 Java 檔案，這些檔案能夠：

1. 允許使用者登入和登出
2. 取得使用者的一些相關資料。

> [AZURE.NOTE] 
為了取得使用者的相關資料，我們必須使用來自 Azure Active Directory 的圖形 API。圖形 API 是安全的 Web 服務，可讓您用來擷取有關組織 (包括個別使用者) 的資料。這優於預先在權杖中填入機密資料，因為它可確保要求資料的使用者已獲授權，而且可能抓取權杖的任何人 (從越獄電話或桌面上的 Web 瀏覽器快取) 都不會取得一堆有關使用者或組織的重要詳細資料。

讓我們編寫一些 Java 檔案，為我們執行這項工作：

1. 在稱為 'adal4jsample' 的根目錄中建立資料夾來儲存我們所有的 Java 檔案。

我們將在 Java 檔案中使用命名空間 `com.microsoft.aad.adal4jsample`。大部分的 IDE 會為此建立巢狀的資料夾結構 (例如 `/com/microsoft/aad/adal4jsample`)。您可以隨意這樣做，但不必要。

2. 在這個資料夾內，建立名為 `JSONHelper.java` 的檔案，我們會用它來幫助剖析來自我們的權杖的 JSON 資料。您可以從下面剪下/貼上：

```Java

package com.microsoft.aad.adal4jsample;

import java.lang.reflect.Field;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.apache.commons.lang3.text.WordUtils;
import org.apache.log4j.Logger;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

/**
 * This class provides the methods to parse JSON Data from a JSON Formatted
 * String.
 * 
 * @author Azure Active Directory Contributor
 * 
 */
public class JSONHelper {

    private static Logger logger = Logger.getLogger(JSONHelper.class);

    JSONHelper() {
        // PropertyConfigurator.configure("log4j.properties");
    }

    /**
     * This method parses an JSON Array out of a collection of JSON Objects
     * within a string.
     * 
     * @param jSonData
     *            The JSON String that holds the collection.
     * @return An JSON Array that would contains all the collection object.
     * @throws Exception
     */
    public static JSONArray fetchDirectoryObjectJSONArray(JSONObject jsonObject) throws Exception {
        JSONArray jsonArray = new JSONArray();
        jsonArray = jsonObject.optJSONObject("responseMsg").optJSONArray("value");
        return jsonArray;
    }

    /**
     * This method parses an JSON Object out of a collection of JSON Objects
     * within a string
     * 
     * @param jsonObject
     * @return An JSON Object that would contains the DirectoryObject.
     * @throws Exception
     */
    public static JSONObject fetchDirectoryObjectJSONObject(JSONObject jsonObject) throws Exception {
        JSONObject jObj = new JSONObject();
        jObj = jsonObject.optJSONObject("responseMsg");
        return jObj;
    }

    /**
     * This method parses the skip token from a json formatted string.
     * 
     * @param jsonData
     *            The JSON Formatted String.
     * @return The skipToken.
     * @throws Exception
     */
    public static String fetchNextSkiptoken(JSONObject jsonObject) throws Exception {
        String skipToken = "";
        // Parse the skip token out of the string.
        skipToken = jsonObject.optJSONObject("responseMsg").optString("odata.nextLink");

        if (!skipToken.equalsIgnoreCase("")) {
            // Remove the unnecessary prefix from the skip token.
            int index = skipToken.indexOf("$skiptoken=") + (new String("$skiptoken=")).length();
            skipToken = skipToken.substring(index);
        }
        return skipToken;
    }

    /**
     * @param jsonObject
     * @return
     * @throws Exception
     */
    public static String fetchDeltaLink(JSONObject jsonObject) throws Exception {
        String deltaLink = "";
        // Parse the skip token out of the string.
        deltaLink = jsonObject.optJSONObject("responseMsg").optString("aad.deltaLink");
        if (deltaLink == null || deltaLink.length() == 0) {
            deltaLink = jsonObject.optJSONObject("responseMsg").optString("aad.nextLink");
            logger.info("deltaLink empty, nextLink ->" + deltaLink);

        }
        if (!deltaLink.equalsIgnoreCase("")) {
            // Remove the unnecessary prefix from the skip token.
            int index = deltaLink.indexOf("deltaLink=") + (new String("deltaLink=")).length();
            deltaLink = deltaLink.substring(index);
        }
        return deltaLink;
    }

    /**
     * This method would create a string consisting of a JSON document with all
     * the necessary elements set from the HttpServletRequest request.
     * 
     * @param request
     *            The HttpServletRequest
     * @return the string containing the JSON document.
     * @throws Exception
     *             If there is any error processing the request.
     */
    public static String createJSONString(HttpServletRequest request, String controller) throws Exception {
        JSONObject obj = new JSONObject();
        try {
            Field[] allFields = Class.forName(
                    "com.microsoft.windowsazure.activedirectory.sdk.graph.models." + controller).getDeclaredFields();
            String[] allFieldStr = new String[allFields.length];
            for (int i = 0; i < allFields.length; i++) {
                allFieldStr[i] = allFields[i].getName();
            }
            List<String> allFieldStringList = Arrays.asList(allFieldStr);
            Enumeration<String> fields = request.getParameterNames();

            while (fields.hasMoreElements()) {

                String fieldName = fields.nextElement();
                String param = request.getParameter(fieldName);
                if (allFieldStringList.contains(fieldName)) {
                    if (param == null || param.length() == 0) {
                        if (!fieldName.equalsIgnoreCase("password")) {
                            obj.put(fieldName, JSONObject.NULL);
                        }
                    } else {
                        if (fieldName.equalsIgnoreCase("password")) {
                            obj.put("passwordProfile", new JSONObject("{"password": "" + param + ""}"));
                        } else {
                            obj.put(fieldName, param);

                        }
                    }
                }
            }
        } catch (JSONException e) {
            e.printStackTrace();
        } catch (SecurityException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return obj.toString();
    }

    /**
     * 
     * @param key
     * @param value
     * @return string format of this JSON obje
     * @throws Exception
     */
    public static String createJSONString(String key, String value) throws Exception {

        JSONObject obj = new JSONObject();
        try {
            obj.put(key, value);
        } catch (JSONException e) {
            e.printStackTrace();
        }

        return obj.toString();
    }

    /**
     * This is a generic method that copies the simple attribute values from an
     * argument jsonObject to an argument generic object.
     * 
     * @param jsonObject
     *            The jsonObject from where the attributes are to be copied.
     * @param destObject
     *            The object where the attributes should be copied into.
     * @throws Exception
     *             Throws a Exception when the operation are unsuccessful.
     */
    public static <T> void convertJSONObjectToDirectoryObject(JSONObject jsonObject, T destObject) throws Exception {

        // Get the list of all the field names.
        Field[] fieldList = destObject.getClass().getDeclaredFields();

        // For all the declared field.
        for (int i = 0; i < fieldList.length; i++) {
            // If the field is of type String, that is
            // if it is a simple attribute.
            if (fieldList[i].getType().equals(String.class)) {
                // Invoke the corresponding set method of the destObject using
                // the argument taken from the jsonObject.
                destObject
                        .getClass()
                        .getMethod(String.format("set%s", WordUtils.capitalize(fieldList[i].getName())),
                                new Class[] { String.class })
                        .invoke(destObject, new Object[] { jsonObject.optString(fieldList[i].getName()) });
            }
        }
    }

    public static JSONArray joinJSONArrays(JSONArray a, JSONArray b) {
        JSONArray comb = new JSONArray();
        for (int i = 0; i < a.length(); i++) {
            comb.put(a.optJSONObject(i));
        }
        for (int i = 0; i < b.length(); i++) {
            comb.put(b.optJSONObject(i));
        }
        return comb;
    }

}

```

3. 下一步，建立名為 `HttpClientHelper.java` 的檔案，我們會用它來幫助剖析來自我們的 AAD 端點的 HTTP 資料。您可以從下面剪下/貼上：

```Java

package com.microsoft.aad.adal4jsample;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.HttpURLConnection;

import org.json.JSONException;
import org.json.JSONObject;

/**
 * This is Helper class for all RestClient class.
 * 
 * @author Azure Active Directory Contributor
 * 
 */
public class HttpClientHelper {

    public HttpClientHelper() {
        super();
    }

    public static String getResponseStringFromConn(HttpURLConnection conn, boolean isSuccess) throws IOException {

        BufferedReader reader = null;
        if (isSuccess) {
            reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        } else {
            reader = new BufferedReader(new InputStreamReader(conn.getErrorStream()));
        }
        StringBuffer stringBuffer = new StringBuffer();
        String line = "";
        while ((line = reader.readLine()) != null) {
            stringBuffer.append(line);
        }

        return stringBuffer.toString();
    }

    public static String getResponseStringFromConn(HttpURLConnection conn, String payLoad) throws IOException {

        // Send the http message payload to the server.
        if (payLoad != null) {
            conn.setDoOutput(true);
            OutputStreamWriter osw = new OutputStreamWriter(conn.getOutputStream());
            osw.write(payLoad);
            osw.flush();
            osw.close();
        }

        // Get the message response from the server.
        BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        String line = "";
        StringBuffer stringBuffer = new StringBuffer();
        while ((line = br.readLine()) != null) {
            stringBuffer.append(line);
        }

        br.close();

        return stringBuffer.toString();
    }

    public static byte[] getByteaArrayFromConn(HttpURLConnection conn, boolean isSuccess) throws IOException {

        InputStream is = conn.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buff = new byte[1024];
        int bytesRead = 0;

        while ((bytesRead = is.read(buff, 0, buff.length)) != -1) {
            baos.write(buff, 0, bytesRead);
        }

        byte[] bytes = baos.toByteArray();
        baos.close();
        return bytes;
    }

    /**
     * for bad response, whose responseCode is not 200 level
     * 
     * @param responseCode
     * @param errorCode
     * @param errorMsg
     * @return
     * @throws JSONException
     */
    public static JSONObject processResponse(int responseCode, String errorCode, String errorMsg) throws JSONException {
        JSONObject response = new JSONObject();
        response.put("responseCode", responseCode);
        response.put("errorCode", errorCode);
        response.put("errorMsg", errorMsg);

        return response;
    }

    /**
     * for bad response, whose responseCode is not 200 level
     * 
     * @param responseCode
     * @param errorCode
     * @param errorMsg
     * @return
     * @throws JSONException
     */
    public static JSONObject processGoodRespStr(int responseCode, String goodRespStr) throws JSONException {
        JSONObject response = new JSONObject();
        response.put("responseCode", responseCode);
        if (goodRespStr.equalsIgnoreCase("")) {
            response.put("responseMsg", "");
        } else {
            response.put("responseMsg", new JSONObject(goodRespStr));
        }

        return response;
    }

    /**
     * for good response
     * 
     * @param responseCode
     * @param responseMsg
     * @return
     * @throws JSONException
     */
    public static JSONObject processBadRespStr(int responseCode, String responseMsg) throws JSONException {

        JSONObject response = new JSONObject();
        response.put("responseCode", responseCode);
        if (responseMsg.equalsIgnoreCase("")) { // good response is empty string
            response.put("responseMsg", "");
        } else { // bad response is json string
            JSONObject errorObject = new JSONObject(responseMsg).optJSONObject("odata.error");

            String errorCode = errorObject.optString("code");
            String errorMsg = errorObject.optJSONObject("message").optString("value");
            response.put("responseCode", responseCode);
            response.put("errorCode", errorCode);
            response.put("errorMsg", errorMsg);
        }

        return response;
    }

}

```

## 6\.建立 Java 圖形 API 模型檔案 (適用於 BasicFilter MVC)

如以上所述，我們將使用圖形 API 來取得有關所登入使用者的資料。為了讓我們順利進行，我們應該建立一個代表**目錄物件**的檔案以及一個代表**使用者**的個別檔案，如此便可以使用 Java 的 OO 模式。

1. 建立名為 `DirectoryObject.java` 的檔案，我們將用它來儲存有關任何 DirectoryObject 的基本資料 (您稍後可以隨意使用它執行任何其他圖形查詢)。您可以從下面剪下/貼上：

```Java

package com.microsoft.aad.adal4jsample;

/**
 * @author Azure Active Directory Contributor
 *
 */
public abstract class DirectoryObject {
	
	public DirectoryObject() {
		super();
	}
	
	/**
	 * 
	 * @return
	 */
	public abstract String getObjectId();
	
	/**
	 * @param objectId
	 */
	public abstract void setObjectId(String objectId);

	/**
	 * 
	 * @return
	 */
	public abstract String getObjectType();

	/**
	 * 
	 * @param objectType
	 */
	public abstract void setObjectType(String objectType);
	
	/**
	 * 
	 * @return
	 */
	public abstract String getDisplayName();

	/**
	 * 
	 * @param displayName
	 */
	public abstract void setDisplayName(String displayName);

}

```

2. 建立名為 `User.java` 的檔案，我們將用它來儲存有關目錄中的任何使用者的基本資料。重申，這是用於目錄資料的基本 getter/setter，因此您可以從下面剪下/貼上：

```Java

package com.microsoft.aad.adal4jsample;

import java.security.acl.Group;
import java.util.ArrayList;

import javax.xml.bind.annotation.XmlRootElement;

import org.json.JSONObject;

/**
 *  The User Class holds together all the members of a WAAD User entity and all the access methods and set methods
 *  @author Azure Active Directory Contributor
 */
@XmlRootElement
public class User extends DirectoryObject{
	
	// The following are the individual private members of a User object that holds
	// a particular simple attribute of an User object.
	protected String objectId;
	protected String objectType;
	protected String accountEnabled;
	protected String city;
	protected String country;
	protected String department;
	protected String dirSyncEnabled;
	protected String displayName;
	protected String facsimileTelephoneNumber;
	protected String givenName;
	protected String jobTitle;
	protected String lastDirSyncTime;
	protected String mail;
	protected String mailNickname;
	protected String mobile;
	protected String password;
	protected String passwordPolicies;
	protected String physicalDeliveryOfficeName;
	protected String postalCode;
	protected String preferredLanguage;
	protected String state;
	protected String streetAddress;
	protected String surname;
	protected String telephoneNumber;
	protected String usageLocation;
	protected String userPrincipalName;
	protected boolean isDeleted;  // this will move to dto

	/**
	 * below 4 properties are for future use
	 */
	// managerDisplayname of this user
	protected String managerDisplayname;
	
	// The directReports holds a list of directReports
	private ArrayList<User> directReports;
	
	// The groups holds a list of group entity this user belongs to. 
	private ArrayList<Group> groups;
	
	// The roles holds a list of role entity this user belongs to. 
	private ArrayList<Group> roles;
	
	
	/**
	 * The constructor for the User class. Initializes the dynamic lists and managerDisplayname variables.
	 */
	public User(){
		directReports = null;
		groups = new ArrayList<Group>();
		roles = new ArrayList<Group>();
		managerDisplayname = null;
	}
//	
//	public User(String displayName, String objectId){
//		setDisplayName(displayName);
//		setObjectId(objectId);
//	}
//	
//	public User(String displayName, String objectId, String userPrincipalName, String accountEnabled){
//		setDisplayName(displayName);
//		setObjectId(objectId);
//		setUserPrincipalName(userPrincipalName);
//		setAccountEnabled(accountEnabled);
//	}
//	

	/**
	 * @return The objectId of this user.
	 */
	public String getObjectId() {
		return objectId;
	}
	
	/**
	 * @param objectId The objectId to set to this User object.
	 */
	public void setObjectId(String objectId) {
		this.objectId = objectId;
	}


	/**
	 * @return The objectType of this User.
	 */
	public String getObjectType() {
		return objectType;
	}

	/**
	 * @param objectType The objectType to set to this User object.
	 */
	public void setObjectType(String objectType) {
		this.objectType = objectType;
	}

	/**
	 * @return The userPrincipalName of this User.
	 */
	public String getUserPrincipalName() {
		return userPrincipalName;
	}

	/**
	 * @param userPrincipalName The userPrincipalName to set to this User object.
	 */
	public void setUserPrincipalName(String userPrincipalName) {
		this.userPrincipalName = userPrincipalName;
	}

	
	/**
	 * @return The usageLocation of this User.
	 */
	public String getUsageLocation() {
		return usageLocation;
	}

	/**
	 * @param usageLocation The usageLocation to set to this User object.
	 */
	public void setUsageLocation(String usageLocation) {
		this.usageLocation = usageLocation;
	}

	/**
	 * @return The telephoneNumber of this User.
	 */
	public String getTelephoneNumber() {
		return telephoneNumber;
	}

	/**
	 * @param telephoneNumber The telephoneNumber to set to this User object.
	 */
	public void setTelephoneNumber(String telephoneNumber) {
		this.telephoneNumber = telephoneNumber;
	}

	/**
	 * @return The surname of this User.
	 */
	public String getSurname() {
		return surname;
	}

	/**
	 * @param surname The surname to set to this User Object.
	 */
	public void setSurname(String surname) {
		this.surname = surname;
	}

	/**
	 * @return The streetAddress of this User.
	 */
	public String getStreetAddress() {
		return streetAddress;
	}

	/**
	 * @param streetAddress The streetAddress to set to this User.
	 */
	public void setStreetAddress(String streetAddress) {
		this.streetAddress = streetAddress;
	}

	/**
	 * @return The state of this User.
	 */
	public String getState() {
		return state;
	}

	/**
	 * @param state The state to set to this User object.
	 */
	public void setState(String state) {
		this.state = state;
	}

	/**
	 * @return The preferredLanguage of this User.
	 */
	public String getPreferredLanguage() {
		return preferredLanguage;
	}

	/**
	 * @param preferredLanguage The preferredLanguage to set to this User.
	 */
	public void setPreferredLanguage(String preferredLanguage) {
		this.preferredLanguage = preferredLanguage;
	}

	/**
	 * @return The postalCode of this User.
	 */
	public String getPostalCode() {
		return postalCode;
	}

	/**
	 * @param postalCode The postalCode to set to this User.
	 */
	public void setPostalCode(String postalCode) {
		this.postalCode = postalCode;
	}

	/**
	 * @return The physicalDeliveryOfficeName of this User.
	 */
	public String getPhysicalDeliveryOfficeName() {
		return physicalDeliveryOfficeName;
	}

	/**
	 * @param physicalDeliveryOfficeName The physicalDeliveryOfficeName to set to this User Object.
	 */
	public void setPhysicalDeliveryOfficeName(String physicalDeliveryOfficeName) {
		this.physicalDeliveryOfficeName = physicalDeliveryOfficeName;
	}

	/**
	 * @return The passwordPolicies of this User.
	 */
	public String getPasswordPolicies() {
		return passwordPolicies;
	}

	/**
	 * @param passwordPolicies The passwordPolicies to set to this User object.
	 */
	public void setPasswordPolicies(String passwordPolicies) {
		this.passwordPolicies = passwordPolicies;
	}

	/**
	 * @return The mobile of this User.
	 */
	public String getMobile() {
		return mobile;
	}

	/**
	 * @param mobile The mobile to set to this User object.
	 */
	public void setMobile(String mobile) {
		this.mobile = mobile;
	}
	
	/**
	 * @return The Password of this User.
	 */
	public String getPassword() {
		return password;
	}

	/**
	 * @param password The mobile to set to this User object.
	 */
	public void setPassword(String password) {
		this.password = password;
	}

	/**
	 * @return The mail of this User.
	 */
	public String getMail() {
		return mail;
	}

	/**
	 * @param mail The mail to set to this User object.
	 */
	public void setMail(String mail) {
		this.mail = mail;
	}
	
	/**
	 * @return The MailNickname of this User.
	 */
	public String getMailNickname() {
		return mailNickname;
	}

	/**
	 * @param mail The MailNickname to set to this User object.
	 */
	public void setMailNickname(String mailNickname) {
		this.mailNickname = mailNickname;
	}


	/**
	 * @return The jobTitle of this User.
	 */
	public String getJobTitle() {
		return jobTitle;
	}

	/**
	 * @param jobTitle The jobTitle to set to this User Object.
	 */
	public void setJobTitle(String jobTitle) {
		this.jobTitle = jobTitle;
	}

	/**
	 * @return The givenName of this User.
	 */
	public String getGivenName() {
		return givenName;
	}

	/**
	 * @param givenName The givenName to set to this User.
	 */
	public void setGivenName(String givenName) {
		this.givenName = givenName;
	}

	/**
	 * @return The facsimileTelephoneNumber of this User.
	 */
	public String getFacsimileTelephoneNumber() {
		return facsimileTelephoneNumber;
	}

	/**
	 * @param facsimileTelephoneNumber The facsimileTelephoneNumber to set to this User Object.
	 */
	public void setFacsimileTelephoneNumber(String facsimileTelephoneNumber) {
		this.facsimileTelephoneNumber = facsimileTelephoneNumber;
	}

	/**
	 * @return The displayName of this User.
	 */
	public String getDisplayName() {
		return displayName;
	}

	/**
	 * @param displayName The displayName to set to this User Object.
	 */
	public void setDisplayName(String displayName) {
		this.displayName = displayName;
	}

	/**
	 * @return The dirSyncEnabled of this User.
	 */
	public String getDirSyncEnabled() {
		return dirSyncEnabled;
	}

	/**
	 * @param dirSyncEnabled The dirSyncEnabled to set to this User.
	 */
	public void setDirSyncEnabled(String dirSyncEnabled) {
		this.dirSyncEnabled = dirSyncEnabled;
	}

	/**
	 * @return The department of this User.
	 */
	public String getDepartment() {
		return department;
	}

	/**
	 * @param department The department to set to this User.
	 */
	public void setDepartment(String department) {
		this.department = department;
	}

	/**
	 * @return The lastDirSyncTime of this User.
	 */
	public String getLastDirSyncTime() {
		return lastDirSyncTime;
	}

	/**
	 * @param lastDirSyncTime The lastDirSyncTime to set to this User.
	 */
	public void setLastDirSyncTime(String lastDirSyncTime) {
		this.lastDirSyncTime = lastDirSyncTime;
	}

	/**
	 * @return The country of this User.
	 */
	public String getCountry() {
		return country;
	}

	/**
	 * @param country The country to set to this User.
	 */
	public void setCountry(String country) {
		this.country = country;
	}

	/**
	 * @return The city of this User.
	 */
	public String getCity() {
		return city;
	}

	/**
	 * @param city The city to set to this User.
	 */
	public void setCity(String city) {
		this.city = city;
	}

	/**
	 * @return The accountEnabled attribute of this User.
	 */
	public String getAccountEnabled() {
		return accountEnabled;
	}

	/**
	 * @param accountEnabled The accountEnabled to set to this User.
	 */
	public void setAccountEnabled(String accountEnabled) {
		this.accountEnabled = accountEnabled;
	}
	
	public boolean isIsDeleted() {
		return this.isDeleted;
	}

	public void setIsDeleted(boolean isDeleted) {
		this.isDeleted = isDeleted;
	}

	@Override
	public String toString() {
		return new JSONObject(this).toString();
	}
	
	public String getManagerDisplayname(){
		return managerDisplayname;
	}
	
	public void setManagerDisplayname(String managerDisplayname){
		this.managerDisplayname = managerDisplayname;
	}
}


/**
 * The Class DirectReports Holds the essential data for a single DirectReport entry. Namely,
 * it holds the displayName and the objectId of the direct entry. Furthermore, it provides the
 * access methods to set or get the displayName and the ObjectId of this entry.
 */
//class DirectReport extends User{
//
//	private String displayName;
//	private String objectId;
//	 
//	/**
//	 * Two arguments Constructor for the DirectReport Class.
//	 * @param displayName
//	 * @param objectId
//	 */
//	public DirectReport(String displayName, String objectId){
//		this.displayName = displayName;
//		this.objectId = objectId;
//	}
//
//	/**
//	 * @return The diaplayName of this direct report entry.
//	 */
//	public String getDisplayName() {
//		return displayName;
//	}
//
//	
//	/**
//	 *  @return The objectId of this direct report entry. 
//	 */
//	public String getObjectId() {
//		return objectId;
//	}
//
//}

```

## 7\.建立驗證模型/控制器檔案 (適用於 BasicFilter)

沒錯，Java 有些累贅，但我們就快完成了。在進行最後一個步驟之前，先編寫更多 `ADAL4J` 程式庫所需的協助程式檔案，再編寫 BasicFilter Servlet 來處理我們的要求。

1. 建立名為 `AuthHelper.java` 的檔案，這可提供我們用來判斷登入的使用者狀態的方法。其中包含：

- `isAuthenticated()` 方法，可傳回使用者是否登入
- `containsAuthenticationData()`，可告訴我們是否權杖是否具有資料
- `isAuthenticationSuccessful()`，可告訴我們是否成功驗證使用者。

剪下/貼上下列程式碼：

```Java
package com.microsoft.aad.adal4jsample;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import com.microsoft.aad.adal4j.AuthenticationResult;
import com.nimbusds.openid.connect.sdk.AuthenticationResponse;
import com.nimbusds.openid.connect.sdk.AuthenticationResponseParser;
import com.nimbusds.openid.connect.sdk.AuthenticationSuccessResponse;

public final class AuthHelper {

    public static final String PRINCIPAL_SESSION_NAME = "principal";

    private AuthHelper() {
    }

    public static boolean isAuthenticated(HttpServletRequest request) {
        return request.getSession().getAttribute(PRINCIPAL_SESSION_NAME) != null;
    }

    public static AuthenticationResult getAuthSessionObject(
            HttpServletRequest request) {
        return (AuthenticationResult) request.getSession().getAttribute(
                PRINCIPAL_SESSION_NAME);
    }

    public static boolean containsAuthenticationData(
            HttpServletRequest httpRequest) {
        Map<String, String[]> map = httpRequest.getParameterMap();
        return httpRequest.getMethod().equalsIgnoreCase("POST") && (httpRequest.getParameterMap().containsKey(
                        AuthParameterNames.ERROR)
                        || httpRequest.getParameterMap().containsKey(
                                AuthParameterNames.ID_TOKEN) || httpRequest
                        .getParameterMap().containsKey(AuthParameterNames.CODE));
    }

    public static boolean isAuthenticationSuccessful(
            AuthenticationResponse authResponse) {
        return authResponse instanceof AuthenticationSuccessResponse;
    }
}
```

2. 建立名為 `AuthParameterNames.java` 的檔案，這可提供我們一些 `ADAL4J` 將會需要的不可變變數。剪下/貼上下列內容：

```Java
package com.microsoft.aad.adal4jsample;

public final class AuthParameterNames {

    private AuthParameterNames() {
    }

    public static String ERROR = "error";
    public static String ERROR_DESCRIPTION = "error_description";
    public static String ERROR_URI = "error_uri";
    public static String ID_TOKEN = "id_token";
    public static String CODE = "code";
}
```

3. 最後，建立名為 `AadController.java` 的檔案，這是我們 MVC 模式的控制器，它將提供我們 JSP 控制器，並公開我們的應用程式 `secure/aad` URL 端點。此外，我們也會將圖形查詢放在這個檔案中。

剪下/貼上下列項目：

```Java
package com.microsoft.aad.adal4jsample;

import java.net.HttpURLConnection;
import java.net.URL;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.json.JSONArray;
import org.json.JSONObject;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.microsoft.aad.adal4j.AuthenticationResult;

@Controller
@RequestMapping("/secure/aad")
public class AadController {

    @RequestMapping(method = { RequestMethod.GET, RequestMethod.POST })
    public String getDirectoryObjects(ModelMap model, HttpServletRequest httpRequest) {
        HttpSession session = httpRequest.getSession();
        AuthenticationResult result = (AuthenticationResult) session.getAttribute(AuthHelper.PRINCIPAL_SESSION_NAME);
        if (result == null) {
            model.addAttribute("error", new Exception("AuthenticationResult not found in session."));
            return "/error";
        } else {
            String data;
            try {
                data = this.getUsernamesFromGraph(result.getAccessToken(), session.getServletContext()
                        .getInitParameter("tenant"));
                model.addAttribute("users", data);
            } catch (Exception e) {
                model.addAttribute("error", e);
                return "/error";
            }
        }
        return "/secure/aad";
    }

    private String getUsernamesFromGraph(String accessToken, String tenant) throws Exception {
        URL url = new URL(String.format("https://graph.windows.net/%s/users?api-version=2013-04-05", tenant,
                accessToken));

        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        // Set the appropriate header fields in the request header.
        conn.setRequestProperty("api-version", "2013-04-05");
        conn.setRequestProperty("Authorization", accessToken);
        conn.setRequestProperty("Accept", "application/json;odata=minimalmetadata");
        String goodRespStr = HttpClientHelper.getResponseStringFromConn(conn, true);
        // logger.info("goodRespStr ->" + goodRespStr);
        int responseCode = conn.getResponseCode();
        JSONObject response = HttpClientHelper.processGoodRespStr(responseCode, goodRespStr);
        JSONArray users = new JSONArray();

        users = JSONHelper.fetchDirectoryObjectJSONArray(response);

        StringBuilder builder = new StringBuilder();
        User user = null;
        for (int i = 0; i < users.length(); i++) {
            JSONObject thisUserJSONObject = users.optJSONObject(i);
            user = new User();
            JSONHelper.convertJSONObjectToDirectoryObject(thisUserJSONObject, user);
            builder.append(user.getUserPrincipalName() + "<br/>");
        }
        return builder.toString();
    }

}

```

## 8\.建立 BasicFilter 檔案 (適用於 BasicFilter MVC)

最後，我們準備要建立 BasicFilter 檔案，以從我們的檢視 (JSP 檔案) 處理我們的要求。

建立名為 `BasicFilter.java` 的檔案，其中包含下列內容：

```Java

package com.microsoft.aad.adal4jsample;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URI;
import java.net.URLEncoder;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

import javax.naming.ServiceUnavailableException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.microsoft.aad.adal4j.AuthenticationContext;
import com.microsoft.aad.adal4j.AuthenticationResult;
import com.microsoft.aad.adal4j.ClientCredential;
import com.nimbusds.oauth2.sdk.AuthorizationCode;
import com.nimbusds.openid.connect.sdk.AuthenticationErrorResponse;
import com.nimbusds.openid.connect.sdk.AuthenticationResponse;
import com.nimbusds.openid.connect.sdk.AuthenticationResponseParser;
import com.nimbusds.openid.connect.sdk.AuthenticationSuccessResponse;

public class BasicFilter implements Filter {

    private String clientId = "";
    private String clientSecret = "";
    private String tenant = "";
    private String authority;

    public void destroy() {

    }

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {

        if (request instanceof HttpServletRequest) {
            HttpServletRequest httpRequest = (HttpServletRequest) request;
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            try {

                String currentUri = request.getScheme()
                        + "://"
                        + request.getServerName()
                        + ("http".equals(request.getScheme())
                                && request.getServerPort() == 80
                                || "https".equals(request.getScheme())
                                && request.getServerPort() == 443 ? "" : ":"
                                + request.getServerPort())
                        + httpRequest.getRequestURI();
                String fullUrl = currentUri
                        + (httpRequest.getQueryString() != null ? "?"
                                + httpRequest.getQueryString() : "");
                // check if user has a session
                if (!AuthHelper.isAuthenticated(httpRequest)) {
                    if (AuthHelper.containsAuthenticationData(httpRequest)) {
                        Map<String, String> params = new HashMap<String, String>();
                        for (String key : request.getParameterMap().keySet()) {
                            params.put(key,
                                    request.getParameterMap().get(key)[0]);
                        }
                        AuthenticationResponse authResponse = AuthenticationResponseParser
                                .parse(new URI(fullUrl), params);
                        if (AuthHelper.isAuthenticationSuccessful(authResponse)) {

                            AuthenticationSuccessResponse oidcResponse = (AuthenticationSuccessResponse) authResponse;
                            AuthenticationResult result = getAccessToken(
                                    oidcResponse.getAuthorizationCode(),
                                    currentUri);
                            createSessionPrincipal(httpRequest, result);
                        } else {
                            AuthenticationErrorResponse oidcResponse = (AuthenticationErrorResponse) authResponse;
                            throw new Exception(String.format(
                                    "Request for auth code failed: %s - %s",
                                    oidcResponse.getErrorObject().getCode(),
                                    oidcResponse.getErrorObject()
                                            .getDescription()));
                        }
                    } else {
                            // not authenticated
                            httpResponse.setStatus(302);
                            httpResponse
                                    .sendRedirect(getRedirectUrl(currentUri));
                            return;
                    }
                } else {
                    // if authenticated, how to check for valid session?
                    AuthenticationResult result = AuthHelper
                            .getAuthSessionObject(httpRequest);

                    if (httpRequest.getParameter("refresh") != null) {
                        result = getAccessTokenFromRefreshToken(
                                result.getRefreshToken(), currentUri);
                    } else {
                        if (httpRequest.getParameter("cc") != null) {
                            result = getAccessTokenFromClientCredentials();
                        } else {
                            if (result.getExpiresOnDate().before(new Date())) {
                                result = getAccessTokenFromRefreshToken(
                                        result.getRefreshToken(), currentUri);
                            }
                        }
                    }
                    createSessionPrincipal(httpRequest, result);
                }
            } catch (Throwable exc) {
                httpResponse.setStatus(500);
                request.setAttribute("error", exc.getMessage());
                httpResponse.sendRedirect(((HttpServletRequest) request)
                        .getContextPath() + "/error.jsp");
            }
        }
        chain.doFilter(request, response);
    }

    private AuthenticationResult getAccessTokenFromClientCredentials()
            throws Throwable {
        AuthenticationContext context = null;
        AuthenticationResult result = null;
        ExecutorService service = null;
        try {
            service = Executors.newFixedThreadPool(1);
            context = new AuthenticationContext(authority + tenant + "/", true,
                    service);
            Future<AuthenticationResult> future = context.acquireToken(
                    "https://graph.windows.net", new ClientCredential(clientId,
                            clientSecret), null);
            result = future.get();
        } catch (ExecutionException e) {
            throw e.getCause();
        } finally {
            service.shutdown();
        }

        if (result == null) {
            throw new ServiceUnavailableException(
                    "authentication result was null");
        }
        return result;
    }

    private AuthenticationResult getAccessTokenFromRefreshToken(
            String refreshToken, String currentUri) throws Throwable {
        AuthenticationContext context = null;
        AuthenticationResult result = null;
        ExecutorService service = null;
        try {
            service = Executors.newFixedThreadPool(1);
            context = new AuthenticationContext(authority + tenant + "/", true,
                    service);
            Future<AuthenticationResult> future = context
                    .acquireTokenByRefreshToken(refreshToken,
                            new ClientCredential(clientId, clientSecret), null,
                            null);
            result = future.get();
        } catch (ExecutionException e) {
            throw e.getCause();
        } finally {
            service.shutdown();
        }

        if (result == null) {
            throw new ServiceUnavailableException(
                    "authentication result was null");
        }
        return result;

    }

    private AuthenticationResult getAccessToken(
            AuthorizationCode authorizationCode, String currentUri)
            throws Throwable {
        String authCode = authorizationCode.getValue();
        ClientCredential credential = new ClientCredential(clientId,
                clientSecret);
        AuthenticationContext context = null;
        AuthenticationResult result = null;
        ExecutorService service = null;
        try {
            service = Executors.newFixedThreadPool(1);
            context = new AuthenticationContext(authority + tenant + "/", true,
                    service);
            Future<AuthenticationResult> future = context
                    .acquireTokenByAuthorizationCode(authCode, new URI(
                            currentUri), credential, null);
            result = future.get();
        } catch (ExecutionException e) {
            throw e.getCause();
        } finally {
            service.shutdown();
        }

        if (result == null) {
            throw new ServiceUnavailableException(
                    "authentication result was null");
        }
        return result;
    }

    private void createSessionPrincipal(HttpServletRequest httpRequest,
            AuthenticationResult result) throws Exception {
        httpRequest.getSession().setAttribute(
                AuthHelper.PRINCIPAL_SESSION_NAME, result);
    }

    private String getRedirectUrl(String currentUri)
            throws UnsupportedEncodingException {
        String redirectUrl = authority
                + this.tenant
                + "/oauth2/authorize?response_type=code%20id_token&scope=openid&response_mode=form_post&redirect_uri="
                + URLEncoder.encode(currentUri, "UTF-8") + "&client_id="
                + clientId + "&resource=https%3a%2f%2fgraph.windows.net"
                + "&nonce=" + UUID.randomUUID() + "&site_id=500879";
        return redirectUrl;
    }

    public void init(FilterConfig config) throws ServletException {
        clientId = config.getInitParameter("client_id");
        authority = config.getServletContext().getInitParameter("authority");
        tenant = config.getServletContext().getInitParameter("tenant");
        clientSecret = config.getInitParameter("secret_key");
    }

}
```

此 Servlet 會公開 `ADAL4J` 預期來自我們的應用程式用以執行的所有方法。其中包括：

- `getAccessTokenFromClientCredentials()` - 從我們的密碼取得存取權杖
- `getAccessTokenFromRefreshToken()` - 從重新整理權杖中取得存取權杖
- `getAccessToken()` - 從我們所使用的 OpenID Connect 流程取得存取權杖
- `createSessionPrincipal()` - 建立我們使用圖形 API 存取的主體
- `getRedirectUrl()` - 取得重新導向 URL，以將它與您在入口網站輸入的值相比較。

##在 Tomcat 中編譯和執行範例

變更回根目錄，並執行下列命令來建置您剛剛使用 `maven` 組成的範例。這將會使用您針對相依性所撰寫的 `pom.xml` 檔案。

`$ mvn package`

您的 `/targets` 目錄中現在應包含 `adal4jsample.war` 檔案。您可以在 Tomcat 容器中部署該檔案並造訪 URL

`http://localhost:8080/adal4jsample/`


> [AZURE.NOTE] 
使用最新的 Tomcat 伺服器部署 WAR 非常容易。只要瀏覽至 `http://localhost:8080/manager/` 並遵循上傳您的 '`adal4jsample.war` 檔案的指示即可。它會為您自動部署正確的端點。

##後續步驟

恭喜！ 您現在有一個可運作的 Java 應用程式，能夠驗證使用者、使用 OAuth 2.0 安全地呼叫 Web API，以及取得使用者的基本資訊。如果您還沒有這麼做，現在是將一些使用者植入租用戶的時候。

如需參考，[此處以 .zip 格式提供](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/complete.zip)完整範例 (不含您的組態值)，您也可以從 GitHub 將其複製：

```git clone --branch complete https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect.git```

<!---HONumber=AcomDC_0921_2016-->