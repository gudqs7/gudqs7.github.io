---
title: 文档生成插件更多细节分享
date: 2021-07-10 20:36:14
tags: 
  - idea
categories:
  - [idea-plugin]
  - [java]
---



# 文档生成插件更多细节分享



## 更多细节



### code 含义

#### 先看效果

![image-20210710205744491](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710205746.png)



#### 实现方式

在原来的 UserController 的基础上，再加一个方法

```java


    /**
     * 新增一个用户
     *
     * @param userEntity 用户信息
     * @return 操作是否成功
     * #code 1 用户名已存在 建议换个用户名再试
     * #code 2 手机号已存在 同上
     * #hiddenRequest userId,lastLoginTime
     * #hiddenResponse totalCount
     */
    @PostMapping("/addUser")
    public BaseResponse<Boolean> addUser(@RequestBody UserEntity userEntity) {
        // todo 新增用户
        log.info("addUser--> " + JsonUtils.getJsonString(userEntity));

        return BaseResponse.success(true);
    }


```

其中 ``#code` 开头的注释便是关键，这应该是一目了然了吧，先`#code` 开头，再空格分隔 code、含义、出现原因三个信息



### 根据不同方法隐藏不同字段（参数、返回值均可）

#### 先看效果

先写两个方法，使用 `#hiddenRequest、#hiddenResponse` 设置要隐藏的字段，这里注意方法参数变量名可以省略（即 `userEntity` ）

```java

    /**
     * 新增一个用户
     *
     * @param userEntity 用户信息
     * @return 操作是否成功
     * #code 1 用户名已存在 建议换个用户名再试
     * #code 2 手机号已存在 同上
     * #hiddenRequest userId,lastLoginTime
     * #hiddenResponse totalCount
     */
    @PostMapping("/addUser")
    public BaseResponse<Boolean> addUser(@RequestBody UserEntity userEntity) {
        // todo 新增用户
        log.info("addUser--> " + JsonUtils.getJsonString(userEntity));

        return BaseResponse.success(true);
    }

    /**
     * 修改用户信息
     *
     * @param userEntity 用户信息
     * @return 操作是否成功
     * #hiddenRequest lastLoginTime,userGender,userAge
     * #hiddenResponse totalCount
     */
    @PostMapping("/updateUser")
    public BaseResponse<Boolean> updateUser(@RequestBody UserEntity userEntity) {
        // todo 修改用户信息
        log.info("updateUser--> " + JsonUtils.getJsonString(userEntity));

        return BaseResponse.success(true);
    }

```





生成文档如图：

- 新增用户接口

<img src="https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710210659.png" alt="image-20210710210651803" style="zoom:50%;" />

- 修改用户接口

![image-20210710210846356](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710210847.png)

> 这里没展示返回字段，但效果是一样的，也是示例和说明表格都会隐藏

#### 实现方式

> 如我刚提的，使用 `#hiddenRequest、#hiddenResponse` 设置要隐藏的字段接口，这个设置仅对文档生成，导出到Postman起作用，不用担心影响代码运行。
>
> 这里需要注意的点是，多个字段之间使用英文逗号分隔。另外方法参数的变量名不需要带上，比如 `userId` ，直接写 `userId` 即可，不需要写 `userEntity.userId`，因为 Spring MVC 也是这么识别的，相应的我生成Postman时就需要这么处理，最终干脆保持统一，省的混乱！
>
> 还有一点就是，如果是纯方法的参数，是不支持设置隐藏的，原因如下：
>
> 1.首先，如果不需要用到此参数，可直接删除，而不像对象，可能多个地方用到，所以需要根据方法定制的去隐藏部分字段
>
> 2.其次，其实我还提供了另一种方式隐藏，甚至可以说，这个隐藏更早的被支持。那就是：在 @param 注释后面 加上 hidden即可，如图：

![image-20210710212211293](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710212212.png)



### 字段的更多注释

> 既然刚才说到了 hidden，那么就此引出字段级别的注释、注解，是的，我们还支持Swagger 注解，实际上注释也是根据Swagger注解定义出来的。

#### 字段级别

```java

    /**
     * 测试字段上的注释有哪些 tag
     * 字段注释说明是可以换行的
     * 当然有个前提就是
     * 最终负责渲染Markdown的引擎需要支持 <br> 标签，也就是支持部分 HTML
     * #required
     * #hidden false
     * #important
     * #example 测试示例值 此处可带空格，方法上的 @param 后面不行
     * #notes 对应其他参考信息 同理，可带空格，方法上的 @param 后面不行
     * #notes 谁说我不能换行的，只是没那么优雅罢了
     * #notes 不过也还凑合，换行
     */
    @ApiModelProperty(value = "这里是Swagger的字段说明\n试试换行", required = true, hidden = false,
            example = "测试示例值", notes = "对应其他参考信息\n试试换行")
    private String testField;

```



生成的示例如下：

```properties
userId:0
nickName:用户昵称
realName:用户姓名
phoneNumber:110
userGender:2
userAge:1024
userAvatar:https://mp.weixin.com/p/jewagheiajifejgihewjg.ifew
lastLoginTime:最近一次登录时间
testField:测试示例值 此处可带空格，方法上的 @param 后面不行
```

生成的字段说明表格如下：

![image-20210710221547548](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710221548.png)



使用的 tag 用法说明如下

- 啥都不加直接写，支持换行，对应字段说明表格的含义

- required 对应字段说明表格中的必填性，不写默认为 false（否），加上后代表必填
- hidden 加上即隐藏此字段，后面跟个 false 和不写效果一样（这个道理放到任何Boolean型tag都生效）
- important 加上代表注释比注解优先级高（默认注解优先级高，即写了注解优先使用注解中的信息）
- example 示例值，生成示例值和导出到Postman时有大用处，懂得都懂
- notes 对应字段说明表上的其他信息参考，若需要换行，请反复添加 notes



> 而其注解，则完全与 tag 同名，且默认注解优先级高，即写了注解优先使用注解中的信息！
>
> 还是推荐使用注释，毕竟不依赖于 Swagger！
>
> 这里顺带提下我对Swagger的看法，总得来说就是，对前端不够友好，其次就是依赖于代码的运行，一旦程序起不起来，就完蛋了，而插件就不一样了，哪怕是你有部分语法错误，只要不撞到参数的解析上，就没有关系，基本是确保方法声明不报错就OK了！而且运行一个 web 程序，始终是需要时间的，而我这插件的运行，只能说是比飞快还飞快；这里要感谢IDEA提供的Api，是IDEA提前解析好这些信息，我直接拿来用的，所以很快！



#### 方法级别注释

```java

    /**
     * 查询用户信息(分页)
     * 试试换行
     * #notes 方法的更多信息需要换行展示，毕竟内容很多啊
     * #notes 这样看起来如何！
     *
     * @param listUserPage 过滤条件
     * @param userEntities 没用的参数 hidden
     * @return 用户列表数据-分页
     * #hiddenRequest userEntities
     * #hiddenResponse data.userAvatar
     */
    @PostMapping("/listUser")
    public BaseResponse<List<ListUserResponse>> listUserPage(ListUserRequest listUserPage, List<UserEntity> userEntities) {
        // todo 查询分页数据
        log.info("listUserPage--> " + JsonUtils.getJsonString(listUserPage));

        return BaseResponse.success(new ArrayList<>(), 0L);
    }

```



文档效果图：

![image-20210710222036880](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710222038.png)



- 啥都不加直接写，支持换行，对应接口名标题（不建议换行，不太好看！！）

- code 不再赘述
- hiddenRequest 不再赘述
- hiddenResponse 不再赘述
- hidden 隐藏此方法，则怎么右键都不会生成文档！批量生成时用于隐藏单个，非常好使！
- important 代表即使使用了 `@ApiOperation` 注解也优先读取注释
- notes 写上，再看看文档的第二行，你就明白了！用于给方法添加更多说明，若需要换行，请反复添加 notes！



```java
// 另外注释有几个对应上面的
@ApiOperation(value = "类似啥都不写，使用\n换行", notes = "对应notes", hidden = true)
// 对应 code，可使用 @ApiResponses 添加多个 code
@ApiResponses({
  @ApiResponse(code = 1, message = "出错了"),
  @ApiResponse(code = 2, message = "出错了2"),
})

// 另外 hiddenRequest/hiddenResponse 不支持注解配置，必须使用注释咯，放心可以上面几个信息使用注解，然后搭配注释中的 hiddenRequest/hiddenResponse，这是OK的！
```





@param 单独说说

```java

    /**
     * 删除一个用户
     *
     * @param userId     用户id example=10000 required notes=更多信息&br;试试换行
     * @param operator   操作人 example=admin required
     * @param deleteFlag 是否彻底删除 hidden
     * @return 操作是否成功
     * #hiddenResponse totalCount
     */
    @GetMapping("/deleteUser")
    @Order(1)
    public BaseResponse<Boolean> deleteUser(Integer userId, String operator, Boolean deleteFlag) {
        // todo 删除一个用户
        if (deleteFlag != null && deleteFlag) {
            //todo 彻底删除数据
        } else {
            // todo 逻辑删除, 修改 deleted 字段值
        }

        return BaseResponse.success(true);
    }

```



文档效果图：

![image-20210710222919511](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710222920.png)

> 首先 `@param userId     用户id` ； `@param` 后面紧跟的字段名和字段说明，这个是 javadoc 就这么写，后面的内容，则支持几个 tag，如下

- hidden 隐藏此参数
- example 设置示例值
- notes 设置更多信息

> 这里必须提一下，notes说明，example都不能有空格，否则影响插件解析注释。不过有个方式可以换行，即添加 `&br;`，这里不能直接加 `<br>`，因为插件的代码会转义所有的 `< 和 >` 符号。



对应注解： 

```java
// 除了 notes，其他都对的上
@ApiParam(value = "1", required = true, hidden = true, example = "2")
```



最后提一嘴 @return ，这个 javadoc，idea 敲 `/**` 回车自动生成的 tag，我也有解析，但一般情况还用不到，只有返回值是普通类型（如 Integer，String，Double，Date）时才会有效，这个大家试试就知道了！另外 @return 不支持 @param 哪些附加 tag（如 hidden，example，notes）！



#### 类级别注释

```java
/**
 * 用户接口
 * @description 用户接口
 * @author wq
 * #tags 用户接口
 * #hidden
 * #important
 */
@Api(description = "用户接口", tags = "用户接口", hidden = true)
@Slf4j
@RestController
@RequestMapping("/api/v1/user")
public class UserController {
}
```



效果：批量生成时的文件名会从注释、注解中取

- 什么都不写，类的说明
- description 和什么都不写效果一样
- tags 参考 Swagger 的注解含义，代表对接口的分类，用来对应 @Api 注解的 tags
- hidden 批量生成时过滤此类
- important 提高注释的优先级（默认 @Api 注解优先级高）



注解作用类似！

> 上面列的 description，tags，以及啥都不写，本质都用于批量生成接口时作为Markdown文件的文件名或导出到Postman时作为请求的上级文件夹名称，优先级最高的是 tags，这是考虑到使用了 Swagger 的项目，一般这个地方都会填写中文含义。其次是
>
> description 和 啥都不写，看谁写在前面，则谁优先！这个是考虑到好多人使用 javadoc 写类注释的模板都是带有这个 @description 用于描述改类的。
>
> 啥都不写则是我个人认为比较好的方式！

效果如图：

![image-20210710225904864](https://gitee.com/gudqs7/many-images/raw/master/Mac-PicGo/20210710225906.png)





#### 总结

> 以上所有我使用 # 开头的 tag，本质上都可替换成 @（除了 @param、@return），比如
>
> #hidden 和 @hidden 效果是一样的！
>
> 其实 @ 开头才是比较正宗的注释的 tag 写法，但太正宗了也不行，IDEA识别了，但只识别了一半，只认识部分tag，这就是使得我们写的这些 tag 都要报个黄色警告（如果你打开了Editor => Inspections）；所以干脆加了个 # 开头，避开idea识别，合适的不得了！
>
> 其实都支持，怎么用就看大家了，我个人偏爱 # 开头！！！



> 本来打算分享部分核心代码的，太晚了，算了，开源吧，但不保证开源的是最新版！！
>
> 项目地址：[Github：open-doc-generator-idea-plugin](https://github.com/gudqs7/open-doc-generator-idea-plugin)

