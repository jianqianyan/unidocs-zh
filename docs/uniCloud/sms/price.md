# 计费模式
- 短信费用为：0.036元/条。

但在实际使用中需要依赖`uniCloud`云服务。如使用uniCloud阿里云商业版，每条大约需要多花0.0000139元，几乎可以忽略不计。您也可以粗略认为每条短信的费用为0.0360139元/条。费用计算规则如下：


`短信`业务涉及费用的部分主要是云函数/云对象的使用量、调用次数、和出网流量(如：使用`uni-id-co`或自定义的云函数/云对象来发送短信)。
接下来，我们对不同资源，分别进行费用评估。

我们按照uniCloud官网列出的[按量计费](https://uniapp.dcloud.net.cn/uniCloud/price.html#aliyun-postpay)规则，可以简单得出如下公式：

`云函数/云对象费用 = 资源使用量 * 0.000110592  + 调用次数 * 0.0133 / 10000 + 出网流量 * 0.8`

其中：
- 资源使用量 = 云函数内存（单位为G） * 云函数平均单次执行时长（单位为秒） * 调用次数
- 调用次数 = 发送短信条数（一般情况下发送条数 = 调用次数，特殊情况除外）


我们假设如下数据模型：

- 云函数内存：512M，即0.5G （云函数内存默认为512M，用户可以自定义设置，最低可设置为128M。如果您仅发送短信，没有其他复杂业务，那么内存设为128M可以进一步的降低费用）
- 云函数平均单次执行时长：200毫秒，即0.2秒
- 短信业务平均每日调用次数：10000次
- 出网流量：单次请求 2 KB

按照如上公式，其`短信`业务云函数每天的费用为：

```
云函数费用（天） = 资源使用量 * 0.000110592  + 调用次数 * 0.0133 / 10000 + 出网流量 * 0.8
			  = 云函数内存（单位为G） * 云函数平均单次执行时长（单位为秒） * 调用次数 + 调用次数 * 0.0133 / 10000 + 出网流量 * 0.8
			  = 0.5G * 0.2S * 10000 * 0.000110592 + 10000 * 0.0133/10000 + 10000 * 2 * 0.8 / (1024 * 1024) 
			  = 0.110592 + 0.0133 + 0.0152587890625
			  = 0.1391507890625（元）
			  ≈ 0.139（元）
```

结论：如果你的`短信`业务平均每天发送条数为10000条，使用阿里云正式版云服务空间后，对应云函数每天大概消耗0.139元，对比之前的短信费用，平均每次调用多花0.0000139元，几乎可忽略不计。


- 计费条数计算方法：短信内容少于70个字符（每个汉字、标点、空格、字母均算一个字符）算作1条短信，短信内容多于70个字符时，每67个字符算作一条短信，并向上取整（不足67个字符的部分也算做1条）。 例： 短信内容有 100个字符时计费短信条数应为 100 / 67 ≈ 1.49 向上取整后算作2条。
- 最终按照成功回执状态为"成功"的短信条数计费，成功回执状态可在"发送记录"页面查看。

特别注意：短信成功回执最长延迟为72小时。