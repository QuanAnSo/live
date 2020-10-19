# AddLiveAppRecordConfig

调用AddLiveAppRecordConfig配置APP录制，输出内容保存到OSS中。

正常情况下，开通直播服务时，您已自动授权“允许直播服务写入用户OSS”，因此直播录制写入您指定的bucket时不存在权限问题。如果该权限意外被删除，您可以重新配置：

-   **通过控制台配置**：您需要授权视频直播可将视频内容写入OSS产品的权限，授权后才能将视频存储至指定的OSS bucket中。详情参见[配置OSS](~~84932~~)。
-   **通过RAM进行权限配置**：详情参见[使用RAM配置子账号访问直播控制台](~~97633~~)。

**说明：**

-   修改配置后，新配置对修改之前的直播流不生效，必须重新推流才能生效！
-   如果指定了时间段，在该时间段内如果没有推流，自然不会录制。限定时间段的配置规则是一次性的，即当指定的时间段过去之后，该规则不会再触发。需要注意的是，**StartTime**和**EndTime**字段填的是UTC时间，请注意和本地时区的对应。
-   **AddLiveAppRecordConfig**接口中的**AppName**和**StreamName**可以填为`*`，表示所有**AppName**和所有**StreamName**（即不限制**AppName**或**StreamName**）。
-   可以通过**AddLiveAppRecordConfig**配置多条规则，规则匹配时存在优先级：
    -   同时指定**DomainName**、**AppName**（不为`*`）、**StreamName**（不为`*`）的优先级最高。
    -   同时指定**DomainName**、**AppName**（不为`*`）的优先级次之。
    -   单独指定**DomainName**，**AppName**为`*`（即只限定域名）的优先级最低。
-   如果想知道自动录制是否生效，或者希望针对每个录制文件做实时处理，可以设置录制回调，详见：[录制内容检索与管理](~~109075~~)。
-   自动录制每隔一定周期（周期时间通过RecordFormat.N.CycleDuration字段配置）会产生一个录制文件。如果在一个录制周期内，直播流发生了断流，但是在3分钟内，该直播流又推上来了，那么仍会在同一个录制文件中继续录制。这就意味着，一条直播流必须断流超过3分钟，才会生成最后一个录制文件。如果您希望修改这个默认的3分钟断流时间，可以提交工单在后台修改。

## 调试

[您可以在OpenAPI Explorer中直接运行该接口，免去您计算签名的困扰。运行成功后，OpenAPI Explorer可以自动生成SDK代码示例。](https://api.aliyun.com/#product=live&api=AddLiveAppRecordConfig&type=RPC&version=2016-11-01)

## 请求参数

|名称|类型|是否必选|示例值|描述|
|--|--|----|---|--|
|Action|String|是|AddLiveAppRecordConfig|系统规定参数，取值：**AddLiveAppRecordConfig**。 |
|AppName|String|是|testApp|直播流所属应用名称。

 支持通配符\(\*\)，代表该域名下所有的AppName。 |
|DomainName|String|是|test.com|加速域名，指播放域名。 |
|OssBucket|String|是|testBucket|OssBucket名称。 |
|OssEndpoint|String|是|oss-cn-shanghai.aliyuncs.com|OssEndpoint域名。 |
|RecordFormat.N.Format|String|否|m3u8|格式。目前支持m3u8、flv或mp4。 |
|RecordFormat.N.OssObjectPrefix|String|否|record/\{AppName\}/\{StreamName\}/\{Sequence\}\{EscapedStartTime\}\{EscapedEndTime\}|OSS存储的录制文件名，小于256 byte，支持变量匹配，包含 \{AppName\}、\{StreamName\}、\{Sequence\}、\{StartTime\}、\{EndTime\}、\{EscapedStartTime\}、\{EscapedEndTime\}。参数值必须要有\{StartTime\}或\{EscapedStartTime\}和\{EndTime\}或\{EscapedEndTime\}变量。默认支持1小时周期录制，最小周期时间15分钟，最多6小时。

 例如：record/\{AppName\}/\{StreamName\}/\{Sequence\}\_\{EscapedStartTime\}\_\{EscapedEndTime\}。

 <note\>\{StartTime\}/\{EndTime\}格式为：2006-01-02-15：04：05，\{EscapedStartTime\}/\{EscapedEndTime\}格式为：2006-01-02-15-04-05，推荐使用 Escaped 格式，避免特殊字符在URL中带来的一些问题。</note\> |
|RecordFormat.N.SliceOssObjectPrefix|String|否|record/\{AppName\}/\{StreamName\}/\{UnixTimestamp\}\_\{Sequence\}|当format格式是m3u8录制，则需要配置，表示ts切片名称。默认30秒一片，小于256byte，支持变量匹配，包含\{AppName\}、\{StreamName\}、\{UnixTimestamp\}、\{Sequence\}。

 例如：record/\{AppName\}/\{StreamName\}/\{UnixTimestamp\}\_\{Sequence\}，参数值必须包含\{UnixTimestamp\}和\{Sequence\}变量。 |
|RecordFormat.N.CycleDuration|Integer|否|1|周期录制时长。单位：秒。不填则默认为**6**小时。 |
|StreamName|String|否|teststream|流名称。 |
|StartTime|String|否|2018-04-10T09:57:21Z|录制开始时间。格式：UTC时间。

 设置的时间必须是实际推流时间\(这条流计划推流录制的时间\)开始7天之内的时间，只在流级别录制\(StreamName不为空\)有效。 |
|EndTime|String|否|2018-04-16T09:57:21Z|录制结束时间。格式：UTC时间。

 设置的时间必须大于StartTime，且与StartTime相差不应超过7天，超过7天将按照7天计算，只在流级别录制\(StreamName不为空\)有效。 |
|OnDemand|Integer|否|1|按需录制。

 -   0表示关闭。
-   1表示通过HTTP回调方式。
-   2表示解析推流参数按需录制
-   7表示默认不录制，通过RealTimeRecordCommand接口手动控制录制启停。

 **说明：** 使用1方式时候需要先通过[AddLiveRecordNotifyConfig接口](~~51831~~)设置OnDemandUrl，否则默认不录制。 |

## 返回数据

|名称|类型|示例值|描述|
|--|--|---|--|
|RequestId|String|16A96B9A-F203-4EC5-8E43-CB92E68F4CD8|请求ID |

## 示例

请求示例

```
http(s)://[live.aliyuncs.com]/?Action=AddLiveAppRecordConfig
&AppName=testApp
&DomainName=test.com
&OssBucket=testBucket
&OssEndpoint=oss-cn-shanghai.aliyuncs.com
&RecordFormat.1.Format=m3u8
&RecordFormat.1.OssObjectPrefix=record%2F%7BAppName%7D%2F%7BStreamName%7D%2F%7BSequence%7D%7BEscapedStartTime%7D%7BEscapedEndTime%7D
&RecordFormat.1.SliceOssObjectPrefix=record%2F%7BAppName%7D%2F%7BStreamName%7D%2F%7BUnixTimestamp%7D_%7BSequence%7D
&RecordFormat.2.Format=mp4
&RecordFormat.2.OssObjectPrefix=record%2F%7BAppName%7D%2F%7BStreamName%7D%2F%7BSequence%7D%7BEscapedStartTime%7D%7BEscapedEndTime%7D
&<公共请求参数>
```

正常返回示例

`XML` 格式

```
<AddLiveAppRecordConfigResponse>
	  <RequestId>16A96B9A-F203-4EC5-8E43-CB92E68F4CD8</RequestId>
</AddLiveAppRecordConfigResponse>
```

`JSON` 格式

```
{
    "RequestId":"16A96B9A-F203-4EC5-8E43-CB92E68F4CD8"
}
```

## 错误码

|HttpCode|错误码|错误信息|描述|
|--------|---|----|--|
|400|InvalidOssBucket.Malformed|Specified parameter OssBucket is not valid.|OSSBucket参数错误，请您确认该OSS BUCKET参数是否正确。|
|400|InvalidOssBucket.NotFound|The parameter OssBucket does not exist.|OSSBucket参数错误，请您确认该OSS BUCKET参数是否正确。|
|400|InvalidFormat.Malformed|Specified parameter Format is not valid.|Format参数错误，请您确认该Format参数是否正确。|
|400|InvalidCycleDuration.Malformed|Specified CycleDuration Format is not valid.|CycleDuration参数格式错误，请您确认该CycleDuration参数格式是否正确。|
|400|MissingOssObjectPrefix|OssObjectPrefix is mandatory for this action.|缺少OssObjectPrefix值，请您确认OssObjectPrefix值是否正确。|
|400|InvalidOssObjectPrefix.Malformed|Specified parameter OssObjectPrefix is not valid.|OSSObjectPrefix参数错误，请您确认该OSSObjectPrefix参数是否正确。|
|400|InvalidSliceOssObjectPrefix.Malformed|Specified parameter SliceOssObjectPrefix is not valid.|SliceOssObjectPrefix参数错误，请您确认该SliceOssObjectPrefix参数是否正确。|
|400|ConfigAlreadyExists|Config has already exist.|配置已添加。|
|400|InvalidStartTime.Malformed|Specified StartTime is malformed.|StartTime参数错误，请您确认该StartTime参数是否正确。|
|400|InvalidEndTime.Malformed|Specified EndTime is malformed.|结束时间错误，请您确认结束时间是否正确。|

访问[错误中心](https://error-center.aliyun.com/status/product/live)查看更多错误码。

