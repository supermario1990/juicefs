---
sidebar_label: 如何设置对象存储
sidebar_position: 4
slug: /how_to_setup_object_storage
---

# JuiceFS 如何设置对象存储

通过阅读 [JuiceFS 的技术架构](../introduction/architecture.md)可以了解到，JuiceFS 是一个数据与元数据分离的分布式文件系统，以对象存储作为主要的数据存储，以 Redis、PostgreSQL、MySQL 等数据库作为元数据存储。

## 存储参数

在创建 JuiceFS 文件系统时，设置数据存储一般涉及以下几个选项：

- `--storage` 指定文件系统要使用的存储类型，例如：`--storage s3`。
- `--bucket` 指定存储访问地址，例如：`--bucket https://myjuicefs.s3.us-east-2.amazonaws.com`。
- `--access-key` 和 `--secret-key` 指定访问存储时的身份认证信息。

例如，以下命令使用 Amazon S3 对象存储创建文件系统：

```shell
$ juicefs format --storage s3 \
	--bucket https://myjuicefs.s3.us-east-2.amazonaws.com \
	--access-key abcdefghijklmn \
	--secret-key nmlkjihgfedAcBdEfg \
	redis://192.168.1.6/1 \
	myjfs
```

## Access Key 和 Secret Key

一般而言，对象存储通过 `Access Key ID` 和 `Access Key Secret` 验证用户身份，对应到 JuiceFS 文件系统就是 `--access-key` 和 `--secret-key` 这两个选项（或者简称为 AK、SK）。

创建文件系统时除了使用 `--access-key` 和 `--secret-key` 两个选项显式指定，更安全的做法是通过 `ACCESS_KEY` 和 `SECRET_KEY` 环境变量传递密钥信息，例如：

```shell
$ export ACCESS_KEY=abcdefghijklmn
$ export SECRET_KEY=nmlkjihgfedAcBdEfg
$ juicefs format --storage s3 \
	--bucket https://myjuicefs.s3.us-east-2.amazonaws.com \
	redis://192.168.1.6/1 \
	myjfs
```

公有云通常允许用户创建 IAM（Identity and Access Management）角色，例如：[AWS IAM 角色](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_roles.html) 或 [阿里云 RAM 角色](https://help.aliyun.com/document_detail/93689.html)，可将角色分配给 VM 实例。如果云服务器实例已经拥有读写对象存储的权限，则无需再指定 `--access-key` 和 `--secret-key`。

## 支持的存储服务

如果你希望使用的存储类型不在列表中，欢迎提交需求 [issue](https://github.com/juicedata/juicefs/issues)。

| Name                                        | Value      |
| --------------------------------------      | ---------- |
| [Amazon S3](#amazon-s3)                     | `s3`       |
| [Google 云存储](#google-云存储)             | `gs`       |
| [Azure Blob 存储](#azure-blob-存储)         | `wasb`     |
| [Backblaze B2](#backblaze-b2)               | `b2`       |
| [IBM 云对象存储](#ibm-云对象存储)           | `ibmcos`   |
| [Scaleway](#scaleway)                       | `scw`      |
| [DigitalOcean Spaces](#digitalocean-spaces) | `space`    |
| [Wasabi](#wasabi)                           | `wasabi`   |
| [Storj DCS](#storj-dcs)                     | `s3`       |
| [Vultr 对象存储](#vultr-对象存储)           | `s3`       |
| [阿里云 OSS](#阿里云-oss)                   | `oss`      |
| [腾讯云 COS](#腾讯云-cos)                   | `cos`      |
| [华为云 OBS](#华为云-obs)                   | `obs`      |
| [百度云 BOS](#百度云-bos)                   | `bos`      |
| [金山云 KS3](#金山云-ks3)                   | `ks3`      |
| [美团云 MMS](#美团云-mms)                   | `mss`      |
| [网易云 NOS](#网易云-nos)                   | `nos`      |
| [青云 QingStor](#青云-qingstor)             | `qingstor` |
| [七牛云 Kodo](#七牛云-kodo)                 | `qiniu`    |
| [新浪云 SCS](#新浪云-scs)                   | `scs`      |
| [天翼云 OOS](#天翼云-oos)                   | `oos`      |
| [移动云 EOS](#移动云-eos)                   | `eos`      |
| [优刻得 US3](#优刻得-us3)                   | `ufile`    |
| [Ceph RADOS](#ceph-rados)                   | `ceph`     |
| [Ceph RGW](#ceph-rgw)                       | `s3`       |
| [Swift](#swift)                             | `swift`    |
| [MinIO](#minio)                             | `minio`    |
| [WebDAV](#webdav)                           | `webdav`   |
| [HDFS](#hdfs)                               | `hdfs`     |
| [Redis](#redis)                             | `redis`    |
| [TiKV](#tikv)                               | `tikv`     |
| [本地磁盘](#本地磁盘)                       | `file`     |

## Amazon S3

S3 支持  [两种风格的 endpoint URI](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/VirtualHosting.html)：`虚拟托管类型` 和 `路径类型`。

- 虚拟托管类型：`https://<bucket>.s3.<region>.amazonaws.com`
- 路径类型：`https://s3.<region>.amazonaws.com/<bucket>`

其中 `<region>` 要替换成实际的区域代码，比如：美国西部（俄勒冈）的区域代码为 `us-west-2`。[点此查看](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)所有的区域代码。

:::note 注意

AWS 中国的用户，应使用 `amazonaws.com.cn` 域名。相应的区域代码信息[点此查看](https://docs.amazonaws.cn/aws/latest/userguide/endpoints-arns.html)。

:::

JuiceFS v0.12 之前的版本仅支持虚拟托管类型，v0.12 以及之后的版本两种风格都支持。例如：

```bash
# 虚拟托管类型
$ juicefs format \
    --storage s3 \
    --bucket https://<bucket>.s3.<region>.amazonaws.com \
    ... \
    myjfs
```

```bash
# 路径类型
$ juicefs format \
    --storage s3 \
    --bucket https://s3.<region>.amazonaws.com/<bucket> \
    ... \
    myjfs
```

:::tip 提示

所有 S3 兼容的对象存储服务都可以参照 Amazon S3 存储类型进行设定。

:::

## Google 云存储

由于 Google 云存储没有 `Access key` 和 `Secret key`，因此在创建文件系统时可以忽略 `--access-key` 和 `--secret-key` 选项。请查阅 Google Cloud 的文档了解 [身份验证](https://cloud.google.com/docs/authentication) 和 [身份及访问管理（IAM）](https://cloud.google.com/iam/docs/overview) 相关内容。

一般来说，无需额外配置，在使用 Google 云服务器时默认已经拥有云存储的访问权限。

另外，由于 bucket 名称是 [全局唯一](https://cloud.google.com/storage/docs/naming-buckets#considerations) 的，创建文件系统时，`--bucket` 选项中只需指定 bucket 名称即可。例如：

```bash
$ juicefs format \
    --storage gs \
    --bucket gs://<bucket> \
    ... \
    myjfs
```

## Azure Blob 存储

使用 Azure Blob 作为 JuiceFS 的数据存储，除了使用 `--access-key` 和 `--secret-key` 选项之外，你也可以使用 [连接字符串](https://docs.microsoft.com/zh-cn/azure/storage/common/storage-configure-connection-string) 并通过 `AZURE_STORAGE_CONNECTION_STRING` 环境变量进行设定。例如：

```bash
# Use connection string
$ export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=XXX;AccountKey=XXX;EndpointSuffix=core.windows.net"
$ juicefs format \
    --storage wasb \
    --bucket https://<container> \
    ... \
    myjfs
```

:::note 注意

Azure China 用户，`EndpointSuffix` 值为 `core.chinacloudapi.cn`。

:::

## Backblaze B2

使用 Backblaze B2 作为 JuiceFS 的数据存储，需要先创建  [application key](https://www.backblaze.com/b2/docs/application_keys.html)，**Application Key ID** 和 **Application Key** 分别对应 `Access key` 和 `Secret key`。

Backblaze B2 支持两种访问接口：B2 原生 API 和 S3 兼容 API。

### B2 原生 API

存储类型应设置为 `b2`，`--bucket` 只需设置 bucket 名称。例如：

```bash
$ juicefs format \
    --storage b2 \
    --bucket <bucket> \
    --access-key <application-key-ID> \
    --secret-key <application-key> \
    ... \
    myjfs
```

### S3 兼容 API

存储类型应设置为 `s3`，`--bucket` 应指定完整的 bucket 地址。例如：

```bash
$ juicefs format \
    --storage s3 \
    --bucket https://s3.eu-central-003.backblazeb2.com/<bucket> \
    --access-key <application-key-ID> \
    --secret-key <application-key> \
    ... \
    myjfs
```

## IBM 云对象存储

使用 IBM 云对象存储创建 JuiceFS 文件系统，你首先需要创建 [API key](https://cloud.ibm.com/docs/account?topic=account-manapikey) 和 [instance ID](https://cloud.ibm.com/docs/key-protect?topic=key-protect-retrieve-instance-ID)。**API key** 和 **instance ID** 分别对应 `Access key` 和 `Secret key`。

IBM 云对象存储为每一个区域提供了 `公网` 和 `内网` 两种 [endpoint 地址](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-endpoints)，你可以根据实际需要选用。例如：

```bash
$ juicefs format \
    --storage ibmcos \
    --bucket https://<bucket>.<endpoint> \
    --access-key <API-key> \
    --secret-key <instance-ID> \
    ... \
    myjfs
```

## Scaleway

使用 Scaleway 对象存储作为 JuiceFS 数据存储，请先 [查看文档](https://www.scaleway.com/en/docs/generate-api-keys) 了解如何创建  `Access key` 和 `Secret key`。

`--bucket` 选项的设置格式为 `https://<bucket>.s3.<region>.scw.cloud`，请将其中的  `<region>`  替换成实际的区域代码，例如：荷兰阿姆斯特丹的区域代码是 `nl-ams`。[点此查看](https://www.scaleway.com/en/docs/object-storage-feature/#-Core-Concepts) 所有可用的区域代码。

```bash
$ juicefs format \
    --storage scw \
    --bucket https://<bucket>.s3.<region>.scw.cloud \
    ... \
    myjfs
```

## DigitalOcean Spaces

使用 DigitalOcean Spaces 作为 JuiceFS 数据存储，请先 [查看文档](https://www.digitalocean.com/community/tutorials/how-to-create-a-digitalocean-space-and-api-key) 了解如何创建  `Access key` 和 `Secret key`。

`--bucket` 选项的设置格式为 `https://<space-name>.<region>.digitaloceanspaces.com`，请将其中的  `<region>`  替换成实际的区域代码，例如：`nyc3`。[点此查看](https://www.digitalocean.com/docs/spaces/#regional-availability) 所有可用的区域代码。

```bash
$ juicefs format \
    --storage space \
    --bucket https://<space-name>.<region>.digitaloceanspaces.com \
    ... \
    myjfs
```

## Wasabi

使用 Wasabi 作为 JuiceFS 数据存储，请先 [查看文档](https://wasabi-support.zendesk.com/hc/en-us/articles/360019677192-Creating-a-Root-Access-Key-and-Secret-Key) 了解如何创建  `Access key` 和 `Secret key`。

`--bucket` 选项的设置格式为 `https://<bucket>.s3.<region>.wasabisys.com`，请将其中的  `<region>`  替换成实际的区域代码，例如：US East 1 (N. Virginia) 的区域代码为 `us-east-1`。[点此查看](https://wasabi-support.zendesk.com/hc/en-us/articles/360.15.26031-What-are-the-service-URLs-for-Wasabi-s-different-regions-) 所有可用的区域代码。

```bash
$ juicefs format \
    --storage wasabi \
    --bucket https://<bucket>.s3.<region>.wasabisys.com \
    ... \
    myjfs
```

:::note 注意

Tokyo (ap-northeast-1) 区域的用户，查看 [这篇文档](https://wasabi-support.zendesk.com/hc/en-us/articles/360039372392-How-do-I-access-the-Wasabi-Tokyo-ap-northeast-1-storage-region-) 了解 endpoint URI 的设置方法。

:::

## Storj DCS

使用 Storj DCS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://docs.storj.io/api-reference/s3-compatible-gateway) 了解如何创建 `Access key` 和 `Secret key`。

Storj DCS 兼容 AWS S3，存储类型使用 `s3` ，`--bucket` 格式为 `https://gateway.<region>.storjshare.io/<bucket>`。`<region>` 为存储区域，目前 DCS 有三个可用存储区域：us1、ap1 和 eu1。

```shell
$ juicefs format \
	--storage s3 \
	--bucket https://gateway.<region>.storjshare.io/<bucket> \
	--access-key <your-access-key> \
	--secret-key <your-sceret-key> \
	... \
	my-jfs
```

## Vultr 对象存储

Vultr 的对象存储兼容 S3 API，存储类型使用 `s3`，`--bucket` 格式为 `https://<bucket>.<region>.vultrobjects.com/`。例如：

```shell
$ juicefs format \
	--storage s3 \
	--bucket https://<bucket>.ewr1.vultrobjects.com/ \
	--access-key <your-access-key> \
	--secret-key <your-sceret-key> \
	redis://localhost/1 my-jfs
```

访问对象存储的 API 密钥可以在 [管理控制台](https://my.vultr.com/objectstorage/) 中找到。


## 阿里云 OSS

使用阿里云 OSS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://help.aliyun.com/document_detail/38738.html) 了解如何创建 `Access key` 和 `Secret key`。如果你已经创建了 [RAM 角色](https://help.aliyun.com/document_detail/93689.html) 并指派给了云服务器实例，则在创建文件系统时可以忽略 `--access-key` 和 `--secret-key` 选项。

阿里云也支持使用 [Security Token Service (STS)](https://help.aliyun.com/document_detail/100624.html) 作为 OSS 的临时访问身份验证。如果你要使用 STS，请设置  `ALICLOUD_ACCESS_KEY_ID`、`ALICLOUD_ACCESS_KEY_SECRET` 和 `SECURITY_TOKEN ` 环境变量，不要设置 `--access-key` and `--secret-key` 选项。例如：

```bash
$ export ALICLOUD_ACCESS_KEY_ID=XXX
$ export ALICLOUD_ACCESS_KEY_SECRET=XXX
$ export SECURITY_TOKEN=XXX
$ juicefs format \
    --storage oss \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

阿里云 OSS 为每个区域都提供了 `公网` 和 `内网` [endpoint 链接](https://help.aliyun.com/document_detail/31834.html)，你可以根据实际的场景选用。

如果你是在阿里云的服务器上创建文件系统，可以在 `--bucket` 选项中直接指定 bucket 名称。例如：

```bash
$ juicefs format \
    --storage oss \
    --bucket <bucket> \
    ... \
    myjfs
```

## 腾讯云 COS

使用腾讯云 COS 作为 JuiceFS 数据存储，Bucket 名称格式为 `<bucket>-<APPID>`，即需要在 bucket 名称后面指定 `APPID`，[点此查看](https://cloud.tencent.com/document/product/436/13312) 如何获取  `APPID` 。

`--bucket` 选项的完整格式为 `https://<bucket>-<APPID>.cos.<region>.myqcloud.com`，请将 `<region>` 替换成你实际使用的存储区域，例如：上海的区域代码为 `ap-shanghai`。[点此查看](https://cloud.tencent.com/document/product/436/6224) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage cos \
    --bucket https://<bucket>-<APPID>.cos.<region>.myqcloud.com \
    ... \
    myjfs
```

如果你是在腾讯云的服务器上创建文件系统，可以在 `--bucket` 选项中直接指定 bucket 名称。 例如：

```bash
# Running within Tencent Cloud
$ juicefs format \
    --storage cos \
    --bucket <bucket>-<APPID> \
    ... \
    myjfs
```

## 华为云 OBS

使用华为云 OBS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://support.huaweicloud.com/usermanual-ca/zh-cn_topic_0046606340.html) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.obs.<region>.myhuaweicloud.com`，请将 `<region>` 替换成你实际使用的存储区域，例如：北京一的区域代码为 `cn-north-1`。[点此查看](https://developer.huaweicloud.com/endpoint?OBS) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage obs \
    --bucket https://<bucket>.obs.<region>.myhuaweicloud.com \
    ... \
    myjfs
```

如果是你在华为云的服务器上创建文件系统，可以在 `--bucket` 直接指定 bucket 名称。例如：

```bash
# Running within Huawei Cloud
$ juicefs format \
    --storage obs \
    --bucket <bucket> \
    ... \
    myjfs
```

## 百度 BOS

使用百度云 BOS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://cloud.baidu.com/doc/Reference/s/9jwvz2egb) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.<region>.bcebos.com`，请将 `<region>` 替换成你实际使用的存储区域，例如：北京的区域代码为 `bj`。[点此查看](https://cloud.baidu.com/doc/BOS/s/Ck1rk80hn#%E8%AE%BF%E9%97%AE%E5%9F%9F%E5%90%8D%EF%BC%88endpoint%EF%BC%89) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage bos \
    --bucket https://<bucket>.<region>.bcebos.com \
    ... \
    myjfs
```

如果你是在百度云的服务器上创建文件系统，可以在 `--bucket` 直接指定 bucket 名称。例如：

```bash
# Running within Baidu Cloud
$ juicefs format \
    --storage bos \
    --bucket <bucket> \
    ... \
    myjfs
```

## 金山云 KS3

使用金山云 KS3 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://docs.ksyun.com/documents/1386) 了解如何创建 `Access key` 和 `Secret key`。

金山云 KS3 为每个区域都提供了 `公网` 和 `内网` [endpoint 链接](https://docs.ksyun.com/documents/6761)，你可以根据实际的场景选用。

```bash
$ juicefs format \
    --storage ks3 \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

## 美团云 MMS

使用美团云 MMS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://www.mtyun.com/doc/api/mss/mss/fang-wen-kong-zhi) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.<endpoint>`，请将 `<endpoint>` 替换成你实际地址，例如：`mtmss.com`。[点此查看](https://www.mtyun.com/doc/products/storage/mss/index#%E5%8F%AF%E7%94%A8%E5%8C%BA%E5%9F%9F) 所有可用的 endpoint 地址。例如：

```bash
$ juicefs format \
    --storage mss \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

## 网易云 NOS

使用网易云 NOS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://www.163yun.com/help/documents/55485278220111872) 了解如何创建 `Access key` 和 `Secret key`。

网易云 NOS 为每个区域都提供了 `公网` 和 `内网` [endpoint 链接](https://www.163yun.com/help/documents/67078583131230208)，你可以根据实际的场景选用。例如：

```bash
$ juicefs format \
    --storage nos \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

## 青云 QingStor

使用青云 QingStor 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://docsv3.qingcloud.com/storage/object-storage/api/practices/signature/#%E8%8E%B7%E5%8F%96-access-key) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.<region>.qingstor.com`，请将 `<region>` 替换成你实际使用的存储区域，例如：北京 3-A 的区域代码为 `pek3a`。[点此查看](https://docs.qingcloud.com/qingstor/#%E5%8C%BA%E5%9F%9F%E5%8F%8A%E8%AE%BF%E9%97%AE%E5%9F%9F%E5%90%8D) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage qingstor \
    --bucket https://<bucket>.<region>.qingstor.com \
    ... \
    myjfs
```

## 七牛云 Kodo

使用七牛云 Kodo 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://developer.qiniu.com/af/kb/1479/how-to-access-or-locate-the-access-key-and-secret-key) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.s3-<region>.qiniucs.com`，请将 `<region>` 替换成你实际使用的存储区域，例如：中国东部的区域代码为 `cn-east-1`。[点此查看](https://developer.qiniu.com/kodo/4088/s3-access-domainname) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage qiniu \
    --bucket https://<bucket>.s3-<region>.qiniucs.com \
    ... \
    myjfs
```

## 新浪云 SCS

使用新浪云 SCS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://scs.sinacloud.com/doc/scs/guide/quick_start#accesskey) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项格式为 `https://<bucket>.stor.sinaapp.com`。例如：

```bash
$ juicefs format \
    --storage scs \
    --bucket https://<bucket>.stor.sinaapp.com \
    ... \
    myjfs
```

## 天翼云 OOS

使用天翼云 OOS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://www.ctyun.cn/help2/10000101/10473683) 了解如何创建 `Access key` 和 `Secret key`。

`--bucket` 选项的格式为 `https://<bucket>.oss-<region>.ctyunapi.cn`，请将 `<region>` 替换成你实际使用的存储区域，例如：成都的区域代码为 `sccd`。[点此查看](https://www.ctyun.cn/help2/10000101/10474062) 所有可用的区域代码。例如：

```bash
$ juicefs format \
    --storage oos \
    --bucket https://<bucket>.oss-<region>.ctyunapi.cn \
    ... \
    myjfs
```

## 移动云 EOS

使用移动云 EOS 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://ecloud.10086.cn/op-help-center/doc/article/24501) 了解如何创建 `Access key` 和 `Secret key`。

移动云 EOS 为每个区域都提供了 `公网` 和 `内网` [endpoint 链接](https://ecloud.10086.cn/op-help-center/doc/article/40956)，你可以根据实际的场景选用。例如：

```bash
$ juicefs format \
    --storage eos \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

## 优刻得 US3

使用优刻得 US3 作为 JuiceFS 数据存储，请先参照 [这篇文档](https://docs.ucloud.cn/uai-censor/access/key) 了解如何创建 `Access key` 和 `Secret key`。

优刻得 US3（原名 UFile） 为每个区域都提供了 `公网` 和 `内网` [endpoint 链接](https://docs.ucloud.cn/ufile/introduction/region)，你可以根据实际的场景选用。例如：

```bash
$ juicefs format \
    --storage ufile \
    --bucket https://<bucket>.<endpoint> \
    ... \
    myjfs
```

## Ceph RADOS

:::note 注意
JuiceFS 支持的 Ceph 最低版本是 Luminous（v12.2.*），请确认你的 Ceph 版本是否符合要求。
:::

[Ceph 存储集群](https://docs.ceph.com/en/latest/rados) 具有消息传递层协议，该协议使客户端能够与 Ceph Monitor 和 Ceph OSD 守护程序进行交互。[`librados`](https://docs.ceph.com/en/latest/rados/api/librados-intro) API 使您可以与这两种类型的守护程序进行交互：

- [Ceph Monitor](https://docs.ceph.com/en/latest/rados/configuration/common/#monitors) 维护群集映射的主副本
- [Ceph OSD Daemon (OSD)](https://docs.ceph.com/en/latest/rados/configuration/common/#osds) 将数据作为对象存储在存储节点上

JuiceFS 支持使用基于 `librados` 的本地 Ceph API。您需要分别安装 `librados` 库并重新编译 `juicefs` 二进制文件。

首先安装 `librados`：

:::note 注意
建议使用匹配你的 Ceph 版本的 `librados`，例如 Ceph 版本是 Octopus（v15.2.\*），那么 `librados` 也建议使用 v15.2.\* 版本。某些 Linux 发行版（如 CentOS 7）自带的 `librados` 版本可能较低，如果编译 JuiceFS 失败可以尝试下载更高版本的安装包。
:::

```bash
# Debian based system
$ sudo apt-get install librados-dev

# RPM based system
$ sudo yum install librados2-devel
```

然后为 Ceph 编译 JuiceFS（要求 Go 1.16+ 和 GCC 5.4+）：

```bash
$ make juicefs.ceph
```

[存储池](https://docs.ceph.com/zh_CN/latest/rados/operations/pools) 是用于存储对象的逻辑分区，您可能需要首先创建一个存储池。 `--access-key` 选项的值是 Ceph 集群名称，默认集群名称是 `ceph`。` --secret-key` 选项的值是 [Ceph 客户端用户名](https://docs.ceph.com/en/latest/rados/operations/user-management)，默认用户名是 `client.admin`。

为了连接到 Ceph Monitor，`librados` 将通过搜索默认位置读取 Ceph 的配置文件，并使用找到的第一个。 这些位置是：

- `CEPH_CONF` 环境变量
- `/etc/ceph/ceph.conf`
- `~/.ceph/config`
- 在当前工作目录中的 `ceph.conf`

例如：

```bash
$ juicefs.ceph format \
    --storage ceph \
    --bucket ceph://<pool-name> \
    --access-key <cluster-name> \
    --secret-key <user-name> \
    ... \
    myjfs
```

## Ceph RGW

[Ceph Object Gateway](https://ceph.io/ceph-storage/object-storage) 是在 `librados` 之上构建的对象存储接口，旨在为应用程序提供访问 Ceph 存储集群的 RESTful 网关。Ceph 对象网关支持 S3 兼容的接口，因此我们可以将 `--storage` 设置为 `s3`。

`--bucket` 选项的格式为 `http://<bucket>.<endpoint>`（虚拟托管类型），例如：

```bash
$ juicefs format \
    --storage s3 \
    --bucket http://<bucket>.<endpoint> \
    ... \
    myjfs
```

## Swift

[OpenStack Swift](https://github.com/openstack/swift) 是一种分布式对象存储系统，旨在从一台计算机扩展到数千台服务器。Swift 已针对多租户和高并发进行了优化。Swift 广泛适用于备份、Web 和移动内容的理想选择，可以无限量存储任何非结构化数据。

`--bucket` 选项格式为 `http://<container>.<endpoint>`，`container` 用来设定对象的命名空间。

**当前，JuiceFS 仅支持  [Swift V1 authentication](https://www.swiftstack.com/docs/cookbooks/swift_usage/auth.html)。**

`--access-key` 选项的值是用户名，`--secret-key` 选项的值是密码。例如：

```bash
$ juicefs format \
    --storage swift \
    --bucket http://<container>.<endpoint> \
    --access-key <username> \
    --secret-key <password> \
    ... \
    myjfs
```

## MinIO

[MinIO](https://min.io) 是开源的轻量级对象存储，兼容 Amazon S3 API。

使用 Docker 可以很容易地在本地运行一个 MinIO 对象存储实例。例如，以下命令通过 `--console-address ":9900"` 为控制台设置并映射了 `9900` 端口，还将 MinIO 对象存储的数据路径映射到了当前目录下的 `minio-data` 文件夹中，你可以按需修改这些参数：

```shell
$ sudo docker run -d --name minio \
    -p 9000:9000 \
    -p 9900:9900 \
    -e "MINIO_ROOT_USER=minioadmin" \
    -e "MINIO_ROOT_PASSWORD=minioadmin" \
    -v $PWD/minio-data:/data \
    --restart unless-stopped \
    minio/minio server /data --console-address ":9900"
```

容器创建成功以后使用以下地址访问：

- **MinIO 管理界面**：[http://127.0.0.1:9900](http://127.0.0.1:9900/)
- **MinIO API**：[http://127.0.0.1:9000](http://127.0.0.1:9000/)

对象存储初始的 Access Key 和 Secret Key 均为 `minioadmin`。

使用 MinIO 作为 JuiceFS 的数据存储，`--storage` 选项设置为 `minio`。

```bash
$ juicefs format \
    --storage minio \
    --bucket http://127.0.0.1:9000/<bucket> \
    --access-key minioadmin \
    --secret-key minioadmin \
    ... \
    myjfs
```

:::note 注意

当前，JuiceFS 仅支持路径风格的 MinIO URI 地址，例如：`http://127.0.0.1:9000/myjfs`。

:::

## WebDAV

[WebDAV](https://en.wikipedia.org/wiki/WebDAV) 是 HTTP 的扩展协议，有利于用户间协同编辑和管理存储在万维网服务器的文档。JuiceFS 0.15+ 支持使用 WebDAV 协议的存储系统作为后端数据存储。

你需要将 `--storage` 设置为 `webdav`，并通过 `--bucket` 来指定访问 WebDAV 的地址。如果存储系统启用了用户验证，用户名和密码可以通过 `--access-key` 和 `--secret-key` 来指定，例如：

```bash
$ juicefs format \
    --storage webdav \
    --bucket http://<endpoint>/ \
    --access-key <username> \
    --secret-key <password> \
    ... \
    myjfs
```

## HDFS

Hadoop 的文件系统 [HDFS](https://hadoop.apache.org) 也可以作为对象存储供 JuiceFS 使用。

当使用 HDFS 作为 JuiceFS 数据存储，`--access-key` 的值设置为用户名，默认的超级用户通常是 `hdfs`。例如：

```bash
$ juicefs format \
    --storage hdfs \
    --bucket namenode1:8020 \
    --access-key hdfs \
    ... \
    myjfs
```

如果在创建文件系统时不指定 `--access-key`，JuiceFS 会使用执行 `juicefs mount` 命令的用户身份或通过 Hadoop SDK 访问 HDFS 的用户身份。如果该用户没有 HDFS 的读写权限，则程序会失败挂起，发生 IO 错误。

JuiceFS 会尝试基于 `$HADOOP_CONF_DIR` 或 `$HADOOP_HOME` 为 HDFS 客户端加载配置。如果 `--bucket` 选项留空，将使用在 Hadoop 配置中找到的默认 HDFS。

对于 HA 群集，可以像下面这样一起指定 NameNodes 的地址：`--bucket=namenode1:port,namenode2:port`。

## Redis

Redis 既可以作为 JuiceFS 的元数据存储，也可以作为数据存储，但当使用 Redis 作为数据存储时，建议不要存储大规模数据。

`--bucket` 选项格式为 `redis://<host>:<port>/<db>`。`--access-key` 选项的值是用户名，`--secret-key` 选项的值是密码。例如：

```bash
$ juicefs format \
    --storage redis \
    --bucket redis://<host>:<port>/<db> \
    --access-key <username> \
    --secret-key <password> \
    ... \
    myjfs
```

## TiKV

[TiKV](https://tikv.org) 是一个高度可扩展、低延迟且易于使用的键值数据库。它提供原始和符合 ACID 的事务键值 API。

TiKV 既可以用作 JuiceFS 的元数据存储，也可以用于 JuiceFS 的数据存储。

`--bucket` 选项格式类似 `<host>:<port>,<host>:<port>,<host>:<port>`，其中 `<host>` 是 Placement Driver（PD）的地址。`--access-key` 和 `--secret-key` 选项没有作用，可以省略。例如：

```bash
$ juicefs format \
    --storage tikv \
    --bucket "<host>:<port>,<host>:<port>,<host>:<port>" \
    ... \
    myjfs
```

## 本地磁盘

在创建 JuiceFS 文件系统时，如果没有指定任何存储类型，会默认使用本地磁盘作为数据存储，root 用户默认存储路径为 `/var/jfs`，普通用户默认存储路径为 `~/.juicefs/local`。

例如，以下命令使用本地的 Redis 数据库和本地磁盘创建了一个名为 `myfs` 的文件系统：

```
$ juicefs format redis://localhost:6379/1 myjfs
```

本地存储通常仅用于了解和体验 JuiceFS 的基本功能，创建的 JuiceFS 存储无法被网络内的其他客户端挂载，只能单机使用。
