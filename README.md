# 基于lin-cms-tp5文件上传类扩展
## 概述

在cms的绝大数业务或者功能中，我们都是以数据流的形式来处理数据的，但是有些情况下，比如我们需要存储一个图片，或者一段视频。这些数据都是以文件的形式存在，它们皆以硬盘为载体。

你可以设想这样的一个业务，leader要求你可以从前端操作页面上传一个图片，又要求上传一份报表或者一份PPT，于是你开始忙碌起来，开发一个上传图片的接口，又上传报表的接口。
但是这两者完全耦合了，于是你干脆开发一个上传文件的接口，但是这个接口设计的颇为复杂。此时，次扩展诞生了，帮你一下子解决这个问题。
首先我们还要继承`file`文件这个基类的实现类必须重写`upload`方法。如实例：

```php
class LocalUploader extends File{

    //需要重写的文件上传的方法
    public function upload(){
    
    }
}
```
具体控制器方法可以查看[`lin-cms-tp5`](https://github.com/ChenJinchuang/lin-cms-tp5.git)中上传接口`cms/file`这个方法。
当需要实现其他的上传类时，如上传到阿里云OSS,我们只需要重新定义个Aliyun类，在类中重新实现这个`upload`方法即可。

## 实操
首先，我们了解一个file的具体配置
```php
    return [
        "store_dir" => './uploads',       # 文件的存储路径
        "single_limit" => 1024 * 1024 * 2, # 单个文件的大小限制，默认2M
        "total_limit"=> 1024 * 1024 * 20, # 所有文件的大小限制，默认20M
        "nums" => 10,                      # 文件数量限制，默认10
        "include" => [],                   # 文件后缀名的排除项，默认排除[]，即允许所有类型的文件上传
        "exclude" => []                   # 文件后缀名的包括项
    ];
```
在单个配置的后面，已经注释的方式解释了每项的作用，当然还需要着重解释一个`exclude`和`include`这两项，默认情况下，只会读取它们中的一项，读取其中不为`undefined`的一项，如果两项皆为`undefined`的话，会通过所有文件类型，如果二者都有则取`inclde`为有效配置。总之，系统只会支持使用一项，二者都为`undefined`的情况下，则通过所有文件类型。

做好这些后，使用`postman`测试一下文件上传。
上传成功后，得到如果结果：
```php
    [
        {
            "id": 19,
            "key": "one",
            "path": "2019/07/17/e892f8da-a840-11e9-8088-c4b30191a4f8.png",
            "url": "http://127.0.0.1:5000/assets/2019/07/17/e892f8da-a840-11e9-8088-c4b30191a4f8.png"
        },
        {
            "id": 20,
            "key": "two",
            "path": "2019/07/17/e89816ba-a840-11e9-9f59-c4b30191a4f8.png",
            "url": "http://127.0.0.1:5000/assets/2019/07/17/e89816ba-a840-11e9-9f59-c4b30191a4f8.png"
        }
    ]

```
由于上传了两个文件，因此我们得到了两个元素的数组，他们的结构如下

| name | 说明 | 类型 |
| :-----:| :----: | :----: |
| id | 文件存储到数据的id | string |
| key | 文件上传的key | string |
| path | 文件的服务器本地相对路径 | string |
| url | 可访问的url | string |

>系统会自动帮助我们上传的文件做md5,因此大可不必担心文件重复上传，如果你上传了重复的文件，服务器会返回已传文件的数据
