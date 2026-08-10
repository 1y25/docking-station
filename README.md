# 拓展坞 2 号板 · STM32F103C8T6 + 1.8寸 ST7735 屏幕

2 号拓展坞：屏幕**上接安装**（与 1 号板上下相反，需 `--flip-v`），横屏显示动画。

## 引脚连接表

| 信号 | STM32 引脚 | 说明 |
|---|---|---|
| 屏幕 SCK | **PB13** | SPI2 硬件时钟 |
| 屏幕 MOSI/SDA | **PB15** | SPI2 硬件数据 |
| 屏幕 RST | **PB11** | 复位 |
| 屏幕 DC | **PB10** | 数据/命令 |
| 屏幕 CS | **PB12** | 片选 |
| 屏幕背光 BL | **PB14** | 高电平亮（固件驱动） |
| W25Q64 CS | **PB1** | 图片存储 |
| W25Q64 CLK/DI/DO | PA5 / PA7 / PA6 | SPI1 |
| USART1 TX/RX | PA9 / PA10 | 串口烧录 + 调试（CH340） |
| 按键 | PA1(帧间隔) / PA2(切图库) | 按下接地 |
| LED 呼吸灯 | PC13 | TIM3 中断软件 PWM |

## 固件要点（与 1 号板的差异）

- **屏幕方向**：MADCTL `0x36 = 0xE0`（MV=1 横屏 + MX=1 镜像；此屏 **MY 位不生效**，上下靠图片翻转解决）
- **图片尺寸**：**162x130**（盖满屏幕显示区，坐标基准 162x130）
- **2 号板生成图必须加 `--flip-v`**（屏幕 MY 位失效，上下从数据层翻转）

## 编译烧录固件

1. Keil 打开 `o/project.uvprojx` → 器件 **STM32F103C8**
2. F7 编译 → LOAD 烧录（ST-Link）
3. 上电黑屏，动画启动后播放

## 动图制作 + 烧录

需要 Python 3 + `pip install pillow pyserial`：

```powershell
# 生成(注意: 2号板必须加 --flip-v, 尺寸162x130)
python TOOL/img1c.py 动图.gif --out .\mygif --size 162x130 --frames 25 --flip-v

# 烧录(改 TOOL/flashpic.py 的 PORT 为板子CH340的COM口)
python TOOL/flashpic.py .\mygif 2 25
```

- 每图库最多 25 帧（162x130 ≈ 21KB/帧），共 4 个图库
- 换图库：按键 PA2 或串口 `CHPIC-x`；调帧间隔：按键 PA1 或 `SetDelay-xx`

## 串口指令（9600）

| 指令 | 作用 |
|---|---|
| `CHPIC-x` | 切换图库 1~4 |
| `SetDelay-xx` | 帧间隔（支持 2~3 位，如 50 / 200） |
| `ReadInfo` | 查看各图库信息 |
| `COMMAND` | 帮助 |

## 工具

- `TOOL/img1c.py` — 图片/GIF → 帧 `.c`（支持 `--flip-v/--flip-h` 方向修正）
- `TOOL/img2c.py` — 同上（额外支持 `--autocenter` 自动居中）
- `TOOL/flashpic.py` — 串口烧录

## 仓库说明

本仓库（docking-station）只放 2 号板工程；1 号板在 `github.com/1y25/hub`。
