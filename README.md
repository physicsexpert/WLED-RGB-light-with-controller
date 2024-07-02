@[TOC](文章目录)

---
# 前言
WLED是一个开源项目，首先放上[开源地址](https://github.com/Aircoookie/WLED/releases)，基于此开源项目的控制器原理图来自up主：[HACK实验室](https://space.bilibili.com/395145107?spm_id_from=333.337.0.0)，感谢大佬开源的项目文件，我在其原理图的基础上，将其绘制在嘉立创EDA专业版上（原PCB为KICAD绘制），该控制器主控采用ESP32模组，具有两路WS281X系列灯带控制，五路PWM控制，两种输出电压切换等功能，兼容市面上几乎所有的灯带产品，可以免费白嫖，需要资源请私信我
完整原理图、PCB、外壳如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ac9fe1e7c454e1d9ac1742402031763.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8675ed43c90c4c0894ce990dae8f579e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/54047c48c9e54e989f968de27a2b1857.png)


---
# 一、CH224K构成快充诱骗电路
由于不同的灯带需要不同的输出电压，所以需要一个快充管理芯片进行快充诱骗将电压提高，这里使用了南京沁恒的CH224K，CH224 单芯片集成 USB PD 等多种快充协议，支持 PD3.0/2.0，BC1.2 等升压快充协议，自动检测VCONN 及模拟 E-Mark 芯片，最高支持 100W 功率，内置 PD 通讯模块，集成度高，外围精简。集成输出电压检测功能，并且提供过温、过压保护等功能。
通过数据手册可以了解到该芯片的工作原理，如果我们需要请求充电器输出不同的电压，就需要给给2，3，9引脚提供不同的高低电平。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b2e244fb32e44b1b450e1e6274bf3a0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/acd4ce6bc02f4abe94466d48c10e1780.png)

原理图如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/b46c6caf45af4e53aedf622b65fd437c.png)
# 二、输出电压切换电路
接上一段，当我们需要电压切换功能是，一个个改变引脚电平是不方便的，所以这里使用了一个六角两档开关
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c8e70e4cab54983af5c5d48852addd6.png)
当开关拨到左边时，CFG1与VCFG接通，VBUS与+5V接通，CH224K的电压选择引脚构成101，请求电压为5V,同时VBUS与+5V接通，VBUS直接接入+5V网络，不经过变压电路。
当开关拨到右边时，CFG1与GND接通，VBUS与VBUS_L接通，CH224K的电压选择引脚构成001，请求电压为12V,同时VBUS与VBUS_L接通，VBUS接入+5V变压电路，输出12V和5V。




# 三、电压掉电恢复电路
![在这里插入图片描述](https://img-blog.csdnimg.cn/77230428dac64520b0cd2f4068cde5a1.png)
请求电压在5V和12V切换的时候，我们都需要使用5V的电压，所以采用BAT54C构成掉电恢复电路
其工作原理如下：当请求电压为5V时，+5V端有5V电压，左侧LDO由于无输入电压所以输出浮地，进而BAT54C的右侧肖特基二极管导通，5V_L由右侧+5V供电，输出为5V。当请求电压为12V时，VBUS_L端有12V电压，左侧LDO输出5V，+5V网络无电压，进而BAT54C的左侧肖特基二极管导通，5V_L由左侧LDO供电，输出为5V。



# 四、PWM输出驱动电路
![在这里插入图片描述](https://img-blog.csdnimg.cn/714dd3d9c35b44328885db24ebb19903.png)
PWM输出采用AO3400N沟道场效应管构成输出驱动电路，G极下拉，防止上电误触发。





# 五、电平转换电路
![在这里插入图片描述](https://img-blog.csdnimg.cn/55f2baf11d52483a9230bbda79e4b47e.png)
由SN74HC244构成的电平转换电路，将ESP32IO输出的3.3V转换为5V，确保NMOS可以完全打开。

