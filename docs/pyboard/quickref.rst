.. _pyboard_quickref:

pyboard 快速参考手册
===============================

以下是PYBv1.1-CN引脚图：

.. only:: not latex

   .. image:: http://www.01studio.org/micropython/picture/pyboard_v1.1-CN_pinout.png
      :alt: PYBv1.1 pinout
      :width: 700px

.. only:: latex

   .. image:: http://www.01studio.org/micropython/picture/pyboard_v1.1-CN_pinout.png
      :alt: PYBv1.1 pinout

以下是快速参考内容，如果你是第一次使用pyboard，请考虑先阅读以下章节内容：

.. toctree::
   :maxdepth: 1

   general.rst
   tutorial/index.rst

通用控制
---------------------
See :mod:`pyb`. ::

    import pyb

    pyb.repl_uart(pyb.UART(1, 9600)) # 复制 REPL 到 UART(1)
    pyb.wfi() # 暂停 CPU，等待中断
    pyb.freq() # 获取 CPU 和总线的频率
    pyb.freq(60000000) # 设置 CPU 工作频率为 60MHz
    pyb.stop() # 暂停 CPU, 等待外部中断

延时和计时器
----------------

使用 :mod:`time <utime>` 模块::

    import time

    time.sleep(1)           # 睡眠1s
    time.sleep_ms(500)      # 睡眠500ms
    time.sleep_us(10)       # 睡眠10ms
    start = time.ticks_ms() # 获取毫秒计数器的值
    delta = time.ticks_diff(time.ticks_ms(), start) # 计算启动时间

板载LEDs
-------------

See :ref:`pyb.LED <pyb.LED>`. ::

    from pyb import LED

    led = LED(1) # 1=红, 2=绿, 3=黄, 4=蓝
    led.toggle()
    led.on()
    led.off()
    
    # LEDs 3 和 4 支持 PWM 调节亮度 (0-255)
    LED(4).intensity()    # 获取亮度值
    LED(4).intensity(128) # 设置亮度值为一半

板载按键
---------------

See :ref:`pyb.Switch <pyb.Switch>`. ::

    from pyb import Switch

    sw = Switch()
    sw.value() # 返回 True 或者 False
    sw.callback(lambda: pyb.LED(1).toggle()) #按键按下执行相关函数

引脚和GPIO口
-------------

See :ref:`pyb.Pin <pyb.Pin>`. ::

    from pyb import Pin

    p_out = Pin('X1', Pin.OUT_PP)
    p_out.high()
    p_out.low()

    p_in = Pin('X2', Pin.IN, Pin.PULL_UP)
    p_in.value() # 获取数值, 0 或者 1

舵机控制
-------------

See :ref:`pyb.Servo <pyb.Servo>`. ::

    from pyb import Servo

    s1 = Servo(1) # 舵机连接到接口1 (X1, VIN, GND)
    s1.angle(45) # 旋转到45°位置
    s1.angle(-60, 1500) # 在1500毫秒内转到-60°的位置
    s1.speed(50) # 适用于连续旋转舵机

外部中断
-------------------

See :ref:`pyb.ExtInt <pyb.ExtInt>`. ::

    from pyb import Pin, ExtInt

    callback = lambda e: print("intr")
    ext = ExtInt(Pin('Y1'), ExtInt.IRQ_RISING, Pin.PULL_NONE, callback)

计时器
------

See :ref:`pyb.Timer <pyb.Timer>`. ::

    from pyb import Timer

    tim = Timer(1, freq=1000)
    tim.counter() # 获取计时器数值
    tim.freq(0.5) # 0.5 Hz
    tim.callback(lambda t: pyb.LED(1).toggle())

实时时钟
---------------------

See :ref:`pyb.RTC <pyb.RTC>` ::

    from pyb import RTC

    rtc = RTC()
    rtc.datetime((2017, 8, 23, 1, 12, 48, 0, 0)) # 设置日期和时间
    rtc.datetime() # 获取日期和时间

PWM (脉宽调变) 
----------------------------

See :ref:`pyb.Pin <pyb.Pin>` and :ref:`pyb.Timer <pyb.Timer>`. ::

    from pyb import Pin, Timer

    p = Pin('X1') # X1 has TIM2, CH1
    tim = Timer(2, freq=1000)
    ch = tim.channel(1, Timer.PWM, pin=p)
    ch.pulse_width_percent(50)

ADC (模数转换) 
----------------------------------

See :ref:`pyb.Pin <pyb.Pin>` and :ref:`pyb.ADC <pyb.ADC>`. ::

    from pyb import Pin, ADC

    adc = ADC(Pin('X19'))
    adc.read() # 读取数值, 0-4095

DAC (数模转换) 
----------------------------------

See :ref:`pyb.Pin <pyb.Pin>` and :ref:`pyb.DAC <pyb.DAC>`. ::

    from pyb import Pin, DAC

    dac = DAC(Pin('X5'))
    dac.write(120) # 输出数值 0 至 255

UART(串行总线) 
-----------------

See :ref:`pyb.UART <pyb.UART>`. ::

    from pyb import UART

    uart = UART(1, 9600)
    uart.write('hello')
    uart.read(5) # 读取 5 个字节

SPI总线
-------

See :ref:`pyb.SPI <pyb.SPI>`. ::

    from pyb import SPI

    spi = SPI(1, SPI.MASTER, baudrate=200000, polarity=1, phase=0)
    spi.send('hello')
    spi.recv(5) # 接收5个字节
    spi.send_recv('hello') # 发送和接收5个字节

I2C总线
-------

Hardware I2C is available on the X and Y halves of the pyboard via ``I2C('X')``
and ``I2C('Y')``.  Alternatively pass in the integer identifier of the peripheral,
eg ``I2C(1)``.  Software I2C is also available by explicitly specifying the
``scl`` and ``sda`` pins instead of the bus name.  For more details see
:ref:`machine.I2C <machine.I2C>`. ::

    from machine import I2C

    i2c = I2C('X', freq=400000)                 # create hardware I2c object
    i2c = I2C(scl='X1', sda='X2', freq=100000)  # create software I2C object

    i2c.scan()                          # returns list of slave addresses
    i2c.writeto(0x42, 'hello')          # write 5 bytes to slave with address 0x42
    i2c.readfrom(0x42, 5)               # read 5 bytes from slave

    i2c.readfrom_mem(0x42, 0x10, 2)     # read 2 bytes from slave 0x42, slave memory 0x10
    i2c.writeto_mem(0x42, 0x10, 'xy')   # write 2 bytes to slave 0x42, slave memory 0x10

Note: for legacy I2C support see :ref:`pyb.I2C <pyb.I2C>`.

CAN总线 (controller area network)
---------------------------------

See :ref:`pyb.CAN <pyb.CAN>`. ::

    from pyb import CAN

    can = CAN(1, CAN.LOOPBACK)
    can.setfilter(0, CAN.LIST16, 0, (123, 124, 125, 126))
    can.send('message!', 123)   # send a message with id 123
    can.recv(0)                 # receive message on FIFO 0

板载三轴加速度传感器
----------------------

See :ref:`pyb.Accel <pyb.Accel>`. ::

    from pyb import Accel

    accel = Accel()
    print(accel.x(), accel.y(), accel.z(), accel.tilt()) #打印X,Y,Z值
