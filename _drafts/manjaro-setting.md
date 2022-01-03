 

## Manjaro 常用设置

1、manjaro 设置：https://my.oschina.net/langxSpirit/blog/1647000

2、显示异常 或 latte dock 无法设置背景颜色
    更改或重置渲染后端
    打开设置，依次进入：SystemSettings->Display and Monitor->Compositor, re-enable it from plasma 
    中文为：系统设置 -> 硬件 -> 显示和监控 -> 混成器

3、窗口最小化特效（魔灯）：系统设置 -> 工作空间行为 -> 桌面特殊

4、更改任务切换效果：系统设置 -> 窗口管理 -> 任务切换器 -> 主窗口中更换为“封面切换”

5、设置鼠标点击：系统设置 -> 工作空间行为 -> 常规行为 -> 点击行为中更改为“双击打开文件和文件夹(单击以选择)”

---------------------------------------------------------------------------------------------------
VMware 启动问题解决

https://blog.csdn.net/weixin_43968923/article/details/100184356

---------------------------------------------------------------------------------------------------
### 解决双显卡切换问题

1、安装 bumblebee、bbswitch（bumblebee 是用来选择使用哪个显卡运行程序，bbswtich 是用来开启或关闭显卡程序）
sudo yay -S bumblebee bbswitch

2、查看显卡状态
lspci | grep -i nvidia # 查看 nvidia 显卡
lspci | grep -i vga # 查看 intel 显卡

注意：如果显示的信息最后面是 “rev ff"，则表明显卡已经关闭；如果是 ”rev 数字“，则表明显卡已经开启

3、关闭或开启 Nvida 显卡
sudo tee /proc/acpi/bbswitch <<< ON # ON 开启，OFF 关闭

4、使用 bumblebee 指定显卡运行程序
optirun glxgears # 独立显卡运行
glxgears # 集成显卡运行