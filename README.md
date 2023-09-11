# 北京用电信息查询

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![Stable](https://img.shields.io/github/v/release/georgezhao2010/bj_sgcc_energy)](https://github.com/georgezhao2010/bj_sgcc_energy/releases/latest)

通过“国网北京电力”微信公众号的接口，采集你的家庭用电信息。

# 使用之前
首先关注“国网北京电力”微信公众号，打开微营业厅->个人中心->户号关联，确保你至少已关联一个北京国网电力的户号。如果没有关联，在此进行户号关联操作。此时点开微营业厅，应已经可以看到关联的用户，点击用户，可以看到该用户的用电信息。

使用任何网络抓包软件，如安卓手机的Fiddler， 苹果手机的Stream，进行抓包，过滤条件可以选择"HTTP"。抓包时在微营业厅上进行操作，查看一下用电信息。看到HTTP HEADER中有内容为“user_openid=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX”的内容，将user_openid=后的内容复制下来，如果最后是以\r\n结尾，则去掉\r\n，如果后部包含等于号"="，保留等于号。这个字符串就是openid，保存备用。

![screenshot](https://user-images.githubusercontent.com/27534713/129531245-c5190326-3258-4181-a8e9-e86598ff27bf.png)


# 安装
使用HACS以自定义存储库方式安装，或者从[Latest release](https://github.com/georgezhao2010/bj_sgcc_energy/releases/latest)下载最新的Release版本，将其中的`custom_components/bj_sgcc_engergy`放到你的Home Assistant的`custom_components/bj_sgcc_engergy`中。

# 配置

通过UI配置，在集成-北京用电信息查询中，输入openid，然后选择户号即可。

![config1](images/config1.png)

![config2](images/config2.png)

# 特性
- 支持多用户用电信息的采集。
- 支持实时用电单价实体，可用于Home Assistant 2021.8.X最新的能源模块的实时电费计算。
- 数据为定时更新，更新间隔为10分钟。
- 支持北京市居民的阶梯用电计价策略
- 支持北京市非居民的峰平谷用电计价策略(Beta)

## 传感器
包含的传感器

| entity_id形式 | 含义 | 属性 | 备注 |
| ---- | ---- | ---- | ---- |
| sensor.XXXXXXXXXX_balance | 电费余额 | last_update - 网端数据更新时间 |
| sensor.XXXXXXXXXX_current_level | 当前用电阶梯(峰平谷用户无此项) |
| sensor.XXXXXXXXXX_current_level_consume | 当前阶梯用电(峰平谷用户无此项) |
| sensor.XXXXXXXXXX_current_level_remain | 当前阶梯剩余额度(峰平谷用户无此项) |
| sensor.XXXXXXXXXX_current_pgv_type | 当前电价类别(阶梯用户无此项) | |可能的值:峰、平、谷、尖峰(?)|
| sensor.XXXXXXXXXX_current_price | 当前电价 |
| sensor.XXXXXXXXXX_year_consume | 本年度用电量 |
| sensor.XXXXXXXXXX_year_consume_bill | 本年度电费 |
| sensor.XXXXXXXXXX_history_* | 过去12个月用电 | name - 月份<br/>consume_bill - 该月电费| \*取值为1-12<br/> |
| sensor.XXXXXXXXXX_daily_* | 过去30天用电 | name - 日期<br/>bill_time - 采集时间<br/>day_consume1 - 平时用电<br/>day_consume2 - 尖时用电?<br/>day_consume3 - 峰时用电?<br/>day_consume4 - 谷时用电?| \*取值为1-30|

其中XXXXXXXXXX为北京国电用户户号

# 示例
历史数据采用[flex-table-card](https://github.com/custom-cards/flex-table-card)展示
```
type: vertical-stack
cards:
  - type: entities
    entities:
      - entity: sensor.XXXXXXXXXX_balance
      - entity: sensor.XXXXXXXXXX_current_level
      - entity: sensor.XXXXXXXXXX_current_level_consume
      - entity: sensor.XXXXXXXXXX_current_level_remain
      - entity: sensor.XXXXXXXXXX_current_price
      - entity: sensor.XXXXXXXXXX_year_consume
      - entity: sensor.XXXXXXXXXX_year_consume_bill
    title: 家1
  - type: custom:flex-table-card
    title: 过去12个月用电情况
    entities:
      include: sensor.XXXXXXXXXX_history_*
    columns:
      - name: 月份
        data: name
      - name: 用电量
        data: state
      - name: 电费
        data: consume_bill
  - type: custom:flex-table-card
    title: 过去30天用电情况
    entities:
      include: sensor.XXXXXXXXXX_daily_*
    columns:
      - name: 日期
        data: name
      - name: 用电量
        data: state
      - name: 尖时用电
        data: day_consume2
      - name: 峰时用电
        data: day_consume3
      - name: 平时用电
        data: day_consume1
      - name: 谷时用电
        data: day_consume4
```
![screenshot](https://user-images.githubusercontent.com/27534713/129530748-0f3d980b-357f-4538-b4b4-4f4f65e3df48.png)

你也可以根据需要采用自己的展示形式

# 特别鸣谢
[瀚思彼岸论坛](https://bbs.hassbian.com/)的[@crazysiri](https://bbs.hassbian.com/thread-13355-1-1.html)，直接使用了他的部分代码。

