---
layout: post
title: Ubuntu resolution
---

### 리눅스에서 모니터 해상도 작게 나오는 이슈

```
$ xrandr
Screen 0: minimum 320 x 200, current 1024 x 768, maximum 16384 x 16384
DisplayPort-0 disconnected (normal left inverted right x axis y axis)
DVI-0 connected primary 1024x768+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   1024x768      60.00* 
   800x600       60.32    56.25  
   848x480       60.00  
   640x480       59.94  
  1280x768_60.00 (0x4db) 79.500MHz -HSync +VSync
        h: width  1280 start 1344 end 1472 total 1664 skew    0 clock  47.78KHz
        v: height  768 start  771 end  781 total  798           clock  59.87Hz
        
$ cvt 1920 1080
# 1920x1080 59.96 Hz (CVT 2.07M9) hsync: 67.16 kHz; pclk: 173.00 MHz
Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync

# 위 Modeline 뒤의 Text를 복사하여 아래 newmode로 등록한다.
$ xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
$ xrandr --addmode DVI-0 "1920x1080_60.00"
$ xrandr --output DVI-0 --mode "1920x1080_60.00"
```


## 리부팅시 자동으로 설정되도록 적용

```
$ vi ~/.profile
...
# setup 1920x1080 resolution
xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
xrandr --addmode DVI-0 "1920x1080_60.00"
xrandr --output DVI-0 --mode "1920x1080_60.00"
...
```
