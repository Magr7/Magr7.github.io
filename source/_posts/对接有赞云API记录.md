---
title: 有赞云对接Api
---
关于获取无容器自主开发/有容器定制开发 不是太懂，具体对接的时候再了解。
#### JAVA
+ SDK下载
详情参考 [有赞云SDK下载](https://doc.youzanyun.com/resource/develop-guide/41185/41704)
+ 可以通过Maven来管理依赖
``` xml
 <dependency>  
    <groupId>com.youzan.cloud</groupId>  
    <artifactId>open-sdk-core</artifactId>  
    <version>1.0.22-RELEASE</version>
</dependency>

<dependency>  
    <groupId>com.youzan.cloud</groupId>  
    <artifactId>open-sdk-gen</artifactId>  
    <version>1.0.22.80701202109281055-RELEASE</version>
</dependency>
```
+ maven本地配置饮用有赞云的私仓
``` xml
     <mirrors>
         <mirror>
             <id>youzan-nexus-snapshot</id>
             <name>Maven Repository Mirror running on maven.youzanyun.com</name>
             <url>http://maven.youzanyun.com/repository/maven-public</url>
             <mirrorOf>central</mirrorOf>
         </mirror>
     </mirrors> 

     <profiles>
         <profile>
             <id>dev</id>
             <repositories>
                <repository>
                   <id>youzanyun-central</id>
                   <name>Nexus Release Repository</name>
                   <url>http://maven.youzanyun.com/repository/maven-central/</url>
                   <releases>
                       <enabled>true</enabled>
                   </releases>
                   <snapshots>
                       <enabled>false</enabled>
                   </snapshots>
                </repository>
                <repository>
                   <id>youzanyun-releases</id>
                   <name>Nexus Release Repository</name>
                   <url>http://maven.youzanyun.com/repository/maven-release/</url>
                   <releases>
                       <enabled>true</enabled>
                   </releases>
                   <snapshots>
                      <enabled>false</enabled>
                   </snapshots>
               </repository>
               <repository>
                   <id>youzanyun-snapshots</id>
                   <name>Nexus Snapshot Repository</name>
                   <url>http://maven.youzanyun.com/repository/maven-snapshots/</url>
                   <releases>
                     <enabled>false</enabled>
                   </releases>
                   <snapshots>
                     <enabled>true</enabled>
                   </snapshots>
               </repository>
           </repositories>
           <pluginRepositories>
             <pluginRepository>
               <id>youzanyun-plugin</id>
               <name>youzanyun repository</name>
               <url>http://maven.youzanyun.com/repository/maven-releases/</url>
               <releases>
                 <enabled>true</enabled>
               </releases>
               <snapshots>
                 <enabled>false</enabled>
               </snapshots>
             </pluginRepository>
             <pluginRepository>
               <id>youzanyun-central</id>
               <name>Nexus Release Repository</name>
               <url>http://maven.youzanyun.com/repository/maven-central/</url>
               <releases>
                   <enabled>true</enabled>
                   </releases>
               <snapshots>
                   <enabled>false</enabled>
                </snapshots>
              </pluginRepository>
              <pluginRepository>
                   <id>youzanyun-snapshots</id>
                   <name>Nexus Snapshot Repository</name>
                   <url>http://maven.youzanyun.com/repository/maven-snapshots/</url>
                   <releases>
                       <enabled>false</enabled>
                   </releases>
                   <snapshots>
                       <enabled>true</enabled>
                   </snapshots>
               </pluginRepository>
           </pluginRepositories>
         </profile>
     </profiles>
     <activeProfiles>
         <activeProfile>dev</activeProfile>
     </activeProfiles> 
```

+ 具体代码实现，针对不同的业务
``` java
    // 使用Spring容器管理Youzan Client
    @Configuration
    public class YzClientUtils {

        @Bean(name = "defaultYZClient")
        public DefaultYZClient defaultYZClient() {

            return new DefaultYZClient();
        }
    }

    // 使用的时候
    @Resource(name = "defaultYZClient")
    private DefaultYZClient yzClient;    
```
+ 单元测试
``` java
    // 获取token
    @Test
    public void getToken() throws SDKException {
        TokenParameter tokenParameter = TokenParameter.self()
                .clientId(youzanClientID)
                .clientSecret(youzanSecret)
                .grantId(kdtid)
                .refresh(true)
                .build();

        OAuthToken oAuthToken = defaultYZClientTest.getOAuthToken(tokenParameter);

        System.out.println(oAuthToken);
    }
    // 创建用户
    @Test
    public void createUser() throws SDKException {
        YouZanClient yzClient = new DefaultYZClient();
        Token token = new Token(accessToken);

        YouzanScrmCustomerCreate youzanScrmCustomerCreate = new YouzanScrmCustomerCreate();

        //创建参数对象,并设置参数
        YouzanScrmCustomerCreateParams youzanScrmCustomerCreateParams = new YouzanScrmCustomerCreateParams();

        YouzanScrmCustomerCreateParams.YouzanScrmCustomerCreateParamsCustomercreate youzanScrmCustomerCreateParamsCustomerCreate  = new YouzanScrmCustomerCreateParams.YouzanScrmCustomerCreateParamsCustomercreate();
        youzanScrmCustomerCreateParams.setCustomerCreate(youzanScrmCustomerCreateParamsCustomerCreate);
        youzanScrmCustomerCreateParamsCustomerCreate.setGender((short)0);
        youzanScrmCustomerCreateParamsCustomerCreate.setWeiXin("");
        youzanScrmCustomerCreateParamsCustomerCreate.setName("测试1613");
        youzanScrmCustomerCreateParams.setMobile("183****1613");

        youzanScrmCustomerCreateParams.setIsDoExtPoint(false);
        youzanScrmCustomerCreateParams.setCreateDate(DateUtil.format(new Date(), "yyyy-MM-dd HH:mm:ss"));

        youzanScrmCustomerCreate.setAPIParams(youzanScrmCustomerCreateParams);
        YouzanScrmCustomerCreateResult result = yzClient.invoke(youzanScrmCustomerCreate, token, YouzanScrmCustomerCreateResult.class);

        System.out.println(result.getCode());
        System.out.println(result.getData());
    }

    // 批量手机号获取用户
    @Test
    public void UserQueryByListPhone() throws SDKException {
        YouZanClient yzClient = new DefaultYZClient();
        //YouZanClient 建议全局唯一,使用 spring 容器管理
        Token token = new Token(accessToken);

        YouzanScrmCustomerListPhone youzanScrmCustomerListPhone = new YouzanScrmCustomerListPhone();


        //创建参数对象,并设置参数
        YouzanScrmCustomerListPhoneParams youzanScrmCustomerListPhoneParams = new YouzanScrmCustomerListPhoneParams();
        youzanScrmCustomerListPhoneParams.setPhones(Arrays.asList("18321771613"));

        youzanScrmCustomerListPhone.setAPIParams(youzanScrmCustomerListPhoneParams);
        YouzanScrmCustomerListPhoneResult result = yzClient.invoke(youzanScrmCustomerListPhone, token, YouzanScrmCustomerListPhoneResult.class);
        System.out.println(JSON.toJSONString(result));
    }
```
+ 接下来，定义接口+实现类 具体处理业务就可以了
  
+ 有赞云消息回调

#### PHP
YouzanYun SDK
+ 引入
``` bash
$ composer require youzanyun/open-sdk
```
+ 使用
详情参考 [examples](https://github.com/youzan/open-sdk-php/tree/620b89e06a13c6e33b117ca0812d2743dd29d4a6/examples)

