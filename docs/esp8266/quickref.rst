.. _esp8266_quickref:

ESP8266 快速参考手册
===============================

.. image:: img/adafruit_products_pinoutstop.jpg
    :alt: Adafruit Feather HUZZAH board
    :width: 640px

The Adafruit Feather HUZZAH board (image attribution: Adafruit).

以下是快速参考内容.  如果你是第一次使用ESP8266开发板，请考虑先阅读以下章节内容:

.. toctree::
   :maxdepth: 1

   general.rst
   tutorial/index.rst

安装 MicroPython
----------------------

请参考教程的相应部分: :ref:`intro`. 它还包括故障排除小节.

通用控制
---------------------

MicroPython 的串口交互调试（REPL）在 UART0 (GPIO1=TX, GPIO3=RX)，波特率为：115200。
Tab按键补全功能对于找到每个对象的使用方法非常有用。
粘贴模式 (ctrl-E) 对需要复制比较多的python代码到REPL非常有用。

The :mod:`machine` module::

    import machine

    machine.freq()          # 获取CPU当前工作频率
    machine.freq(160000000) # 设置CPU的工作频率为 160 MHz

The :mod:`esp` module::

    import esp

    esp.osdebug(None)       # 关闭原厂 O/S 调试信息
    esp.osdebug(0)          # 将原厂 O/S 调试信息重定向到 UART(0) 输出

Networking
----------

The :mod:`network` module::

    import network

    wlan = network.WLAN(network.STA_IF) # 创建station接口
    wlan.active(True)       # 激活接口
    wlan.scan()             # 搜索允许的访问SSID
    wlan.isconnected()      # 检查创建的station是否连接到AP
    wlan.connect('essid', 'password') # 连接到指定ESSID网络
    wlan.config('mac')      # 获取接口的MAC地址
    wlan.ifconfig()         # 获取接口的 IP/netmask(子网掩码)/gw(网关)/DNS 地址

    ap = network.WLAN(network.AP_IF) # 创捷一个AP热点接口
    ap.active(True)         # 激活接口
    ap.config(essid='ESP-AP') # 设置AP的ESSID名称
    
连接到本地WIFI网络的函数参考::

    def do_connect():
        import network
        wlan = network.WLAN(network.STA_IF)
        wlan.active(True)
        if not wlan.isconnected():
            print('connecting to network...')
            wlan.connect('essid', 'password')
            while not wlan.isconnected():
                pass
        print('network config:', wlan.ifconfig())

一旦网络建立成功，就可以通过 :mod:`socket <usocket>` 模块创捷和使用 TCP/UDP socket通讯。 

延时和时间
----------------

Use the :mod:`time <utime>` module::

    import time

    time.sleep(1)           # 睡眠1秒
    time.sleep_ms(500)      # 睡眠500毫秒
    time.sleep_us(10)       # 睡眠10微妙
    start = time.ticks_ms() # 获取毫秒计时器开始值
    delta = time.ticks_diff(time.ticks_ms(), start) # 计算从开始到当前时间的差值

定时器
------

支持虚拟 (基于RTOS) 定时器。使用 :ref:`machine.Timer <machine.Timer>` 模块通过设置 timer ID 号为 -1::

    from machine import Timer

    tim = Timer(-1) 
    tim.init(period=5000, mode=Timer.ONE_SHOT, callback=lambda t:print(1)) #1次
    tim.init(period=2000, mode=Timer.PERIODIC, callback=lambda t:print(2)) #周期

该周期的单位为毫秒(ms)。

引脚和GPIO口
-------------

使用 :ref:`machine.Pin <machine.Pin>` 模块::

    from machine import Pin

    p0 = Pin(0, Pin.OUT)    # 创建对象p0，对应GPIO0口输出
    p0.on()                 # 设置引脚为 "on" (1)高电平 
    p0.off()                # 设置引脚为 "off" (0)低电平
    p0.value(1)             # 设置引脚为 "on" (1)高电平 

    p2 = Pin(2, Pin.IN)     # 创建对象p2，对应GPIO2口输入
    print(p2.value())       # 获取引脚输入值, 0（低电平） or 1（高电平）

    p4 = Pin(4, Pin.IN, Pin.PULL_UP) # 打开内部上拉电阻
    p5 = Pin(5, Pin.OUT, value=1) # 初始化时候设置引脚的值为 1（高电平）

以下为可用引脚: 0, 1, 2, 3, 4, 5, 12, 13, 14, 15, 16, 分别对应ESP8266芯片的实际GPIO引脚编号。
请注意，很多用户使用自己的开发板有特定的引脚命名方式 (例如： D0, D1, ...)。由于MicroPython致力于支持
不同的开发板和模块，因此我们采用最原始简单具且有共同特征的引脚命名方式。如果你使用自己的开发板，请参考其原理图。

请注意，引脚 Pin(1) and Pin(3) 是串口交互（REPL） UART TX 和 RX 引脚.
同时请注意 Pin(16) 是一个特殊的引脚 (用于从深度睡眠模式中唤醒) 所以有可能不能使用高级的类模块如``Neopixel``.

PWM (脉宽调制)
----------------------------

PWM 可以通过所有引脚输出除了 Pin(16).  所有通道都有1个特定的频率，从1到1000之间（单位是Hz）。占空比的值为0至1023之间。

Use the ``machine.PWM`` class::

    from machine import Pin, PWM

    pwm0 = PWM(Pin(0))      # 从1个引脚中创建 PWM 对象
    pwm0.freq()             # 获取当前频率
    pwm0.freq(1000)         # 设置频率
    pwm0.duty()             # 获取当前占空比
    pwm0.duty(200)          # 设置占空比
    pwm0.deinit()           # 关闭引脚的 PWM

    pwm2 = PWM(Pin(2), freq=500, duty=512) # 在同一语句下创建和配置 PWM

ADC (模数转换)
----------------------------------

ADC 需要使用专用的引脚。请注意ADC引脚输入电压必须是0v 至 1.0v。

Use the :ref:`machine.ADC <machine.ADC>` class::

    from machine import ADC

    adc = ADC(0)            # 在ADC引脚上创建ADC对象
    adc.read()              # 读取测量值, 0-1024

软件 SPI总线
----------------

There are two SPI drivers. One is implemented in software (bit-banging)
and works on all pins, and is accessed via the :ref:`machine.SPI <machine.SPI>`
class::

    from machine import Pin, SPI

    # construct an SPI bus on the given pins
    # polarity is the idle state of SCK
    # phase=0 means sample on the first edge of SCK, phase=1 means the second
    spi = SPI(-1, baudrate=100000, polarity=1, phase=0, sck=Pin(0), mosi=Pin(2), miso=Pin(4))

    spi.init(baudrate=200000) # set the baudrate

    spi.read(10)            # read 10 bytes on MISO
    spi.read(10, 0xff)      # read 10 bytes while outputing 0xff on MOSI

    buf = bytearray(50)     # create a buffer
    spi.readinto(buf)       # read into the given buffer (reads 50 bytes in this case)
    spi.readinto(buf, 0xff) # read into the given buffer and output 0xff on MOSI

    spi.write(b'12345')     # write 5 bytes on MOSI

    buf = bytearray(4)      # create a buffer
    spi.write_readinto(b'1234', buf) # write to MOSI and read from MISO into the buffer
    spi.write_readinto(buf, buf) # write buf to MOSI and read MISO back into buf


硬件 SPI总线
----------------

The hardware SPI is faster (up to 80Mhz), but only works on following pins:
``MISO`` is GPIO12, ``MOSI`` is GPIO13, and ``SCK`` is GPIO14. It has the same
methods as the bitbanging SPI class above, except for the pin parameters for the
constructor and init (as those are fixed)::

    from machine import Pin, SPI

    hspi = SPI(1, baudrate=80000000, polarity=0, phase=0)

(``SPI(0)`` is used for FlashROM and not available to users.)

I2C总线
-------

The I2C driver is implemented in software and works on all pins,
and is accessed via the :ref:`machine.I2C <machine.I2C>` class::

    from machine import Pin, I2C

    # construct an I2C bus
    i2c = I2C(scl=Pin(5), sda=Pin(4), freq=100000)

    i2c.readfrom(0x3a, 4)   # read 4 bytes from slave device with address 0x3a
    i2c.writeto(0x3a, '12') # write '12' to slave device with address 0x3a

    buf = bytearray(10)     # create a buffer with 10 bytes
    i2c.writeto(0x3a, buf)  # write the given buffer to the slave

实时时钟 (RTC)
---------------------

See :ref:`machine.RTC <machine.RTC>` ::

    from machine import RTC

    rtc = RTC()
    rtc.datetime((2017, 8, 23, 1, 12, 48, 0, 0)) # set a specific date and time
    rtc.datetime() # get date and time

深度睡眠模式
---------------

Connect GPIO16 to the reset pin (RST on HUZZAH).  Then the following code
can be used to sleep, wake and check the reset cause::

    import machine

    # configure RTC.ALARM0 to be able to wake the device
    rtc = machine.RTC()
    rtc.irq(trigger=rtc.ALARM0, wake=machine.DEEPSLEEP)

    # check if the device woke from a deep sleep
    if machine.reset_cause() == machine.DEEPSLEEP_RESET:
        print('woke from a deep sleep')

    # set RTC.ALARM0 to fire after 10 seconds (waking the device)
    rtc.alarm(rtc.ALARM0, 10000)

    # put the device to sleep
    machine.deepsleep()

单总线驱动（OneWire）
--------------

The OneWire driver is implemented in software and works on all pins::

    from machine import Pin
    import onewire

    ow = onewire.OneWire(Pin(12)) # create a OneWire bus on GPIO12
    ow.scan()               # return a list of devices on the bus
    ow.reset()              # reset the bus
    ow.readbyte()           # read a byte
    ow.writebyte(0x12)      # write a byte on the bus
    ow.write('123')         # write bytes on the bus
    ow.select_rom(b'12345678') # select a specific device by its ROM code

There is a specific driver for DS18S20 and DS18B20 devices::

    import time, ds18x20
    ds = ds18x20.DS18X20(ow)
    roms = ds.scan()
    ds.convert_temp()
    time.sleep_ms(750)
    for rom in roms:
        print(ds.read_temp(rom))

Be sure to put a 4.7k pull-up resistor on the data line.  Note that
the ``convert_temp()`` method must be called each time you want to
sample the temperature.

NeoPixel 驱动
---------------

Use the ``neopixel`` module::

    from machine import Pin
    from neopixel import NeoPixel

    pin = Pin(0, Pin.OUT)   # set GPIO0 to output to drive NeoPixels
    np = NeoPixel(pin, 8)   # create NeoPixel driver on GPIO0 for 8 pixels
    np[0] = (255, 255, 255) # set the first pixel to white
    np.write()              # write data to all pixels
    r, g, b = np[0]         # get first pixel colour

For low-level driving of a NeoPixel::

    import esp
    esp.neopixel_write(pin, grb_buf, is800khz)

APA102 驱动
-------------

Use the ``apa102`` module::

    from machine import Pin
    from apa102 import APA102

    clock = Pin(14, Pin.OUT)     # set GPIO14 to output to drive the clock
    data = Pin(13, Pin.OUT)      # set GPIO13 to output to drive the data
    apa = APA102(clock, data, 8) # create APA102 driver on the clock and the data pin for 8 pixels
    apa[0] = (255, 255, 255, 31) # set the first pixel to white with a maximum brightness of 31
    apa.write()                  # write data to all pixels
    r, g, b, brightness = apa[0] # get first pixel colour

For low-level driving of an APA102::

    import esp
    esp.apa102_write(clock_pin, data_pin, rgbi_buf)

DHT 驱动
----------

The DHT driver is implemented in software and works on all pins::

    import dht
    import machine

    d = dht.DHT11(machine.Pin(4))
    d.measure()
    d.temperature() # eg. 23 (°C)
    d.humidity()    # eg. 41 (% RH)

    d = dht.DHT22(machine.Pin(4))
    d.measure()
    d.temperature() # eg. 23.6 (°C)
    d.humidity()    # eg. 41.3 (% RH)

WebREPL (Web浏览器交互提示)
----------------------------------------

WebREPL (REPL over WebSockets, accessible via a web browser) is an
experimental feature available in ESP8266 port. Download web client
from https://github.com/micropython/webrepl (hosted version available
at http://micropython.org/webrepl), and configure it by executing::

    import webrepl_setup

and following on-screen instructions. After reboot, it will be available
for connection. If you disabled automatic start-up on boot, you may
run configured daemon on demand using::

    import webrepl
    webrepl.start()

The supported way to use WebREPL is by connecting to ESP8266 access point,
but the daemon is also started on STA interface if it is active, so if your
router is set up and works correctly, you may also use WebREPL while connected
to your normal Internet access point (use the ESP8266 AP connection method
if you face any issues).

Besides terminal/command prompt access, WebREPL also has provision for file
transfer (both upload and download). Web client has buttons for the
corresponding functions, or you can use command-line client ``webrepl_cli.py``
from the repository above.

See the MicroPython forum for other community-supported alternatives
to transfer files to ESP8266.
