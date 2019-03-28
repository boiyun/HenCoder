# HTTP 
> ⼀种⽹络传输协议，位于 TCP / IP 协议族的最顶层——应用层.用于传输超文本的协议。和 HTML (Hypertext Markup Language 超⽂本标记 语言) 一起诞⽣生，⽤于在⽹络上请求和传输 HTML 内容。

## HTTP的工作方式：
### 浏览器：
用户输入地址后回车或点击链接->浏览器拼装HTTP报文并发送给服务器，服务器处理后发送响应报文发回给浏览器->浏览器解析响应报文并使用渲染引擎显示到界面。
### 手机App：
用户点击或界面自动触发联网需求->Android代码调用拼装HTTP报文并发送请求到服务器->服务器处理请求后发送响应报文给手机->Android代码处理响应报文并作出相应处理（如存储数据、加工数据、显示数据到界面）
## URL格式：
协议类型：//服务器地址[：端口号]路径   
http://hencoder.com/users?gender=male
## 请求报文
请求报文格式：请求行、Headers、Body 

* 请求行:Method、Path、HTTP Version
* Headers：请求的meta data（元数据）
* Body：要发送给服务器的内容

![](/Users/chenbo/Desktop/hencoder_pic/request.png)
## 响应报文
响应报文格式：状态行、Headers、Body

* 状态行:HTTP Version、Status Code、Status Message
* Headers：Host、Content-Type/Content-Length、Location、User-Agent、Range/Accept-Range
* Body：要发送给服务器的内容

![](/Users/chenbo/Desktop/hencoder_pic/response.jpeg)
## Request Methods 请求方法
* `GET`：获取资源；对服务器数据不进行修改；请求时不发送Body；幂等（即反复调用多次时会得到相同的结果）  

    对应Retrofit的代码：

    ```
    @GET("/users/{id}")   
    Call<User> getUser(@Path("id") String id, @Query("gender") String gender);
    ```
* `POST`：增加或修改资源；有Body；非幂等
    
    对应Retrofit的代码：

    ```    @FormUrlEncoded
    @POST("/users")
    Call<User> addUser(@Field("name") String name, @Field("gender") String gender);
    ```
* `PUT`：修改资源；有Body；幂等

    对应Retrofit的代码：

    ```    @FormUrlEncoded
    @PUT("/users/{id}")
    Call<User> updateGender(@Path("id") String id, @Field("gender") String gender);
    ```
* `DELETE`：删除资源；没有Body；幂等

    对应Retrofit的代码：

    ```    @DELETE("/users/{id}")
    Call<User> getUser(@Path("id") String id, @Query("gender") String gender);
    ```
 
* HEAD：相当于没有返回响应体的GET；

## Status Code 状态码
> 作用：对结果作出类型化描述（如：「获取成功」「内容未找到」） 

* 1xx：临时性消息，100（继续发送）101（正在切换协议）
* 2xx：成功,200（成功），201（创建成功）
* 3xx：重定向；301（永久迁移），302（临时迁移），304（内容未改变，使用浏览器缓存） 
* 4xx：客户端错误；400（客户端请求错误）；401（未授权，认证失败）；403（被禁止）404（页面找不到），
* 5xx：服务端错误；500（服务器内部错误）

## Header 首部
> 作用：HTTP消息的元数据（metadata）「关于数据的数据」
	
* Host：服务器主机地址(目标主机)     
   如：hencoder.com 通过 域名系统-DNS（Domain Name System）查询出ip地址
  （仅用于找到目标主机后确认主机域名和端口）   
 ` 注意:`不是在网络中寻址的，而是在目标服务器上用于定位子服务器的。
* Content-Type：指定Body的类型
   *  `text/html:` 请求Web页面时返回响应的类型，Body中返回html文本
   *  `application/x-www-form-urlencoded:`普通表单(field) 
        
      对应Retrofit的代码：

        ```        @FormUrlEncoded
        @POST("/users")
        Call<User> addUser(@Field("name") String name, @Field("gender") String gender);
    ```
   * `multipart/form-data:`二进制数据，如上传图片，文件
        格式如下：
        
        ```        POST  /users  HTTP/1.1
        Host: hencoder.com
        Content-Type: multipart/form-data; boundary=----
        WebKitFormBoundary7MA4YWxkTrZu0gW
        Content-Length: 2382
        ```
         
        ```        ------WebKitFormBoundary7MA4YWxkTrZu0gW
        Content-Disposition: form-data; name="name"
        rengwuxian
        ------WebKitFormBoundary7MA4YWxkTrZu0gW
        Content-Disposition: form-data;name="avatar";filename="avatar.jpg"
        Content-Type: image/jpeg
        JFIFHHvOwX9jximQrWa......
        ------WebKitFormBoundary7MA4YWxkTrZu0gW--
        ```
           
      对应Retrofit的代码：

        ```                @Multipart
        @POST("/users")
        Call<User> addUser(@Part("name") RequestBody name, @Part("avatar") RequestBody avatar);

        RequestBody namePart = RequestBody.create(MediaType.parse("text/plain"),
        nameStr);
        RequestBody avatarPart = RequestBody.create(MediaType.parse("image/jpeg"),
avatarFile);
api.addUser(namePart, avatarPart);
    ```
      boundary: 分块(part)（浏览器或okhttp自动生成）
   * `application/json:`json形式，用于Web Api的响应或POST/PUT请求
     格式如下：
        
        ```        POST /users HTTP/1.1
        Host: hencoder.com
        Content-Type: application/json; charset=utf-8
        Content-Length: 38
        {"name":"rengwuxian","gender":"male"}
        ```
     对应Retrofit的代码：

        ```         @POST("/users")
        Call<User> addUser(@Body("user") User user);
        // 需要使⽤用 JSON 相关的 Converter
        api.addUser(user);
    ```
   * image/jpeg：单文件(知道的人少，几乎不用)

      请求中提交二进制数据:
      
        ```         POST /user/1/avatar HTTP/1.1
        Host: hencoder.com
        Content-Type: image/jpeg
        Content-Length: 1575
        JFIFHH9......
        ```
    
      对应 Retrofit 的代码:
      
        ```                  @POST("users/{id}/avatar")
        Call<User> updateAvatar(@Path("id") String id, @Body RequestBody avatar);

        RequestBody avatarBody = RequestBody.create(MediaType.parse("image/jpeg"),
        avatarFile);
        api.updateAvatar(id, avatarBody)
        ```
   * application/zip：单文件(几乎不用)
   
* `Content-Length`：指定Body的长度（字节）
* `Transfer-Encoding:chunked`（分块传输编码Chunked Transfer
   Encoding）
    用于当响应发起时，内容长度还没能确定的情况下。和Content-Length不同时使用。更快对客户端作出响应，减少用户等待.  
* Location：指定重定向的目标URL
* User-Agent：用户代理，即是谁实际发送请求、接受响应的，例如手机浏览器、手机app。
* Range/Accept-Range：按范围取数据。作用：断点续传、多线程下载。       
    `Accept-Range: bytes` 响应报⽂中出现，表示服务器⽀持按字节来取范围数据  
    `Range: bytes=<start>-<end>` 请求报⽂中出现，表示要取哪段数据   
    `Content-Range:<start>-<end>/total` 响应报⽂中出现，表示发送的是哪段数据

## 其他Headers
* Accept：客户端能接受的数据类型。如 text/html
* Accept-Charset：客户端接受的字符集。如 utf-8
* Accept-Encoding：客户端接受的压缩编码类型。如 gzip
* Content-encoding: 压缩类型。gzip,deflate,br

## Cache
> 作用：在客户端或中间⽹络节点缓存数据，降低从服务器取数据的频率，以提⾼⽹络性能。

## REST
扔物线的观点:   
`REST HTTP 即正确使用 HTTP`包括:   

* 使⽤资源的格式来定义 URL   
* 规范地使用 method 来定义⽹络请求操作    
* 规范地使用 status code 来表示响应状态 
* 其他符合 HTTP 规范的设计准则   


