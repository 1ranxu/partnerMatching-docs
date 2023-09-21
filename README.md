# 樱络 - 伙伴匹配系统

简介:帮助大家找到志同道合的伙伴，移动端的H5网页（尽量兼容PC端）

用户可以去用户中心注册，登录用户中心，每个用户可以打上一些标签，伙伴匹配系统根据标签帮家找到志同道合的伙伴

## 需求分析

1. 用户给自己添加标签；修改标签，系统根据标签对用户进行分类（有哪些标签，如何对标签进行分类）分类比如：学习方向，就业状态，理想城市，工作，大学等
2. 主动搜索：允许用户根据标签去搜索其他用户
   1. 把需要被搜索的数据加到Redis缓存

3. 组队（前端的同学想找后端的同学帮助自己完成后端，后端的同学也需要前端的同学帮助自己完成前端，需要帮助的同学就可以创建一个队伍，等待有缘人，只能由一方单向发起组队）
   1. 创建队伍
   2. 加入队伍
   3. 根据标签查询队伍
   4. 删除队伍
   5. 编辑队伍
   6. 邀请其他人
   7. 队伍人数由上限，但为了防止有人放鸽子，可以多加几个人作为替补

4. 推荐相似伙伴
   1. 使用相似度计算算法　＋　本地分布式（采用多线程）


## 技术选型

### 前端

1. Vue3框架（提高页面开发效率）
2. Vant3 UI组件库（基于Vue的移动端组件库）（它也有基于React的版本Zent）
3. Vite4（打包工具，速度快）
4. Nginx 单机部署

### 后端

1. Java8 （编程语言）
2. Spring（依赖注入框架，管理Java对象，集成一些其他内容） 
3. SpringMvc （Web框架，提供接口访问，restful接口等能力）
4. Mybatis（Java操作数据库的框架，持久层框架，对JDBC的封装） 
5. Mybatis-Plus（对mybatis的增强，不用写SQL也能实现增删改查） 
6. SpringBoot（快速启动/快速集成项目，帮助管理Spring的配置，帮助整合框架） 
7. MySQL（数据库）
8. Redis（缓存，NoSQL）
9. Swagger + Knife4j 接口文档



## 计划

1. **前端项目初始化**
2. **前端主页 + 组件概览**
3. **数据库表设计**
   1. 标签表
   2. 用户表
4. **后端项目初始化**
5. **后端开发**
   1. 使用标签搜索用户
   2. 整合Swagger + Knife4j 接口文档
   3. 存量用户信息导入及同步（爬虫）
   4. 用户修改个人信息
   5. 主页推荐（默认推荐和自己兴趣相当的用户）
   6. 优化主页性能（缓存 +定时任务+分布式锁）
   7. 标签内容整理
   8. 组队
6. **前端开发**
   1. 整合路由
   2. 搜索页面-根据标签搜索用户
   3. 用户信息页
   4. 用户信息修改页
   5. 搜索结果页
   6. 主页
7. **部分细节优化**




## 前端项目初始化

**文档介绍**

![image-20230912203234599](assets/image-20230912203234599.png)

### 脚手架初始化项目

使用[ Vue CLI ](https://cli.vuejs.org/zh/)或者[Vite 官方](https://cn.vitejs.dev/guide/)

本项目使用Vite

![image-20230912214604415](assets/image-20230912214604415.png)

```
yarn create vite
```

![image-20230912215308003](assets/image-20230912215308003.png)

```sh
#安装依赖
npm install
```

![image-20230913133810106](assets/image-20230913133810106.png)

![image-20230913134340025](assets/image-20230913134340025.png)

### 整合Vant组件库

1. 安装vant

![image-20230913141746828](assets/image-20230913141746828.png)

```
npm i vant
```

2. 安装插件

```sh
#它可以自动引入组件，并按需引入组件的样式。
npm i vite-plugin-style-import@1.4.1 -D
```

3. 让Vite认识Vant-在vite.config,ts中修改代码

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import styleImport,{ VantResolve } from 'vite-plugin-style-import';
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(),styleImport({
    resolves:[VantResolve()],
    libs: [
      {
        libraryName: 'vant',
        esModule: true,
        resolveStyle: (name) => `../es/${name}/style`
      }
    ]
  }),],
})
```

4. 引入组件

![image-20230913142523914](assets/image-20230913142523914.png)

5. 修改main.ts文件

```ts
//不用多引入样式
import { createApp } from 'vue'
import App from './App.vue'
import {Button, NavBar} from "vant";

const app=createApp(App)

app.mount('#app')
```

## 前端主页 + 组件概览

**开发页面经验**：

1. 多参考
2. 从整体到局部
3. 先明确页面的布局，再写代码

**精简页面**

![image-20230913193149609](assets/image-20230913193149609.png)

![image-20230913193359006](assets/image-20230913193359006.png)

### 页面设计

1. 导航条：展示当前页面名称
2. 主页搜索框 ==> 搜索页（标签筛选页，通过标签筛选用户）==> 搜索结果页
3. 内容：

2. Tab栏
   - 主页（推荐页 + **广告**）
     - 搜锁框
     - banner
     - 推荐信息流
   - 队伍
   - 用户页（消息-暂时考虑发邮件）

### 组件概览

1. 导航栏

   ![image-20230913195343599](assets/image-20230913195343599.png)

2. TabBar

   ![image-20230914153548155](assets/image-20230914153548155.png)

3.  

### 开发

#### 通用布局

1. 抽象一个通用的布局（Layout），使得其他页面能够复用布局中的组件/样式，利于维护

![image-20230913195129701](assets/image-20230913195129701.png)

2. 复制导航栏的代码到这个布局中

![image-20230914123843394](assets/image-20230914123843394.png)

3. 在App.vue中添加布局

![image-20230914151646028](assets/image-20230914151646028.png)

4. 在main.ts中引入NavBar,Icon

![image-20230914152329292](assets/image-20230914152329292.png)

5. 对导航栏进行修改

![image-20230914152614404](assets/image-20230914152614404.png)

![image-20230914152636314](assets/image-20230914152636314.png)

6. 引入TabBar

![image-20230914153823356](assets/image-20230914153823356.png)

![image-20230914154945288](assets/image-20230914154945288.png)

![image-20230914163717175](assets/image-20230914163717175.png)

7. 效果

![image-20230914163929186](assets/image-20230914163929186.png)

![image-20230914163956560](assets/image-20230914163956560.png)

## 数据库表设计

### 新增标签表

**标签**：

性别：男，女

方向：Java，C++，Go，前端，后端

目标：考研，春招，秋招，社招，考公，竞赛，转行，跳槽

段位：初级，中级，高级，码神

身份：小学，初中，高中，大一，大二，大三，大四，学生，待就业，已就业，研一，研二，研三

状态：乐观，聒噪，狂热，一般，单身，已婚，有对象

用户自己定义标签：开发者不可能想到所有用户需要的标签，让用户去决定要右哪些标签

**字段**：

id   		   bigint          主键 	

tagName vachar  	 非空标签名（标签具体内容，比如男）（唯一索引）

userId 	 bigint 		上传标签的用户 （用户可以自定义标签，然后上传） （普通索引，根据userId查询已上传的标签）

parentId  bigint 		父标签id (就是标签的抽象，比如性别)

isParent   tinyint   	是否为父标签（0-不是，1-是） 

createTime datetime 创建时间

updateTime datetime 更新时间

isDeleted tinyint		 逻辑删除(0-未删除，1-已删除)

通过父标签id来分组 

```mysql
DROP TABLE IF EXISTS tag;

create table user_center.tag
(
    id         bigint auto_increment comment '(主键) '
        primary key,
    tagName    varchar(256)                       null comment '标签名称',
    userId     bigint                             null comment '用户id',
    parentId   bigint                             null comment '父标签id',
    isParent   tinyint                            null comment '0-不是父标签，1-是父标签',
    createTime datetime default CURRENT_TIMESTAMP not null comment '用户创建时间',
    updateTime datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '用户更新时间',
    isDeleted  tinyint  default 0                 not null comment '标签是否已删除',
    constraint uniIdx_tagName
        unique (tagName)
)
    comment '标签表';

create index Idx_userId
    on tag (userId);
```



### 修改用户表

**根据自己的需求！！！**

1. 直接在用户表补充tags字段，['Java','男'] Json（**采用**）

   优点：查询方便，不用新建关联表，标签是用户的固有属性（除了该系统，其他系统可能也会用到），节省开发成本。可以用缓存，提高用户查询性能

   缺点：用户表多一列

2. 加一个关联表，记录用户和标签的关系（**尽量减少关联查询**）

   优点：查询灵活，可以正查反查

   缺点：查询100个用户的列表时，查完用户表得到用户id，又要根据用户id查关联表获取标签id，又要根据标签id查标签表获取标签，影响扩展和查询性能

**用户表**：

```mysql
DROP TABLE IF EXISTS user;

create table user_center.user
(
    id           bigint auto_increment comment '(主键) '
        primary key,
    userAccount  varchar(256)                       null comment '登录账号',
    username     varchar(256)                       null comment '用户昵称',
    avatarUrl    varchar(1024)                      null comment '用户头像',
    gender       tinyint                            null comment '用户性别',
    userPassword varchar(256)                       not null comment '用户密码',
    phone        varchar(128)                       null comment '用户电话',
    email        varchar(512)                       null comment '用户邮箱',
    userStatus   int      default 0                 not null comment '用户状态 0-正常 ',
    createTime   datetime default CURRENT_TIMESTAMP not null comment '用户创建时间',
    updateTime   datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '用户更新时间',
    isDeleted    tinyint  default 0                 not null comment '用户是否删除',
    userRole     int      default 0                 not null comment '用户角色 0-普通用户 1-管理员',
    authCode     varchar(512)                       null comment '付费系统编号，用于校验用户'
    
)
    comment '用户表';
#补充tags字段    
alter table user add COLUMN tags varchar(1024) null comment '标签列表Json'
#补充profile字段  
alter table user add COLUMN profile varchar(512) null comment '个人简介'
```

## 后端项目初始化

本来想把标签的增删改查等业务放到新项目里，实际分析后，因为标签和用户关联比较强，所以还是把标签的增删改查放到用户中心项目里

1. 把用户中心项目后端代码复制，目录改成partnerMatching-backend，删掉.idea，iml文件；修改pom.xml文件

![image-20230915095601977](assets/image-20230915095601977.png)

## 用户中心提供用户的查询，操作，注册，登录，鉴权



## 后端开发

### 使用标签搜索用户

建议通过实际测试来分析哪种查询比较快，数据量大的时候验证效果更明显！

- 如果参数可以分析，根据用户的参数去选择查询方式，比如标签数。标签数少，用SQL查询；标签数多，用内存查询

- 如果参数不可分析，并且数据库连接足够、内存空间足够，可以并发同时查询，谁先返回用谁。
- 还可以 SQL 查询与内存查询相结合，比如先用 SQL 过滤掉部分 tag

**SQL查询**

1. 允许用户传入多个标签，多个标签在用户的tags字段都存在，用户才能被搜出来 

   ```
   tags like '%Java%' and  tags like '%C++%' and tags like '%Go%'
   ```

2. 允许用户传入多个标签，任何标签在用户的tags字段存在，用户就能被搜出来

   ```
   tags like '%Java%' or  tags like '%C++%' or tags like '%Go%'
   ```

```java
/**
 * 使用标签搜索用户（SQL查询版）
 *
 * @param tagList 用户传入的标签
 * @return
 */
@Deprecated
public List<UserDTO> queryUsersByTagsBySQL(List<String> tagList) {
    //判空
    if (CollectionUtil.isEmpty(tagList)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "标签不能为空");
    }
    //SQL模糊查询
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    for (String tag : tagList) {
        queryWrapper.like(StringUtils.isNotBlank(tag), User::getTags, tag);
    }
    List<User> users = userMapper.selectList(queryWrapper);
    //脱敏
    return users.stream().map(user -> BeanUtil.copyProperties(user, UserDTO.class)).collect(Collectors.toList());
}
```

**内存查询**

1. 在用户中心的UserServiceImpl中提供queryUsersByTagsByMemory接口

```java
/**
 * 使用标签搜索用户(内存过滤）
 *
 * @param tagList 用户传入的标签
 * @return
 */
@Override
public List<UserDTO> queryUsersByTagsByMemory(List<String> tagList) {
    //判空
    if (CollectionUtil.isEmpty(tagList)) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "标签不能为空");
    }
    //内存查询
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    //查询所有用户
    List<User> users = userMapper.selectList(queryWrapper);
    Gson gson = new Gson();
    return users.stream().filter(user -> {
        Set<String> tempTagSet = gson.fromJson(user.getTags(), new TypeToken<Set<String>>() {
        }.getType());
        //有些用户的tags字段可能为空，要先判空，否则报控空指针异常
        tempTagSet = Optional.ofNullable(tempTagSet).orElse(new HashSet<>());
        //过滤
        for (String tag : tagList) {
            if (!tempTagSet.contains(tag))
                return false;
        }
        return true;
    }).map(user -> BeanUtil.copyProperties(user, UserDTO.class)).collect(Collectors.toList());
}
```

2. 因为user表加了字段，相应的做些修改

![image-20230915152948531](assets/image-20230915152948531.png)

![image-20230915153041395](assets/image-20230915153041395.png)

![image-20230915153137469](assets/image-20230915153137469.png)

3. 在UserController中添加searchByTags方法，接受前端请求

```
@GetMapping("/searchByTags")
public Result searchByTags(@RequestParam(required = false) List<String> tags){
    if (CollectionUtil.isEmpty(tags)){
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"标签为空");
    }
    return Result.success(userService.queryUsersByTagsByMemory(tags));
}
```

4. 解决跨域

![image-20230919203356552](assets/image-20230919203356552.png)

### 整合Swagger + Knife4j 接口文档

1. 什么是接口文档？

   用于描述接口信息，包括：

   - 请求参数
   - 响应参数
   - 接口地址
   - 接口名称
   - 请求类型
   - 请求格式
   - 备注

2. 由谁提供，由谁使用？

   一般是后端或者负责人来提供，后端和前端都可以使用

3. 为什么需要接口文档？

   - 便于参考和查阅，便于**沉淀和维护**，拒绝口口相传
   - 便于前后端开发对接，是前后端联调的**介质**
   - 支持在线调试，在线测试， 可以作为工具，提高我们的开发效率

4. 怎么做接口文档？

   - 手写（MarkDown笔记等)
   - 自动化接口文档生成：（自动根据项目代码生成完整的文档或在线调试的网页）
     - **Swagger（采用）**，Postman（侧重接口管理）（国外）
     - apifox，apipost，eolink（国产）

5. 接口文档右哪些技巧？

6. Swagger原理

   1. 引入依赖（Swagger 或 Knife4j：[1.6 快速开始 | knife4j (xiaominfo.com)](https://doc.xiaominfo.com/v2/documentation/get_start.html)）

   2. 自定义Swagger配置类

   3. 定义需要生成接口文档的代码的位置（controller）

   4. 启动项目即可

   5. 可以通过在 controller 方法上添加 @Api、@ApiImplicitParam(name = "name",value = "姓名",required = true)    @ApiOperation(value = "向客人问好") 等注解来自定义生成的接口描述信息

   6. 如果 springboot version >= 2.6，需要添加如下配置：

      ```yaml
      spring:
        mvc:
        	pathmatch:
            matching-strategy: ANT_PATH_MATCHER
      ```

   7. 访问http://localhost:8080/api/doc.html

7. 注意：线上环境不要把接口暴露出去！！！

8. 隐藏 可以通过在 SwaggerConfig 配置类开头加上 `@Profile({"dev", "test"})` 限定配置仅在部分环境开启，其实是限制bean在特定环境下被注册到容器中

**整合**

1. 引入依赖

   ```java
   <!--swagger-->
   <dependency>
       <groupId>com.github.xiaoymin</groupId>
       <artifactId>knife4j-spring-boot-starter</artifactId>
       <!--在引用时请在maven中央仓库搜索2.X最新版本号-->
       <version>2.0.9</version>
   </dependency>
   ```

2. 编写配置类

   ```java
   package com.luoying.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import springfox.documentation.builders.ApiInfoBuilder;
   import springfox.documentation.builders.PathSelectors;
   import springfox.documentation.builders.RequestHandlerSelectors;
   import springfox.documentation.service.ApiInfo;
   import springfox.documentation.service.Contact;
   import springfox.documentation.spi.DocumentationType;
   import springfox.documentation.spring.web.plugins.Docket;
   import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;
   
   /**
    * 自定义Swagger接口文档使用的配置
    *
    * @author 落樱的悔恨
    */
   @Configuration
   @EnableSwagger2WebMvc
   public class Swagger2Configuration {
       @Bean(value = "defaultApi2")
       public Docket createRestApi(){
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
                   //这里标注controller包的位置
                   .apis(RequestHandlerSelectors.basePackage("com.luoying.controller"))
                   .paths(PathSelectors.any())
                   .build();
       }
    
       //api基本信息的配置，信息会在api文档上显示
       private ApiInfo apiInfo(){
           return new ApiInfoBuilder()
                   .title("LUOYING用户中心")
                   .description("LUOYING用户中心相关接口的文档")
                   .termsOfServiceUrl("http://github.com/1ranxu")
                   .contact(new Contact("ranxu","http://github.com/1ranxu","1574925401@qq.com"))
                   .version("1.0")
                   .build();
       }
   }
   ```

3. 修改配置文件

   ```yml
   spring:  
     mvc:
       pathmatch:
         matching-strategy: ant_path_matcher
   #springboot版本>=2.6，
   ```

4. 效果

   ![image-20230916195725901](assets/image-20230916195725901.png)

### 存量用户信息导入及同步（爬虫）

1. 把所有星球用户的信息导入用户表
2. 从每个用户的自我介绍中解析标签，在用户表中给每个用户打标签，并把解析的标签导入标签表

**看上了网页信息，怎么抓取**

FeHelper

1. 分析原网站是用哪个接口获取的[api.zsxq.com/v2/hashtags/48844541281228/topics?count=20](https://api.zsxq.com/v2/hashtags/48844541281228/topics?count=20)

   按 F 12 打开控制台，查看网络请求，复制 curl 代码便于查看和执行：

   ```bash
   curl "https://api.zsxq.com/v2/hashtags/48844541281228/topics?count=20" ^
     -H "authority: api.zsxq.com" ^
     -H "accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7" ^
     -H "accept-language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6" ^
     -H "cache-control: max-age=0" ^
     -H "cookie: zsxq_access_token=AA992A14-3A7E-86B6-02ED-06E0FC6B65FE_2EF81009CEAA9466; sensorsdata2015jssdkcross=^%^7B^%^22distinct_id^%^22^%^3A^%^2218a8ea5df263fd-099a715b3ba0c38-7f5d547e-1327104-18a8ea5df27174d^%^22^%^2C^%^22first_id^%^22^%^3A^%^22^%^22^%^2C^%^22props^%^22^%^3A^%^7B^%^22^%^24latest_traffic_source_type^%^22^%^3A^%^22^%^E5^%^BC^%^95^%^E8^%^8D^%^90^%^E6^%^B5^%^81^%^E9^%^87^%^8F^%^22^%^2C^%^22^%^24latest_search_keyword^%^22^%^3A^%^22^%^E6^%^9C^%^AA^%^E5^%^8F^%^96^%^E5^%^88^%^B0^%^E5^%^80^%^BC^%^22^%^2C^%^22^%^24latest_referrer^%^22^%^3A^%^22https^%^3A^%^2F^%^2Fwww.yuque.com^%^2F^%^22^%^7D^%^2C^%^22identities^%^22^%^3A^%^22eyIkaWRlbnRpdHlfY29va2llX2lkIjoiMThhOGVhNWRmMjYzZmQtMDk5YTcxNWIzYmEwYzM4LTdmNWQ1NDdlLTEzMjcxMDQtMThhOGVhNWRmMjcxNzRkIn0^%^3D^%^22^%^2C^%^22history_login_id^%^22^%^3A^%^7B^%^22name^%^22^%^3A^%^22^%^22^%^2C^%^22value^%^22^%^3A^%^22^%^22^%^7D^%^2C^%^22^%^24device_id^%^22^%^3A^%^2218a8ea5df263fd-099a715b3ba0c38-7f5d547e-1327104-18a8ea5df27174d^%^22^%^7D; abtest_env=product; zsxqsessionid=b0e9ee1668ca2ed22f495c4c2190422f" ^
     -H "sec-ch-ua: ^\^"Microsoft Edge^\^";v=^\^"117^\^", ^\^"Not;A=Brand^\^";v=^\^"8^\^", ^\^"Chromium^\^";v=^\^"117^\^"" ^
     -H "sec-ch-ua-mobile: ?0" ^
     -H "sec-ch-ua-platform: ^\^"Windows^\^"" ^
     -H "sec-fetch-dest: document" ^
     -H "sec-fetch-mode: navigate" ^
     -H "sec-fetch-site: none" ^
     -H "sec-fetch-user: ?1" ^
     -H "upgrade-insecure-requests: 1" ^
     -H "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36 Edg/117.0.2045.31" ^
     --compressed
   ```


2. **用程序去调用接口** （java okhttp httpclient / python 都可以）
3. 处理（清洗）一下数据，之后就可以写到数据库里

**流程**

1. 从 excel 中导入全量用户数据，**判重** 。 [EasyExcel官方文档 - 基于Java的Excel处理工具 | Easy Excel (alibaba.com)](https://easyexcel.opensource.alibaba.com/index.html)
2. 抓取写了自我介绍的同学信息，提取出用户昵称、用户唯一 id、自我介绍信息
3. 从自我介绍中提取信息，然后写入到数据库中

**使用EasyExcel从Excel表中读取数据**

1. 引入依赖

   ```java
   <!-- https://mvnrepository.com/artifact/com.alibaba/easyexcel -->
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>easyexcel</artifactId>
       <version>3.1.1</version>
   </dependency>
   ```

2. 建立对象，和Excel表字段形成映射关系

   ```java
   @Data
   public class DemoData {
       /**
        * 成员编号
        */
       @ExcelProperty("成员编号")
       private String authCode;
   
       /**
        * 用户昵称
        */
       @ExcelProperty("成员昵称")
       private String username;
   
   }
   ```

3. 读取模式（任选一种)

   1. 监听器

   ```java
   // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
   @Slf4j
   public class DemoDataListener implements ReadListener<DemoData> {
   
       /**
        * 这个每一条数据解析都会来调用
        *
        * @param data    one row value. Is is same as {@link AnalysisContext#readRowHolder()}
        * @param context
        */
       @Override
       public void invoke(DemoData data, AnalysisContext context) {
           System.out.println(data);
       }
   
       /**
        * 所有数据解析完成了 都会来调用
        *
        * @param context
        */
       @Override
       public void doAfterAllAnalysed(AnalysisContext context) {
           System.out.println("完成");
       }
   
   }
   ```

   2. 同步读

   ```java
   /**
    * 同步的读取，不推荐使用，如果数据量大会把数据放到内存里面
    */
   public static void synchronousRead() {
       // 写法2
       String fileName = "prodExcel.xlsx";
       // 这里 需要指定读用哪个class去读，然后读取第一个sheet 同步读取会自动finish
       List<DemoData> toatlDataList = EasyExcel.read(fileName).head(DemoData.class).sheet().doReadSync();
       for (DemoData demoData : toatlDataList) {
           System.out.println(demoData);
       }
   }
   ```

两种读对象的方式：

1. 确定表头：建立对象，和表头形成映射关系
2. 不确定表头：每一行数据映射为 Map<String, Object>

两种读取模式：

1. 监听器：先创建监听器、在读取文件时绑定监听器。单独抽离处理逻辑，代码清晰易于维护；一条一条处理，适用于数据量大的场景。
2. 同步读：无需创建监听器，一次性获取完整数据。方便简单，但是数据量大时会有等待时常，也可能内存溢出。

### 用户修改个人信息

1. 重写userUpdate，添加getLoginUser，迁移isAdmin

![image-20230921112906473](assets/image-20230921112906473.png)

![image-20230921113337257](assets/image-20230921113337257.png)

![image-20230921113435280](assets/image-20230921113435280.png)

![image-20230921113532147](assets/image-20230921113532147.png)

2. 修改UserController

![image-20230921113922404](assets/image-20230921113922404.png)

![image-20230921114205296](assets/image-20230921114205296.png)

![image-20230921114720510](assets/image-20230921114720510.png)

### 主页推荐

### 组队



## 前端开发

### 整合路由

Vue-Router 其实就是帮助你根据不同的 url 来展示不同的页面（组件），不用自己写 if / else

路由配置影响整个项目，所以建议单独用 config 目录、单独的配置文件去集中定义和管理。

有些组件库可能自带了和 Vue-Router 的整合，所以尽量先看组件文档、省去自己写的时间。

[安装 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/installation.html)

1.安装Vue Router

```sh
#如果安装失败，可能是项目正在运行，把项目停掉，删除node_modules目录，再安装
yarn add vue-router@4
```

![image-20230916102856564](assets/image-20230916102856564.png)

2. 引入vue-router

[入门 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/)

![image-20230916110556135](assets/image-20230916110556135.png)

![image-20230916110836957](assets/image-20230916110836957.png)

![image-20230916111026425](assets/image-20230916111026425.png)

![image-20230916111325663](assets/image-20230916111325663.png)

![image-20230916111459403](assets/image-20230916111459403.png)

3. 修改通用布局

   ![image-20230916151307992](assets/image-20230916151307992.png)

   编程式路由参考[编程式导航 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/essentials/navigation.html)

### 封装myAxios

1. 安装axios

```sh
#用来发请求
npm install axios
```

![image-20230919194157437](assets/image-20230919194157437.png)

创建实例[axios中文文档|axios中文网 | axios (axios-js.com)](http://axios-js.com/zh-cn/docs/#axios-create-config)

```js
import axios from "axios";
const my_axios = axios.create({
    baseURL: 'http://localhost:8080/api/',
    timeout: 100000,
    headers: {
        'Authorization': sessionStorage.getItem("token"),
    }
});
```

创建拦截器[axios中文文档|axios中文网 | axios (axios-js.com)](http://axios-js.com/zh-cn/docs/#拦截器)

```js
my_axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    console.log("我要发请求了",config)
    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
});

// 添加响应拦截器
my_axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    console.log("我收到响应了",response)
    return response
}, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
});

export default my_axios;
```

2. 允许请求携带Cookie

![image-20230921110151688](assets/image-20230921110151688.png)

### 封装获取当前登录用户功能

![image-20230921104224748](assets/image-20230921104224748.png)

### 搜索页

1.选择组件

![image-20230916114642295](assets/image-20230916114642295.png)

![image-20230916114750370](assets/image-20230916114750370.png)

![image-20230916131223513](assets/image-20230916131223513.png)

![image-20230916132158857](assets/image-20230916132158857.png)

2. 引入组件

![image-20230916144112168](assets/image-20230916144112168.png)

3. 编写搜索页

   1. 编写导航栏

      ![image-20230916144328890](assets/image-20230916144328890.png)

      ![image-20230916145103806](assets/image-20230916145103806.png)

      ![image-20230916145403396](assets/image-20230916145403396.png)

   2. 分割已选标签和选择标签

      ![image-20230916145221852](assets/image-20230916145221852.png)

   3. 渲染已选标签

      ![image-20230916150007197](assets/image-20230916150007197.png)

      ![image-20230916150254924](assets/image-20230916150254924.png)

   4. 引入TreeSelect组件

      ![image-20230916150405714](assets/image-20230916150405714.png)

      ![image-20230916150909719](assets/image-20230916150909719.png)

   5. 添加搜素页的路由

      ![image-20230916151107567](assets/image-20230916151107567.png)

   6. 最终效果

      ![image-20230916150959126](assets/image-20230916150959126.png)

### 用户信息页

把用户数据库中的用户信息展示到这个页面

1. 选择组件

   ![image-20230916152214015](assets/image-20230916152214015.png)

2. 编写用户类型-方便接受后端的数据

   ![image-20230916160254065](assets/image-20230916160254065.png)

3. 引入组件

   ![image-20230916160321644](assets/image-20230916160321644.png)

   ![image-20230916160455018](assets/image-20230916160455018.png)

   ![image-20230916160535677](assets/image-20230916160535677.png)

4. 效果

   ![image-20230916160600348](assets/image-20230916160600348.png)
   
   5. 修改用户信息页
   
   ![image-20230921105131541](assets/image-20230921105131541.png)
   
   ![image-20230921105342032](assets/image-20230921105342032.png)

### 用户信息修改页

1. 选择组件

   ![image-20230916163707773](assets/image-20230916163707773.png)

   ![image-20230916171923061](assets/image-20230916171923061.png)

2. 引入组件

   ![image-20230916172014807](assets/image-20230916172014807.png)

   ![image-20230916172354874](assets/image-20230916172354874.png)

   ![image-20230916172729570](assets/image-20230916172729570.png)

3. 添加路由

   ![image-20230916173335315](assets/image-20230916173335315.png)

4. 修改用户信息页

   ![image-20230916173316206](assets/image-20230916173316206.png)

5. 修改通用布局页

   ![image-20230916173448107](assets/image-20230916173448107.png)

6. 效果

   ![image-20230916173529752](assets/image-20230916173529752.png)

7. 修改用户信息修改页

![image-20230921105947435](assets/image-20230921105947435.png)

### 搜索结果页

1. 安装qs

```sh
#axios使用qs解决数组传参问题
npm i qs
```

在使用axios请求的过程中,直接在请求时传参数组，axios显示直接传数组去get请求时是 **tags[]=%E7%94%B7	&tags[]=Java** 我们如果想要没有 [] 连接的格式就需要进行`参数序列化`：使用`qs.stringify`,设置axios配置项中的 `paramsSerializer`

![image-20230919195802237](assets/image-20230919195802237.png)

```js
my_axios.get('/user/searchByTags', {
    params: {
      tags: tags
    },
    paramsSerializer: params => qs.stringify(params, {indices: false})
  })
//indices: false可以取消URL里数组的下标，如果不这么处理，后端收不到这个数组（名字因为加了下标而不匹配）
```

![image-20230919200130240](assets/image-20230919200130240.png)

3. 选择组件

![image-20230919200256361](assets/image-20230919200256361.png)

![image-20230919200353616](assets/image-20230919200353616.png)

4. 引入组件

![image-20230919200459410](assets/image-20230919200459410.png)

![image-20230919200947885](assets/image-20230919200947885.png)

5. 添加路由

![image-20230919201604552](assets/image-20230919201604552.png)

6. 修改搜索页

![image-20230919201123344](assets/image-20230919201123344.png)

![image-20230919201422277](assets/image-20230919201422277.png)

7. 修改user.d.ts文件

![image-20230919201800397](assets/image-20230919201800397.png)

8. 在搜素结果页发请求，获取数据，渲染页面

![image-20230919202710179](assets/image-20230919202710179.png)

![image-20230919204047018](assets/image-20230919204047018.png)

9. 效果

![image-20230919204220431](assets/image-20230919204220431.png)

![-](assets/image-20230919204238304.png)

### 登录页

1. 选择组件

![image-20230921104427097](assets/image-20230921104427097.png)

2. 引入组件

![image-20230921104605948](assets/image-20230921104605948.png)

![image-20230921104759490](assets/image-20230921104759490.png)

3. 添加路由

![image-20230921105414644](assets/image-20230921105414644.png)

### 主页

