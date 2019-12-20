---
layout: post
title:  "RESTful 创建文件接收 API"
date:   2019-12-17 20:57:22 +0800
categories: Flask
tags: [python, flask, RESTful, 文件上传]
---
文件「文本、文档、图片等等」是一个服务器不可缺少的部分，在 [使用 Flask 创建 RESTful 服务](https://mp.weixin.qq.com/s/kR6oiyYGDdhhpD9HGMy7ZA) 介绍了如何使用 Flask 创建一个支持 RESTful API 的服务器。这篇文章介绍如何使用 RESTful API 来完成文件的接收，并将文件保存在静态目录下。

以下是文件接收的代码「这是实现的是图片的接收」：
```python
    parse = reqparse.RequestParser()
    parse.add_argument('image', type=werkzeug.datastructures.FileStorage, location='files')
    args = parse.parse_args()
    stream = args.get('image')
    basepath = current_app.config['BASEDIR']
    upload_path = os.path.join(basepath, "server/static/uploads", secure_filename(stream.filename))
    stream.save(upload_path)
```

> The FileStorage class is a thin wrapper over incoming files. It is used by the request object to represent uploaded files. All the attributes of the wrapper stream are proxied by the file storage so it’s possible to do storage.read() instead of the long form storage.stream.read().

以上代码实现通过参数传输图片上传至服务端，在服务端以文件流的方式读取文件并将文件保存到服务器的静态文件目录下。

以下是通过 Postman 测试文件上传 API 的配置方式。
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/截屏2019-12-1下午9.33.04.png)

在 Anddroid 下可以是 Retrofit 来完成文件的上传示例代码如下：
```java
public class Server {
    private static final String TAG = "Server";

    public interface UserAPIs {
        @Multipart
        @POST("/image")
        Call<ResponseBody> uploadImage(@Part MultipartBody.Part file,
                                       @Part("name") RequestBody requestBody);
    }

    public static void uploadImageToServer(Context context, String filePath, Callback callback) {
        Retrofit retrofit = NetworkClinet.getRetrofitClinet(context);
        UserAPIs uploadAPIs = retrofit.create(UserAPIs.class);
        //Create a file object using file path
        File file = new File(filePath);
        // Create a request body with file and image media type
        RequestBody fileReqBody = RequestBody.create(file, MediaType.parse("image/*"));
        // Create MultipartBody.Part using file request-body,file name and part name
        MultipartBody.Part part = MultipartBody.Part.createFormData("image", file.getName(), fileReqBody);
        //Create request body with text description and text media type
        RequestBody description = RequestBody.create("image-type", MediaType.parse("text/plain"));

        Call call = uploadAPIs.uploadImage(part, description);
        call.enqueue(callback);
    }
}

final class NetworkClinet {
    private static final String BASE_URL = "http://127.0.0.1:5000/";
    private static Retrofit retrofit;

    public static Retrofit getRetrofitClinet(Context context) {
        if (retrofit == null) {
            OkHttpClient okHttpClient = new OkHttpClient.Builder()
                    .connectTimeout(30, TimeUnit.SECONDS)
                    .writeTimeout(30, TimeUnit.SECONDS)
                    .readTimeout(30, TimeUnit.SECONDS)
                    .build();
            retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .client(okHttpClient)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();
        }
        return retrofit;
    }
}
```
