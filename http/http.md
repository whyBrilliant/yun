# http

## Content-Type 存在于数据的头部，用于标识数据的类型。

1. Response Headers

    在响应中，Content-Type告诉客户端实际返回的内容的类型。比如返回JSON格式的数据，那么需要将Content-Type设置为application/json

2. Request Headers

    在请求中，客户端告诉服务器实际发送的数据类型。这里指**除get外**的请求，常见的有

    2.1 x-www-form-urlencoded, 常用的form表单默认的就是这种形式

    ```
    <form method="post">
    </form>
    ```

    比如我们要发送的数据

    ```
    {
      "name": "ken",
      "age": 26
    }
    ```

    会被编码为

    ```
      name=ken&age=26
    ```

    2.2 multipart/form-data, 常用语文件的上传


