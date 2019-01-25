.. _pyboard_quickref:

pyboard 快速参考手册
===============================

The below pinout is for PYBv1.0.  You can also view pinouts for
other versions of the pyboard:
`PYBv1.1 <http://micropython.org/resources/pybv11-pinout.jpg>`__
or `PYBLITEv1.0-AC <http://micropython.org/resources/pyblitev10ac-pinout.jpg>`__
or `PYBLITEv1.0 <http://micropython.org/resources/pyblitev10-pinout.jpg>`__.

.. only:: not latex

   .. image:: http://micropython.org/resources/pybv10-pinout.jpg
      :alt: PYBv1.0 pinout
      :width: 700px

.. only:: latex

   .. image:: http://micropython.org/resources/pybv10-pinout-800px.jpg
      :alt: PYBv1.0 pinout

Below is a quick reference for the pyboard.  If it is your first time working with
this board please consider reading the following sections first:

.. toctree::
   :maxdepth: 1

   general.rst
   tutorial/index.rst

通用控制
---------------------
看 :mod:`pyb`. ::

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

    led = LED(1) # 1=red, 2=green, 3=yellow, 4=blue
    led.toggle()
    led.on()
    led.off()
    
    # LEDs 3 and 4 support PWM intensity (0-255)
    LED(4).intensity()    # get intensity
    LED(4).intensity(128) # set intensity to half

板载按键
---------------

See :ref:`pyb.Switch <pyb.Switch>`. ::

    from pyb import Switch

    sw = Switch()
    sw.value() # returns True or False
    sw.callback(lambda: pyb.LED(1).toggle())

引脚和GPIO口
-------------

See :ref:`pyb.Pin <pyb.Pin>`. ::

    from pyb import Pin

    p_out = Pin('X1', Pin.OUT_PP)
    p_out.high()
    p_out.low()

    p_in = Pin('X2', Pin.IN, Pin.PULL_UP)
    p_in.value() # get value, 0 or 1

伺服电机控制
-------------

See :ref:`pyb.Servo <pyb.Servo>`. ::

    from pyb import Servo

    s1 = Servo(1) # servo on position 1 (X1, VIN, GND)
    s1.angle(45) # move to 45 degrees
    s1.angle(-60, 1500) # move to -60 degrees in 1500ms
    s1.speed(50) # for continuous rotation servos

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
    tim.counter() # get counter value
    tim.freq(0.5) # 0.5 Hz
    tim.callback(lambda t: pyb.LED(1).toggle())

实时时钟
---------------------

See :ref:`pyb.RTC <pyb.RTC>` ::

    from pyb import RTC

    rtc = RTC()
    rtc.datetime((2017, 8, 23, 1, 12, 48, 0, 0)) # set a specific date and time
    rtc.datetime() # get date and time

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
    adc.read() # read value, 0-4095

DAC (数模转换) 
----------------------------------

See :ref:`pyb.Pin <pyb.Pin>` and :ref:`pyb.DAC <pyb.DAC>`. ::

    from pyb import Pin, DAC

    dac = DAC(Pin('X5'))
    dac.write(120) # output between 0 and 255

UART(串行总线) 
-----------------

See :ref:`pyb.UART <pyb.UART>`. ::

    from pyb import UART

    uart = UART(1, 9600)
    uart.write('hello')
    uart.read(5) # read up to 5 bytes

SPI总线
-------

See :ref:`pyb.SPI <pyb.SPI>`. ::

    from pyb import SPI

    spi = SPI(1, SPI.MASTER, baudrate=200000, polarity=1, phase=0)
    spi.send('hello')
    spi.recv(5) # receive 5 bytes on the bus
    spi.send_recv('hello') # send and receive 5 bytes

I2C总线
-------

See :ref:`pyb.I2C <pyb.I2C>`. ::

    from pyb import I2C

    i2c = I2C(1, I2C.MASTER, baudrate=100000)
    i2c.scan() # returns list of slave addresses
    i2c.send('hello', 0x42) # send 5 bytes to slave with address 0x42
    i2c.recv(5, 0x42) # receive 5 bytes from slave
    i2c.mem_read(2, 0x42, 0x10) # read 2 bytes from slave 0x42, slave memory 0x10
    i2c.mem_write('xy', 0x42, 0x10) # write 2 bytes to slave 0x42, slave memory 0x10

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
    print(accel.x(), accel.y(), accel.z(), accel.tilt())
