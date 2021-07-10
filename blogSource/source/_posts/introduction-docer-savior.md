---
title: 文档生成插件使用指南
date: 2021-07-09 11:07:44
tags: 
  - idea
categories:
  - [idea-plugin]
  - [java]
---



# 文档生成插件使用指南



## 根据 Spring MVC Controller 下方法生成接口文档

### 先看示例代码

#### UserController

```java
package cn.gudqs.business.docer.controller;

import cn.gudqs.business.docer.dto.request.ListUserRequest;
import cn.gudqs.business.docer.dto.response.BaseResponse;
import cn.gudqs.business.docer.dto.response.ListUserResponse;
import cn.gudqs.util.JsonUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * 用户接口
 * @author wq
 */
@Slf4j
@RestController
@RequestMapping("/api/v1/user")
public class UserController {

    /**
     * 查询用户信息(分页)
     * @param listUserPage 过滤条件
     * @return 用户列表数据-分页
     */
    @PostMapping("/listUser")
    public BaseResponse<List<ListUserResponse>> listUserPage(ListUserRequest listUserPage) {
        // todo 查询分页数据
        log.info("listUserPage--> "+ JsonUtils.getJsonString(listUserPage));

        return BaseResponse.success(new ArrayList<>(), 0L);
    }

}

```

#### BaseResponse

```java
package cn.gudqs.business.docer.dto.response;

import lombok.Data;

/**
 * @author wq
 */
@Data
public class BaseResponse<T> {

    /**
     * 状态码
     * 0: 代表成功
     * -1: 代表未知异常
     * >0: 代表已知异常
     */
    private Integer code;

    /**
     * 错误信息
     */
    private String message;

    /**
     * 是否成功
     * true: 代表成功, 此时 code = 0
     * false: 代表失败, 此时 code != 0
     */
    private Boolean success;

    /**
     * 返回数据
     */
    private T data;

    /**
     * 分页-总条数
     */
    private Long totalCount;

    public static <T> BaseResponse<T> success() {
        return success(null);
    }

    public static <T> BaseResponse<T> success(T data) {
        return success(data, 0L);
    }

    /**
     * 返回一个成功的 BaseResponse
     * @param data 携带的数据
     * @param totalCount 分页-总条数
     * @return BaseResponse
     */
    public static <T> BaseResponse<T> success(T data, Long totalCount) {
        BaseResponse<T> baseResponse = new BaseResponse<>();
        baseResponse.setSuccess(true);
        baseResponse.setCode(0);
        baseResponse.setMessage("success");
        baseResponse.setData(data);
        baseResponse.setTotalCount(totalCount);
        return baseResponse;
    }

}
```

#### ListUserResponse

```java
package cn.gudqs.business.docer.dto.response;

import lombok.Data;

import java.util.Date;

/**
 * 用户信息
 *
 * @author wq
 */
@Data
public class ListUserResponse {

    /**
     * 用户昵称
     */
    private String nickName;

    /**
     * 用户姓名
     */
    private String realName;

    /**
     * 用户手机号
     * #example 110
     */
    private String phoneNumber;

    /**
     * 用户性别
     * 0: 保密
     * 1: 男
     * 2: 女
     * #example 2
     */
    private Integer userGender;

    /**
     * 用户年龄
     * #example 1024
     */
    private Integer userAge;

    /**
     * 用户头像地址
     * #example https://mp.weixin.com/p/jewagheiajifejgihewjg.ifew
     */
    private String userAvatar;

    /**
     * 最近一次登录时间
     */
    private Date lastLoginTime;

}
```

#### ListUserRequest

```java
package cn.gudqs.business.docer.dto.request;

import lombok.Data;
import lombok.EqualsAndHashCode;

import java.util.Date;

/**
 * 查询用户-过滤条件
 *
 * @author wq
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class ListUserRequest extends BasePageRequest {

    /**
     * 模糊搜索词
     * 支持用户昵称, 用户姓名, 用户手机号
     */
    private String searchKeyword;

    /**
     * 用户性别
     * 0: 保密
     * 1: 男
     * 2: 女
     * #example 2
     */
    private Integer gender;

    /**
     * 过滤年龄范围-起始
     */
    private Integer ageStart;

    /**
     * 过滤年龄范围-结束
     */
    private Integer ageEnd;

    /**
     * 过滤登录时间范围-开始
     */
    private Date loginTimeStart;

    /**
     * 过滤登录时间范围-结束
     */
    private Date loginTimeEnd;

}
```

#### BasePageRequest

```java
package cn.gudqs.business.docer.dto.request;

import lombok.Data;

/**
 * @author wq
 */
@Data
public class BasePageRequest {

    /**
     * 分页-当前页码
     * #example 1
     */
    private Integer pageNumber;

    /**
     * 分页-分页大小
     * #example 20
     */
    private Integer pageSize;

}
```

### 说明

> 以上都是大家常写的分页代码，此时我在 `listUserPage` 这个方法上右键，弹出菜单中选择：生成Api接口文档（restful），如图

![image-20210710095313814](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710095319.png)



然后就会得到一个弹框，当然如果你根据提示点了 `OK`，则直到下次重启，都不会再出现

![image-20210710100138879](/Users/wq/Library/Application Support/typora-user-images/image-20210710100138879.png)



弹框中内容如下，此内容区域可滚动预览，全选复制粘贴

```markdown
# 查询用户信息(分页)
## 请求信息

### 请求地址
​```
http://192.168.0.104:8080/api/v1/user/listUser
​```

### 请求方法
​```
POST
​```

### 请求体类型
​```
application/x-www-form-urlencoded
​```

## 入参
### 入参示例(Postman==> Bulk Edit)
​```json
pageNumber:1
pageSize:20
searchKeyword:模糊搜索词<br>支持用户昵称, 用户姓名, 用户手机号
gender:2
ageStart:0
ageEnd:0
loginTimeStart:过滤登录时间范围-开始
loginTimeEnd:过滤登录时间范围-结束
​```

### 入参字段说明
| **字段** | **类型** | **必填性** | **含义** | **其他信息参考** |
| -------- | -------- | -------- | -------- | -------- |
| pageNumber     | **Integer**     | 否  |  分页-当前页码 |   |
| pageSize     | **Integer**     | 否  |  分页-分页大小 |   |
| searchKeyword     | **String**     | 否  |  模糊搜索词<br>支持用户昵称, 用户姓名, 用户手机号 |   |
| gender     | **Integer**     | 否  |  用户性别<br>0: 保密<br>1: 男<br>2: 女 |   |
| ageStart     | **Integer**     | 否  |  过滤年龄范围-起始 |   |
| ageEnd     | **Integer**     | 否  |  过滤年龄范围-结束 |   |
| loginTimeStart     | **Date**     | 否  |  过滤登录时间范围-开始 |   |
| loginTimeEnd     | **Date**     | 否  |  过滤登录时间范围-结束 |   |






## 出参
### 出参示例
​```json
{
  "code": 0,
  "message": "错误信息",
  "success": false,
  "data": [
    {
      "nickName": "用户昵称",
      "realName": "用户姓名",
      "phoneNumber": "110",
      "userGender": 2,
      "userAge": 1024,
      "userAvatar": "https://mp.weixin.com/p/jewagheiajifejgihewjg.ifew",
      "lastLoginTime": "最近一次登录时间"
    }
  ],
  "totalCount": 0
}
​```

### 返回字段说明
| **字段** | **类型** | **含义** | **其他信息参考** |
| -------- | -------- | -------- | -------- |
| code     | **Integer**     | 否  |  状态码<br>0: 代表成功<br>-1: 代表未知异常<br>\>0: 代表已知异常 |   |
| message     | **String**     | 否  |  错误信息 |   |
| success     | **Boolean**     | 否  |  是否成功<br>true: 代表成功, 此时 code = 0<br>false: 代表失败, 此时 code != 0 |   |
| data     | **List\<ListUserResponse\>**     | 否  |  返回数据 |   |
|└─ nickName     | **String**     | 否  |  用户昵称 |   |
|└─ realName     | **String**     | 否  |  用户姓名 |   |
|└─ phoneNumber     | **String**     | 否  |  用户手机号 |   |
|└─ userGender     | **Integer**     | 否  |  用户性别<br>0: 保密<br>1: 男<br>2: 女 |   |
|└─ userAge     | **Integer**     | 否  |  用户年龄 |   |
|└─ userAvatar     | **String**     | 否  |  用户头像地址 |   |
|└─ lastLoginTime     | **Date**     | 否  |  最近一次登录时间 |   |
| totalCount     | **Long**     | 否  |  分页-总条数 |   |





```

> 由于文章本身就是Markdown格式，所以我用截图来展示下效果：截图使用 Chrome 插件打开 PDF 滚动截屏得到，PDF 是由 typora 生成的

![screencapture-file-Users-wq-File-TempFile-html-2021-07-10-10_09_50](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710101322.png)



### 总结

> 如果有多个方法，想一次性生成，则可在 UserController 上右键，点击生成Api文档（restful） ，如图：

![image-20210710101618777](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710101619.png)



> 此时生成的文档，就是多个方法单独生成的文档的简单的拼接。



## 根据 Spring MVC Controller 的接口信息导出到 Postman

### 项目视图选择类、包，点右键

> 还是沿用上面的代码，但是考虑到单个方法单独导出到 Postman 非常费时费力，可用性不高，因此方法级别的右键导出，并没有实现！但是单个文件级别，还是OK的，只需在文件导航视图找到 UserController 类，右键，点击 导出Api到Postman即可！如图

![image-20210710102303483](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710102304.png)



> 另外这个右键，其实不管是类，包，还是文件夹，乃至根目录，都可以点击导出到 Postman，只是导出的范围不同！
>
> 由于范围比较大，因此过滤了非 @Controller、@RestController 注解的类，可放心使用。

点击导出后，会将 Postman 的json 生成到项目下的 api-doc/postman 文件夹，如图

![image-20210710102846460](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710102848.png)

### 打开Postman点击 File --> Import

此时我们打开 Postman，使用快捷键 `Ctrl + O` 或 `Command + O` , 接着拖拽文件过去，或点击 UploadFiles，弹框选择文件打开，如图

![image-20210710103242732](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710103245.png)



![image-20210710103321866](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710103323.png)



### 最终得到一个 Collection

如图：不管是请求，返回示例，文档，或者说 url 变量，body和参数处理，都帮你搞定了，你只需要修改下参数的实际值（如果使用 #example，则这步也可以省略），然后点蓝色的 Send 按钮就 ok 了（千万别忘记启动项目）！

> 这里简单说下，url 的组成
>
> 1.http:// 这个本地开发一般不用 https 对吧，嘿嘿
>
> 2.ip: 这个使用代码遍历网卡，过滤掉 ipv6，127.0.0.1 后得出的局域网 ip，如果没搞虚拟机啥的，一般获取比较准确，要还是错了，点击 Collection 名称，在 Variables 下修改 {{template-url}} 的 Current Value 就好了
>
> 3.port: 这个默认8080，然后支持 Spring Boot 项目，即读取 application.yml/application.properties 文件获取 server.port 的值，嗯~~ 若存在 application-xxx 这种情况，还真不好取，只能读取 spring.profiles.active 但是也未必准确，毕竟有很多地方支持设置。理论性idea的东西都可以获取，但idea的插件开发文档找不到这方面的信息，大概是不会花精力搞了，毕竟错了直接改变量就好了，改一处，所有请求生效。
>
> 4.剩下的部分就是 Controller 的 @RequestMapping 注解 + 方法的 @RequestMapping（或@GetMapping之类）注解的值拼凑起来。若方法为 GET，则参数会以?xxx=xxx&xx=xx 形式拼接上去，也可在 Params 下查看

![image-20210710104427563](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710104429.png)



## 总结

> 总得来说，这个插件还是太简单了，其实不用使用指南，知道在哪右键起什么作用，就OK了，因为处了右键点击，其他一概不用你管（包括复制）！可谓是偷懒极了。。。。。 
>
> 

说下这个插件的发展史。

> 一开始是用于微服务的接口方法，根据参数和返回值来生成文档给其他开发看，当时还是自己写了个 Util，然后传 Class 作为参数来解析得到参数、返回值等信息，最终生成文档，当时的所有中文备注，都是通过 Swagger 的注解来实现的，没办法，运行时注释早干掉了，得不到。
>
> 后来觉得这东西可以抽出来，不要被项目束缚，先是想到了idea插件，但是发现 Api 及其不同，idea中也无法获取 Class，也就是当时写的代码无法复用，于是又考虑 maven 插件，发现这个是可以取到 Class，进行复用的。
>
> 写完后，又发现使用这个插件，必须设置参数，也就是你要生成那个类的文档，很麻烦啊这，于是想到之前看idea插件的一些api可以获取到这些信息，干脆来了个结合，idea右键获取Class 的全限定名，传给maven插件，maven插件来生成，最后idea弹框展示。其实这样也不错，但是又有个巨大的问题，就是maven插件其实依赖于 target 文件中的信息，这部分必须 compile 后才有，使得每次运行插件都要先跑compile 再生成，本来maven运行速度就慢，这样就更慢了。加上进度条后还是感觉体验极差！！！
>
> 最终，在某个夜黑风高的晚上，我死活睡不着觉，干脆打开电脑，点击备忘录，记下明天摸鱼时间要写个纯idea插件，然后躺下数羊！啊那是不可能的。
>
> 不过这插件最终确实都是在闲的没活干的摸鱼时间写的，最终把原来代码拷贝一份到idea插件项目中，一边参照一边使用idea提供的Api修改代码，最终得到了纯idea插件版本！
>
> 之后遇到了比较坑的点就是泛型的解析和参数的解析，目前都解决了，可以说是随便怎么写，都能识别了。比如最上面的示例代码的 `T data`，这还是小 case，如果改成 `List<T>` 呢? 如果再来一个带泛型的类 `PageInfo<T>` ,而我加个字段 `PageInfo<T> pageData; `来把 T 传给 PageInfo 呢，解析会有问题吗？刚开始会，后来解决了！！！只能说我自己都快绕晕了，但最终还是解决了。这其实值得复盘的，不过我这人比较懒，就不搞了。其实这种情况繁多的时候，要写代码不能只考虑部分情况，或是来一种情况加一种情况，必须找到通用的方法，当然未必什么时候都有通用的方法，不过还是要去追求的！！



后续还会和大家分享一些我插件中的核心代码，包括idea插件提供的api，自己百度找的工具类等等。

另外我上面说的微服务那个接口文档，其实就是不带（restful）后缀的菜单选项，其生产的文档不含有HTTP协议相关的信息，而是纯粹的入参、出参、code含义等。另外文件视图右键有三个选项，另外另个就是批量生成 Markdown文件，毕竟一个一个复制粘贴很累的，而很多笔记软件都支持批量导入Markdown文件，这样就不就简单了。