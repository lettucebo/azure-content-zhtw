<properties
	pageTitle="開始使用 Azure AD Java | Microsoft Azure"
	description="如何建立一個將使用者登入以存取 API 的Java 命令列應用程式。"
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


# 使用 Java 命令列應用程式存取具有 Azure AD 的 API

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
	<artifactId>public-client-adal4j-sample</artifactId>
	<packaging>jar</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>public-client-adal4j-sample</name>
	<url>http://maven.apache.org</url>
	<properties>
		<spring.version>3.0.5.RELEASE</spring.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>com.microsoft.azure</groupId>
			<artifactId>adal4j</artifactId>
			<version>1.1.2</version>
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
</dependencies>
	<build>
		<finalName>public-client-adal4j-sample</finalName>
		<plugins>
		        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>1.2.1</version>
            <configuration>
                <mainClass>PublicClient</mainClass>
            </configuration>
        </plugin>
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
			<plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
        </configuration>
      </plugin>
      <plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-assembly-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>PublicClient</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
		</plugins>
	</build>

</project>


```



## 3\.建立 java PublicClient 檔案

如以上所述，我們將使用圖形 API 來取得有關所登入使用者的資料。為了讓我們順利進行，我們應該建立一個代表**目錄物件**的檔案以及一個代表**使用者**的個別檔案，如此便可以使用 Java 的 OO 模式。

1. 建立名為 `DirectoryObject.java` 的檔案，我們將用它來儲存有關任何 DirectoryObject 的基本資料 (您稍後可以隨意使用它執行任何其他圖形查詢)。您可以從下面剪下/貼上：

```Java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

import javax.naming.ServiceUnavailableException;

import com.microsoft.aad.adal4j.AuthenticationContext;
import com.microsoft.aad.adal4j.AuthenticationResult;

public class PublicClient {

    private final static String AUTHORITY = "https://login.microsoftonline.com/common/";
    private final static String CLIENT_ID = "2a4da06c-ff07-410d-af8a-542a512f5092";

    public static void main(String args[]) throws Exception {

        try (BufferedReader br = new BufferedReader(new InputStreamReader(
                System.in))) {
            System.out.print("Enter username: ");
            String username = br.readLine();
            System.out.print("Enter password: ");
            String password = br.readLine();

            AuthenticationResult result = getAccessTokenFromUserCredentials(
                    username, password);
            System.out.println("Access Token - " + result.getAccessToken());
            System.out.println("Refresh Token - " + result.getRefreshToken());
            System.out.println("ID Token - " + result.getIdToken());
        }
    }

    private static AuthenticationResult getAccessTokenFromUserCredentials(
            String username, String password) throws Exception {
        AuthenticationContext context = null;
        AuthenticationResult result = null;
        ExecutorService service = null;
        try {
            service = Executors.newFixedThreadPool(1);
            context = new AuthenticationContext(AUTHORITY, false, service);
            Future<AuthenticationResult> future = context.acquireToken(
                    "https://graph.windows.net", CLIENT_ID, username, password,
                    null);
            result = future.get();
        } finally {
            service.shutdown();
        }

        if (result == null) {
            throw new ServiceUnavailableException(
                    "authentication result was null");
        }
        return result;
    }
}


```


##編譯並執行範例

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