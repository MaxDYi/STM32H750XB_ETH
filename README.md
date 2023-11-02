# STM32H750XB_ETH

# 0.前言

本教程仅适用于H74x/H75x，对于新版的H72x/H73x方案可能需要修改（未经过测试）。

F4系列中不适用MPU，Lwip的使用可参考配置，网上教程非常多。

# 1.软件和固件环境

- STM32CubeMX 6.9.1
- Firmware Package Version：STM32Cube FW H7 V1.11.1
- Lwip：V2.1.2
- FreeRTOS：CMSIS_V1
- IAR：8.50

# 2.创建工程

## 2.1 配置晶振

使用外部时钟晶振，注意H7系列分V和Y两个版本，可通过观察芯片丝印确定，两版本主频不同（V:480M Y:400M）。

![image-20231102111847710](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021118871.png)

## 2.2 配置MPU

MPU是内存保护单元，共配置2个区域。第一个区域是eth的收发buffer共256B，第二个区域是Lwip协议栈的内存共16KB。

![image-20231102112525368](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021125471.png)

## 2.2 配置ETH

ETH中使用RMII接口。

![image-20231102112950086](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021129179.png)

打开全局中断。

![image-20231102113051431](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021130470.png)

将所有GPIO引脚速度设置为VeryHigh。

![image-20231102113239163](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021132251.png)

## 2.3 配置Lwip

启用Lwip，关闭DHCP，设置静态IP、子网掩码与网关。

![image-20231102113619256](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021136344.png)

PHY芯片选择LAN8742，该芯片可兼容LAN8720系列。

![image-20231102113714525](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021137567.png)

配置Key Options中其他参数，勾选右上角高级设置，开启动态内存。

![image-20231102113920849](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021139903.png)

开启函数回调。

![image-20231102114117251](https://raw.githubusercontent.com/MaxDYi/PicGo/main/202311021141302.png)

## 2.4 FreeRTOS

操作系统可进行选配，如果启用操作系统优先使用TIM6作为系统时钟，该定时器功能较少。

### 3 代码调试

生成工程后进行串口重定向，使用printf进行串口输出。

在主函数循环中加入MX_LWIP_Process();即可进行调试。

如果使用了FreeRTOS操作系统，则在默认task中进行了Lwip的配置，无需添加任何代码即可进行调试。

在lwipopts.h中加入以下代码可打开代码调试输出，但会导致网络延迟高，在实际应用时需要关闭。

```C
#define LWIP_DEBUG

#define LWIP_DBG_TYPES_ON               LWIP_DBG_ON
/* USER CODE BEGIN 1 */
#define LWIP_DBG_MIN_LEVEL              LWIP_DBG_LEVEL_OFF
//#define LWIP_DBG_MIN_LEVEL              LWIP_DBG_LEVEL_WARNING
//#define LWIP_DBG_MIN_LEVEL              LWIP_DBG_LEVEL_SERIOUS
//#define LWIP_DBG_MIN_LEVEL              LWIP_DBG_LEVEL_SEVERE
 
#define LWIP_DBG_TYPES_ON               LWIP_DBG_ON
//#define LWIP_DBG_TYPES_ON               (LWIP_DBG_ON|LWIP_DBG_TRACE|LWIP_DBG_STATE|LWIP_DBG_FRESH)
 
#define ETHARP_DEBUG                    LWIP_DBG_ON     
#define NETIF_DEBUG                     LWIP_DBG_ON     
#define PBUF_DEBUG                      LWIP_DBG_ON
#define API_LIB_DEBUG                   LWIP_DBG_ON
#define API_MSG_DEBUG                   LWIP_DBG_ON
#define SOCKETS_DEBUG                   LWIP_DBG_ON
#define ICMP_DEBUG                      LWIP_DBG_ON
#define IGMP_DEBUG                      LWIP_DBG_ON
#define INET_DEBUG                      LWIP_DBG_ON
#define IP_DEBUG                        LWIP_DBG_ON     
#define IP_REASS_DEBUG                  LWIP_DBG_ON
#define RAW_DEBUG                       LWIP_DBG_ON
#define MEM_DEBUG                       LWIP_DBG_ON
#define MEMP_DEBUG                      LWIP_DBG_ON
#define SYS_DEBUG                       LWIP_DBG_ON
#define TCP_DEBUG                       LWIP_DBG_ON
#define TCP_INPUT_DEBUG                 LWIP_DBG_ON
#define TCP_FR_DEBUG                    LWIP_DBG_ON
#define TCP_RTO_DEBUG                   LWIP_DBG_ON
#define TCP_CWND_DEBUG                  LWIP_DBG_ON
#define TCP_WND_DEBUG                   LWIP_DBG_ON
#define TCP_OUTPUT_DEBUG                LWIP_DBG_ON
#define TCP_RST_DEBUG                   LWIP_DBG_ON
#define TCP_QLEN_DEBUG                  LWIP_DBG_ON
#define UDP_DEBUG                       LWIP_DBG_ON     
#define TCPIP_DEBUG                     LWIP_DBG_ON
#define PPP_DEBUG                       LWIP_DBG_ON
#define SLIP_DEBUG                      LWIP_DBG_ON
#define DHCP_DEBUG                      LWIP_DBG_ON     
#define AUTOIP_DEBUG                    LWIP_DBG_ON
#define SNMP_MSG_DEBUG                  LWIP_DBG_ON
#define SNMP_MIB_DEBUG                  LWIP_DBG_ON
#define DNS_DEBUG                       LWIP_DBG_ON
```

