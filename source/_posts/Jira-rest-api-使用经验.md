---
title: Jira rest api 使用经验
date: 2017-03-26 22:46:49
categories:
- Development
tags:
- Java
- JIRA
---

## 1. JIRA REST API 可以做什么？

<blockquote class="blockquote-center">

The Atlassian REST APIs provide a standard interface for interacting with JIRA and our other applications. 

REST APIs provide access to resources (data entities) via URI paths. To use a REST API, your application will make an HTTP request and parse the response. Your methods will be the standard HTTP methods like GET, PUT, POST and DELETE. REST APIs operate over HTTP(s) making it easy to use with any programming language or framework. The input and output format for the JIRA REST APIs is [JSON](http://www.json.org/).

</blockquote>

我们可以通过REST API去访问收保护的资源，例如增、删、改、查ISSUES 。。。。。

支持标准的HTTP method，例如 GET, PUT, POST , DELETE

所有的API的input 和 output 都是 JSON 格式。

{% note primary %}

 Rest API Reference ： https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis

{% endnote %}

{% note info %}

 REST URI 结构 ： http://host:port/context/rest/api-name/api-version/resource-name

{% endnote %}

{% note success %}

 资源列表 ： https://developer.atlassian.com/static/rest/jira/6.1.html

{% endnote %}

<!-- more -->

##  2. JIRA REST API鉴权方式

{% cq %}

The REST APIs support basic authentication, cookie-based (session) authentication, and OAuth. See the examples of [basic authentication](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-basic-authentication), [cookie-based authentication](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-cookie-based-authentication), and [OAuth](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-oauth-authentication).

{% endcq %}

JIRA REST API 支持3种鉴权方式：

1. [basic authentication](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-basic-authentication)
2. [cookie-based authentication](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-cookie-based-authentication)
3. [OAuth](https://developer.atlassian.com/jiradev/jira-apis/jira-rest-apis/jira-rest-api-tutorials/jira-rest-api-example-oauth-authentication)

{% note warning %} 

目前JIRA REST API 只支持OAuth1.0

 {% endnote %}

##  3. 如何使用OAuth方式鉴权

因为basic authentication 和 cookie-based authentication两种方式比较简单，且不推荐使用，这里我说说推荐使用的OAuth方式如何玩。

1. 首先需要到Jira上配置Application link，注册一个新的consumer
2. 配置客户端oauth信息

### Step1: Create application link

{% note %}

JIRA Administration > System > Applications tab > Application links

{% endnote %}

### Step2: Configuring the client

| OAuth Config       | value                                    |
| :----------------- | :--------------------------------------- |
| request token url  | JIRA_BASE_URL + "/plugins/servlet/oauth/request-token" |
| authorization url  | JIRA_BASE_URL + "/plugins/servlet/oauth/authorize"" |
| access token url   | JIRA_BASE_URL + "/plugins/servlet/oauth/access-token" |
| oauth signing type | RSA-SHA1，需要生成key pair，且粘贴public key到oauth配置里面 |
| consumer key       | Step1里面配置的值                              |

## 4. 使用Java client来获取access token

{% note success %}

Java oauth client 是基于John Kristian的源码做了一些改进

Github ：https://github.com/lisanlai/jira.oauth.client

{% endnote %}

```java
/**
 * Created by lisanlai on 2017/3/27.
 */
public class ClientTest {

    private static final String BASEURL = "Jira domain url";
    private static final String CONSUMER_KEY = "你自己配置的consumer key";
    private static final String CONSUMER_PRIVATE_KEY = "你自己生成的RSA private key";
    private static final String CALLBACK_URI = "http://requestb.in/12f4ung1";

    public static void main(String[] args) throws Exception {
        AtlassianOAuthClient jiraoAuthClient = new AtlassianOAuthClient(CONSUMER_KEY, CONSUMER_PRIVATE_KEY, BASEURL, CALLBACK_URI);
        //STEP 1: 获取request token
        TokenSecretVerifierHolder requestToken = jiraoAuthClient.getRequestToken();
        String authorizeUrl = jiraoAuthClient.getAuthorizeUrlForToken(requestToken.token);
        System.out.println("Token is " + requestToken.token);
        System.out.println("Token secret is " + requestToken.secret);
        System.out.println("Retrieved request token. go to " + authorizeUrl);

        //STEP2 : 授权， 打开STEP1里面获取到的authorize url
        //登录jira并点击allow按钮

        //STEP3 : 获取 access token
        //getAccessToken("810qrpbOePqXdMRWcoOuMLdoBoLuh9To", "M4O6qaRC4OVDKzQOCcVrmvvpCNkYmhpm", "0aOMdW");

    }

    public static void getAccessToken(String requestToken, String tokenSecret, String verifier) throws OAuthException, IOException, URISyntaxException {
        AtlassianOAuthClient jiraoAuthClient = new AtlassianOAuthClient(CONSUMER_KEY, CONSUMER_PRIVATE_KEY, BASEURL, CALLBACK_URI);
        String accessToken = jiraoAuthClient.swapRequestTokenForAccessToken(requestToken, tokenSecret, verifier);
        System.out.println("Access token is : " + accessToken);
    }

}
```

## 5. 使用REST API创建/修改/查询issue

```java
String accessToken = "access token";
String requestUrl =  JIRA_BASE_URL + "/rest/api/2/issue";
String jsonBody = "{\n" +
                "    \"fields\": {\n" +
                "       \"project\":\n" +
                "       { \n" +
                "          \"key\": \"ASYNC\"\n" +
                "       },\n" +
                "       \"summary\": \"REST ye merry gentlemen.\",\n" +
                "       \"description\": \"Creating of an issue using project keys and issue type names using the REST API\",\n" +
                "       \"issuetype\": {\n" +
                "          \"name\": \"Bug\"\n" +
                "       }\n" +
                "   }\n" +
                "}";

String responseAsString = jiraoAuthClient.makeAuthenticatedRequest(accessToken, OAuthMessage.POST, requestUrl, Collections.emptySet(), jsonBody);
System.out.println("RESPONSE IS" + responseAsString);
```

## 6. JIRA OAuth dance 

{% asset_img jira-oauth-dance.png JIRA OAuth dance %}