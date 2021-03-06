## 升级到 PHP SDK V5

如果您细心对比过 PHP SDK V4 和 V5 的文档，您会发现并不是一个简单的增量更新。COS V5 在架构、可用性和安全性上有了非常大的提升，而且在易用性、健壮性和传输性能上也做了非常大的改进。如果您想要升级到 PHP SDK V5，请参考下面的指引，完成 PHP SDK 的升级工作。

### 功能对比

下表列出了 V4 和 V5 的主要功能对比：

| 功能       | V5         | V4                         |
| -------- | :------------: | :------------------:    |
| 文件上传 | 支持本地文件、字节流、输入流上传<br>默认覆盖上传<br>智能判断上传模式<br>简单上传最大支持5GB<br>分块上传最大支持48.82TB（50,000GB） | 只支持本地文件上传<br>可选择是否覆盖<br>需要手动选择是简单还是分片上传<br>简单上传最大支持20MB<br>分片上传最大支持64GB |
| 文件下载 | 支持断点续传 | 不支持断点续传 |
| 文件删除 | 支持批量删除 | 只支持单文件删除 |
| 存储桶基本操作 | 创建存储桶<br>获取存储桶<br>删除存储桶   | 不支持 |
| 存储桶ACL操作 | 设置存储桶ACL<br>获取设置存储桶ACL<br>删除设置存储桶ACL   | 不支持 |
| 存储桶生命周期 | 创建存储桶生命周期<br>获取存储桶生命周期<br>删除存储桶生命周期 | 不支持 |
| 目录操作 | 不支持   | 创建目录<br>查询目录<br>删除目录 |


## 升级步骤

1. 更新您的 PHP SDK
2. 我们的鉴权方式发生了变化，建议后台集成我们的临时密钥（STS）方案，并提供接口，客户端携带获取到的临时密钥执行 COS 请求
3. 根据指引修改 SDK 的初始化方式
4. 我们的 `存储桶名称` 和 `可用区域简称` 有了更新，请对应修改
5. 一些操作的 API 发生了变化，我们做了封装让 SDK 更加易用，具体请参考我们的示例和 [接口文档](https://cloud.tencent.com/document/product/436/12267)

**1. 更新 PHP SDK**

有三种方式安装 COS PHP SDK V5：
* Composer 方式
* Phar 方式
* 源码方式

#### Composer 方式
推荐用户使用 Composer 安装 cos-php-sdk-v5，Composer 是PHP的依赖管理工具，允许您声明项目所需的依赖，然后自动将它们安装到您的项目中。

>?您可以在 [Composer 官网](https://getcomposer.org/) 上找到更多关于如何安装 Composer，配置自动加载以及用于定义依赖项的其他最佳实践等相关信息。

**使用 Composer 安装 COS-PHP-SDK-V5**
1. 打开终端
2. 下载 Composer
```
curl -sS https://getcomposer.org/installer | php
```
3. 创建一个名为`composer.json`的文件，内容为
```
{
    "require": {
        "qcloud/cos-sdk-v5": "1.*"
    }
}
```
4. 使用 Composer 安装
```
php composer.phar install
```
使用该命令后会在当前目录中创建一个 vendor 文件夹，里面包含 sdk 的依赖库和一个 autoload.php 脚本，方便用户在自己的项目中调用。
5. 通过 autoloader 脚本调用 cos-php-sdk-v5
```
require '/path/to/sdk/vendor/autoload.php';
```
现在您的项目已经可以使用COS的V5 SDK了。


#### 2、Phar方式
phar方式安装SDK的步骤如下：

1.  在[github发布页面](https://github.com/tencentyun/cos-php-sdk-v5/releases)下载相应的phar文件

2.  在代码中引入phar文件：
```
require  '/path/to/cos-sdk-v5.phar';
```

#### 3、源码方式
源码方式安装SDK的步骤如下：

1.  在[github发布页面](https://github.com/tencentyun/cos-php-sdk-v5/releases)下载相应的zip文件

2.  解压通过 autoload.php 脚本加载sdk
```
require '/path/to/sdk/vendor/autoload.php';
```


### 签名和权限

COS V5 使用了新的鉴权算法，在 V4 中您需要自己在后台计算好签名，再返回客户端使用，在 V5 中，我们强烈建议您后台接入我们的临时密钥（STS）方案。您不需要了解签名计算过程，只需要在服务器端接入 CAM，将拿到的临时密钥返回到客户端，并设置到 SDK 中，SDK 会负责管理密钥和计算签名。临时密钥在一段时间后会自动失效，而永久密钥不会泄露。您还可以按照不同的粒度来控制访问权限。具体的步骤请参考 [快速搭建移动应用直传服务](https://cloud.tencent.com/document/product/436/9068) 以及 [权限控制实例](https://cloud.tencent.com/document/product/436/30172)。

### 初始化

在 V5 中，我们的初始化接口发生了一些变化：

* 为了区分，`CosXmlServiceConfig` 代替了 `COSClientConfig`，`CosXmlService` 代替了 `COSClient`，但他们的作用相同。
* 您需要在初始化时实例化一个密钥提供者 `QCloudCredentialProvider`，用于提供一个有效的密钥，建议使用临时密钥。

v4的初始化方式如下：

```
require('cos-php-sdk-v4/include.php'); 
use Qcloud\Cos\Api;
//创建COSClientConfig对象，根据需要修改默认的配置参数
$config = array(
    'app_id' => '',
    'secret_id' => '',
    'secret_key' => '',
    'region' => 'gz',
    'timeout' => 60
);
//创建cosApi对象，实现对象存储的操作
$cosApi = new Api($config);
```

v5的初始化方式如下：

```
require '/path/to/sdk/vendor/autoload.php';
$cosClient = new Qcloud\Cos\Client(array('region' => getenv('COS_REGION'),
    'credentials'=> array(
        'secretId'    => getenv('COS_KEY'),
        'secretKey' => getenv('COS_SECRET'))));
```


### Bucket 和 Region 变化

V5 的存储桶名称发生了变化，在 V5 中，存储桶名称由两部分组成：用户自定义字符串 和 APPID，两者以中划线“-”相连。例如 `mybucket1-1250000000`，其中 `mybucket1` 为用户自定义字符串，`1250000000` 为 APPID。APPID 是腾讯云账户的账户标识之一，用于关联云资源。在用户成功申请腾讯云账户后，系统自动为用户分配一个 APPID。可通过 腾讯云控制台 【账号信息】查看 APPID。


V5 的存储桶可用区域简称发生了变化，下面列出了不同区域在 V4 和 V5 中的对应关系：

| 地域       | V5 地域简称         | V4 地域简称                         |
| -------- | ------------ | ---------------------------------------- |
| 北京一区（华北） | ap-beijing-1 | tj |
| 北京       | ap-beijing   | bj |
| 上海（华东）   | ap-shanghai  | sh |
| 广州（华南）   | ap-guangzhou | gz |
| 成都（西南）   | ap-chengdu   | cd |
| 重庆       | ap-chongqing | 无 |
| 新加坡      | ap-singapore | sgp |
| 香港       | ap-hongkong  | hk |
| 多伦多      | na-toronto   | ca |
| 法兰克福     | eu-frankfurt | ger |
| 孟买       | ap-mumbai    | 无 |
| 首尔       | ap-seoul     | 无 |
| 硅谷       | na-siliconvalley     | 无 |
| 弗吉尼亚       | na-ashburn     | 无 |
| 曼谷       | ap-bangkok     | 无 |
| 莫斯科       | eu-moscow     | 无 |

在初始化时，请将按照上述表格填写region字段

### API变化

#### 不再支持目录操作

在 V5 中，我们不再支持目录操作。

对象存储中本身是没有文件夹和目录的概念的，对象存储不会因为上传对象 project/a.txt 而创建一个 project 文件夹。为了满足用户使用习惯，对象存储在控制台、COS browser 等图形化工具中模拟了「 文件夹」或「 目录」的展示方式，具体实现是通过创建一个键值为 project/，内容为空的对象，展示方式上模拟了传统文件夹。

例如：上传对象 project/doc/a.txt ，分隔符 / 会模拟「 文件夹」的展示方式，于是可以看到控制台上出现「 文件夹」project 和 doc，其中 doc 是 project 下一级「 文件夹」，并包含了 a.txt 。

因此，如果您的应用场景只是上传文件，可以直接上传即可，不需要先创建文件夹。

如果您的使用场景里面有文件夹的概念，需要提供创建文件夹的功能，您可以上传一个路径以 '/' 结尾的 0KB 文件。这样在您调用 `GetBucket` 接口时，就可以将这样的文件当做文件夹。


#### 新增API

V5 增加了很多新的API，包括：

* 存储桶的操作，如 PutBucketRequest, GetBucketRequest, ListBucketRequest 等
* 存储桶ACL的操作，如 PutBucketACLRequest，GetBucketACLRequest 等
* 存储桶生命周期的操作，如 PutBucketLifecycleRequest, GetBucketLifecycleRequest 等

具体请参考我们的 [PHP SDK 接口文档](https://cloud.tencent.com/document/product/436/12267)。
