---
title: "[现学现卖]esp32入门流程推荐与踩坑指南"
date: 2024-09-19 21:05:48
tags:
---
<!-- -->
## 引言

不知道怎么开篇 那就先展示一下过年时玩的esp8266吧
![2024.2.15](/picture/esp32/1.png)
众所周知的wifikiller（可惜不能用来打校园网:
![2024.2.15](/picture/esp32/2.png)   

赶上中秋假byr招新项目学了点记了点笔记
最后总结一下留下个资源推荐和踩坑指南收尾吧
<!-- 写引言真费劲 不是想写纯干活吗-->

## 关于硬件

[从零教你做开发板 — 应该选哪个ESP32S3？](https://www.bilibili.com/video/BV1Qe411F7LM/?share_source=copy_web&vd_source=5cdb672a85240041a0304cff7a81bfba)  顺便帮没接触的打个基础并引入
然后再在oshwhub找个开源项目分析一下抄一抄硬件任务就完成了（雾）
（这是个伪硬件入门 不过没接触过连选芯片都迷茫）

## 初入esp32    
最开始肯定是整个IDE下来 但...
什么是ESP-IDF 里面包括什么

![2024.2.15](/picture/esp32/3.png)  

<!-- 这些能查得到的真的要重复写一遍吗 -->
接下来是下载和跑个helloworld，首要肯定是搞来官方文档
https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/get-started/index.html ——从官方下载espidf（令人烦躁的是英文的）
找到一个不错的东西[【立创·ESP32S3R8N8】IDF入门手册](https://lceda001.feishu.cn/wiki/GOIlwwfbIi1SC3k8594cDeFVn8g) 嘉立创写了一份很不错的文档，下载ide这块可以完全跟着走（flash大小按自己的模块配）
![2024.2.15](/picture/esp32/4.png)   
    在搜索栏点f1可以打开一个命令面板打开示例
![2024.2.15](/picture/esp32/5.png)
    接下来就可以愉快得写工程（bug）了
    第一难：玄学bug —— 有的时候再烧一下就可以了
![2024.2.15](/picture/esp32/6.png)    
<!-- 电赛就写在本篇，esp32的感悟可以写在智能车后记；写了1h 才开始测示例 -->    

## esp32 BLE
第一个任务是实现蓝牙通信
找到官方例程 examples/bluetooth/bluedroid/ble/gatt_serve （gatt（Generic Attribute Profile，通用属性配置文件） 就不用理这词；GATT Server 简单来说是 被动的一方，等待GATT Client发起连接请求）
第二难：build.ninja... 如果同一份代码之前烧过其他的芯片 —— 需要clear一下（遇事不决... 应该是flash或配置文件什么的问题） 
![2024.2.15](/picture/esp32/7.png)  
第三难：bt.h等报错 —— menuconfig点一下蓝牙 or 点了忘保存了
![2024.2.15](/picture/esp32/8.png)  
成功了 有点激动 然后又合并上espnow再处理完爆内存后提交！然后被打回。。好吧 把蓝牙代码研究透吧
![2024.2.15](/picture/esp32/9.png)  
<!-- 唉 这时候已经9/14 过了两天了 虽然13号摆了大半天-->
<!-- 进入 软件 four 攻坚时刻-->
<!-- 标黄时间：2h了 但我感觉不好压缩时间-判断筛选（语录、图片）、补充验证（知识点）、拓展（知识点片面或未完）-->
<!-- 标黄状态与环境：环境可高可低（如果知识密度比较大建议分到高状态高环境）状态尽量高；经过测验，还是别吵-->
<!-- 有一些忏悔比如
- 9/17 0点半 ！点醒我了 —— 第二阶段 之前和o1一起整的那些活 都意义不大了；首先任务明确：就搞透esp32的蓝牙和espnow；GPT的弊端与你的选择；没有办法偷懒；而且要么你是真菜 要么你是没有主动迅速进入状态 攻克工程的能力 —— 这才是工科能力！（你别工科理科能力都没有）
-->
<!-- one末 yqf；four前后 打回与自闭-->

【【ESP32教程】第二章: 低功耗蓝牙BLE相关概念及用法】 https://www.bilibili.com/video/BV1ad4y1d7AM/?share_source=copy_web&vd_source=5cdb672a85240041a0304cff7a81bfba 挺好 广播数据包 蓝牙广播数据结构体 广播原理 扫描响应 等等 
<!-- 该不该写呢 我花了那么长时间 还想吐槽 文档太长了 英文太多了-->    
【蓝牙其实很慢！只有几兆！虽然频率2.4G！蓝牙的工作原理！】 https://www.bilibili.com/video/BV1Sm421G7Rp/?share_source=copy_web&vd_source=5cdb672a85240041a0304cff7a81bfba 爱上半导体的 简略一点
![2024.2.15](/picture/esp32/10.png) 
![2024.2.15](/picture/esp32/11.png) 
<!-- 为什么 为什么一个蓝牙就有800行 —— 我tm根本看不了啊 —— 慢慢看吧 虽说只有一个蓝牙 但我给你接下来一下午和傍晚的时间一点点攻破；找gpt吧 -->   
<!-- 蓝牙基础结束 开始刨gpt —— （笔记系统）这些 日常 真的有必要记吗--> 
<!-- 到这一块总结 思考量就增加了 必须双高-->   
想要实现个什么功能呢：
- 在我手机端输入任何东西时 esp32 都会输出字符串 “Halfapear收到信息啦”；
- 在设备连接后每隔 5 秒发送一次数据 发送 嘟噜噜 连接成功×N N是计的数；
梳理和提取一下代码，800行还是有点难绷（是的 确实有官方文档介绍函数 不过在中文文档里还掺着英文。。晚点我去pr一下教程）
首先 搜索蓝牙设备时看到的名字 来源

' #define TEST_DEVICE_NAME "ESP_GATTS_DEMO" '

ps：为了观感 晚点把原代码整个粘贴一下

主要函数：
<!-- 而且我必须保证我讲明白了-->
- **gatts_profile_a_event_handler**：处理 Profile A 的事件，例如设备连接、数据读写、通知启用等。

- **gatts_profile_b_event_handler**：处理 Profile B 的事件，和 Profile A 类似。
<!-- 下面的我好像没看懂-->
- **gap_event_handler**：处理通用的蓝牙事件，比如广播（广告）数据的设置和启动等。

- **gatts_event_handler**：分配事件给对应的 Profile（A 或 B），决定由哪个 Profile 处理蓝牙事件。

- **example_write_event_env**：处理设备接收到的写入请求（当手机发送数据到 ESP32 时调用）。

- **example_exec_write_event_env**：完成写入操作，通常用于处理长数据写入。

- **app_main**：主函数，初始化蓝牙相关的设置和功能，并启动服务。

    看来我们需要重点关注gatts_profile_a_event_handler 函数（负责处理 Profile A 的所有 GATT（通用属性协议）事件）

    在 例程 中，事件 ESP_GATTS_WRITE_EVT 里使用了esp_ble_gatts_send_indicate 里面包括了发生的数据

```c
uint8_t notify_data[15];
for (int i = 0; i < sizeof(notify_data); ++i) {
    notify_data[i] = i % 0xff;
}
```

    esp_ble_gatts_send_indicate里面的内容就体现了刚刚学的蓝牙内容

```c
esp_ble_gatts_send_indicate(gatts_if,
                            param->write.conn_id,
                            gl_profile_tab[PROFILE_A_APP_ID].char_handle,
                            sizeof(notify_data),
                            notify_data,
                            false);
```

<!-- 写这真的有点累啊-->
<!-- 9/20 从22:25开始 中间mqs来交流交流然后放松了一下 然后写到了00:45 才把ble写完 又写了一个多小时吧-->
<!-- em 还是写的不行 经过了一些修改-->
<!-- 我必须修改到新人也看得下去的地步；我必须考察到没有错误(这是很痛苦且费时的)-->
    
在gatts_profile_a_event_handler中 除了ESP_GATTS_WRITE_EVT 还有很多的EVT（即event） 这些函数可以实现发生什么事件然后干什么事；比如在发生客户端写入时就会输出一堆ff
<!-- 对吗 -->
<!-- 这里four ble 原文的混乱没法直接用就表明：我到现在都没有完全理清楚 -->
    ESP_GATTS_REG_EVT 当应用程序成功注册到 GATT 服务器时触发
    ESP_GATTS_CREATE_EVT 当服务创建成功时触发
    ESP_GATTS_ADD_CHAR_EVT 当特征添加成功时触发
    ESP_GATTS_CONNECT_EVT 当客户端（如手机）连接到服务器时触发
    ESP_GATTS_DISCONNECT_EVT 当客户端断开连接时触发
    ESP_GATTS_READ_EVT 当客户端请求读取特征值时触发
    ESP_GATTS_WRITE_EVT 当客户端写入特征或描述符时触发
    ESP_GATTS_EXEC_WRITE_EVT 当执行长写操作时触发
    ESP_GATTS_MTU_EVT 当 MTU（最大传输单元）大小发生变化时触发

<!-- 跳过CCCD 因为不知道到底犯了什么错 —— 所以从这点表明最突出-你根本是依靠gpt才出的  -->

关于客户端 我用的是手机软件 edebug（e调试）
[edebug下载](https://example.com/path/to/file)
<!-- 把链接放在资源站 -->    
![2024.2.15](/picture/esp32/11.png)  
<!-- 这里缺些图 因为当时没有截 现在重新再开会很麻烦 -->     
    - Generic Access Profile（GAP）：用于定义 BLE 设备的基本属性，如设备名称、可连接性等。UUID 为 0x1800
    - Generic Attribute Profile（GATT）：定义了如何组织和交换数据。UUID 为 0x1801。
    - Unknown Service：这是自定义的服务，因为调试软件无法识别定义的 UUID，所以显示为 "Unknown Service" 两个分别是profile_a和b
<!-- 我怀疑还会有些不妥的地方 -->     

函数与配置都讲完了 可以实现想要的效果了
达成目的之前还是会出一些问题
第四难：字面意思 —— 启用手机通知（不同软件开启地方不一样）

I (296092) GATTS_DEMO: Notifications disabled
I (296132) GATTS_DEMO: Timer callback triggered
I (296132) GATTS_DEMO: Notifications not enabled by client

第五难：通过 BLE 发送的特征（attribute）的值超过了 20 字节 —— 每个中文字符占用 3 个字节（所以有点懒 直接中文改英文了）
I (473612) GATTS_DEMO: Timer callback triggered
W (473612) BT_GATT: attribute value too long, to be truncated to 20

最终 我在例程的基础上修改了以下地方，也算是整明白了一点蓝牙

！需要 代码和视频demo
     
## esp32 espnow
紧接着试一试espnow
初步构想：espnow的用途也就是智能家居嘛，能不能收到什么消息然后转发到第二块esp32 比如像按键遥控点灯？ 
espnow的原理比起蓝牙简单很多，一种基于 Wi-Fi 协议的简化通信协议，特点是低功耗
<!-- 你当时在干什么！代码也没看 wifi也不了解 -->    
但官方example仍然有400多行 包含着从Wi-Fi 初始化到ESPNOW 初始化再到开始广播的完整配置流程         
浅聊几个：谁广播谁监听 如何规定的？在 example_espnow_task() 函数中，设备在等待 5 秒后（vTaskDelay(5000 / portTICK_PERIOD_MS);），调用 esp_now_send() 函数向广播地址 FF:FF:FF:FF:FF:FF 发送广播数据；在初始化时都会生成一个随机的 magic 数字（send_param->magic = esp_random();）当设备收到对方的广播消息后，会提取对方的 magic 数字并与自己的进行比较，如果自己的 magic 数字较大，则认为自己应当作为发送者，停止广播并开始单播发送
新手入门还是不要硬攻最复杂的版本 此时可以引入arduino库了 
在jlc工程库里面有一份项目：[使用ESPNOW远程点个灯](https://lceda002.feishu.cn/wiki/Oz0swQU6GiECd1kln6ccNZqUnhf)
里面使用了Thonny（一个简单易用的 Python 集成开发环境 (IDE)）；当然可以用vscode 用arduino库需要arduino as component
这里还是先试用thonny 配置很简单 [ESP32+Thonny环境搭建_thonny esp32](https://blog.csdn.net/shgg2917/article/details/138566167?fromshare=blogdetail&sharetype=blogdetail&sharerId=138566167&sharerefer=PC&sharesource=halfapear&sharefrom=from_link)
几乎没有各种配置 就很简单
![2024.2.15](/picture/esp32/12.png)  
<!-- 9/23 19:50开始 把最后一点突击完（还是不建议分太多次 低状态开写很容易想不起来怎么开始） -->       
移植一下
简单说一下流程——上电或复位时，固件开始执行 app_main() 函数；接着睡NVS、wifi等一堆初始化；然后当接收到数据时，会调用注册的 on_receive 回调函数
话说那while呢？就像arduino只有setup没有loop？——app_main() 函数不包含循环逻辑，但由于 ESP-IDF 内部运行了事件循环（由 esp_event_loop_create_default() 创建），程序会持续监听接收到的消息并根据不同的命令控制 LED 状态
<!-- 如果这里不简单讲一下具体逻辑 恐怕写这就不会有人看 没有任何价值 -->      
<!-- 然后跟着gpt很快就写完了 -->  
![2024.2.15](/picture/esp32/13.png)  

## esp32 综合
分别完成两个功能后 试着把两个挪到一块板子（比如s3）上
第六难：.bin 文件大小是 0x128da8（约 1.2 MB），而设置的闪存大小只有 1 MB（0x100000） 闪存爆了  —— 
![2024.2.15](/pictures/14.png) 
先把flash设高 jlc文档配置最后有讲或直接去menuconfig里面搜flash；
然后还是报错 这时需要修改分区表（感谢两位Y佬）
文档在[esp官方文档](https://docs.espressif.com/projects/esp-idf/zh_CN/stable/esp32/api-guides/partition-tables.html)：自己新建一个partitions.csv（记事本改名），将factory改成合适的值（需要整数）
![2024.2.15](/picture/esp32/15.png) 
    分区表可以写成

```c
# ESP-IDF Partition Table
# Name,   Type, SubType, Offset,  Size,   Flags
nvs,      data, nvs,     0x9000,  0x6000,
phy_init, data, phy,     0xf000,  0x1000,
factory,  app,  factory, 0x10000, 2M,
```

第七难：应用程序的二进制文件大小超过了为其分配的闪存分区空间 —— 是不是没有调flash大小或者调得比分区表小？

![2024.2.15](/picture/esp32/16.png) 

第八难：busy~ —— 是不是其他地方正在监控这个板子？

![2024.2.15](/picture/esp32/17.png) 

结束！新手任务完成了
待demo
<!-- 这种踩坑写法确实很费时间（当时收集和后期总结），但你现在事后也有点低估了当时一个坑的威力，而且写这种坑的题材也挺新奇 -->  
<!-- 9/23 20:30 任务完成 -->    




