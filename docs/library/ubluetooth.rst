:mod:`ubluetooth` --- 低功耗蓝牙
=========================================

.. module:: ubluetooth
   :synopsis: 无线低功耗蓝牙功能

该模块提供蓝牙控制器接口。
支持低功耗蓝牙（BLE）主控制, 外围设备控制,广播, 以及可见性, 并且一个设备可以同时进行多个
角色的操作。

该API用于匹配蓝牙低功耗（BLE）协议并提供用于高级抽象化（比如特定设备类型等）的模块。

BLE 类
---------

函数
-----------

.. class:: BLE()

    返回单个BLE对象。

配置
-------------

.. method:: BLE.active([active])

    无线低功耗蓝牙（BLE）的状态（可选项）, 返回当前状态。

    在使用其他任何一个库之前必须开启无线低功耗蓝牙（BLE）。

.. method:: BLE.config(name)

    通过*name*获取一个配置值. 现支持以下值:

    - ``'mac'``: 返回该设备的MAC地址。 如果该设备的MAC地址被修改
      (如PYBD) ，它将会返回一个值. 另外(如ESP32)当BLE接口被激活时，会生成一个随机的地址。

事件处理
--------------

.. method:: BLE.irq(handler, trigger=0xffff)

    注册一个BLE栈回调事件. *handler*需要两个对象, ``event`` (执行的函数)和``data``
    (特定事件的一组值).

    *trigger*（可选项）让你设置你的程序的目标事件。 默认设置为所有事件。

    一个事件处理展示所有可能的事件::

        def bt_irq(event, data):
            if event == _IRQ_CENTRAL_CONNECT:
                # 一个中央单元连接到这个外围设备
                conn_handle, addr_type, addr = data
            elif event == _IRQ_CENTRAL_DISCONNECT:
                # 一个中央单元与这个外围设备断开链接
                conn_handle, addr_type, addr = data
            elif event == _IRQ_GATTS_WRITE:
                # 中央单元已写入此特征或描述符
                conn_handle, attr_handle = data
            elif event == _IRQ_GATTS_READ_REQUEST:
                # 中央单元发布消息，这是一个硬中断请求
                # 返回None值以拒绝读取
                # 请注意：该事件不支持ESP32
                conn_handle, attr_handle = data
            elif event == _IRQ_SCAN_RESULT:
                # 单次扫描结果
                addr_type, addr, connectable, rssi, adv_data = data
            elif event == _IRQ_SCAN_COMPLETE:
                # 扫描过程完成或者被手动停止
                pass
            elif event == _IRQ_PERIPHERAL_CONNECT:
                # 一个成功的gap_connect()
                conn_handle, addr_type, addr = data
            elif event == _IRQ_PERIPHERAL_DISCONNECT:
                # 与已连接的外围设备断开
                conn_handle, addr_type, addr = data
            elif event == _IRQ_GATTC_SERVICE_RESULT:
                # 为gattc_discover_services()找到的每个服务调用
                conn_handle, start_handle, end_handle, uuid = data
            elif event == _IRQ_GATTC_CHARACTERISTIC_RESULT:
                # 为gattc_discover_services()找到的每个特征调用
                conn_handle, def_handle, value_handle, properties, uuid = data
            elif event == _IRQ_GATTC_DESCRIPTOR_RESULT:
                # 为gattc_discover_descriptors()找到的每个描述符调用
                conn_handle, dsc_handle, uuid = data
            elif event == _IRQ_GATTC_READ_RESULT:
                # 已完成的gattc_read()
                conn_handle, value_handle, char_data = data
            elif event == _IRQ_GATTC_WRITE_STATUS:
                # 已完成的gattc_write()
                conn_handle, value_handle, status = data
            elif event == _IRQ_GATTC_NOTIFY:
                # 外部设备已发送通知请求
                conn_handle, value_handle, notify_data = data
            elif event == _IRQ_GATTC_INDICATE:
                # 外部设备已发送指示请求
                conn_handle, value_handle, notify_data = data

以下是事件码::

    from micropython import const
    _IRQ_CENTRAL_CONNECT                 = const(1 << 0)
    _IRQ_CENTRAL_DISCONNECT              = const(1 << 1)
    _IRQ_GATTS_WRITE                     = const(1 << 2)
    _IRQ_GATTS_READ_REQUEST              = const(1 << 3)
    _IRQ_SCAN_RESULT                     = const(1 << 4)
    _IRQ_SCAN_COMPLETE                   = const(1 << 5)
    _IRQ_PERIPHERAL_CONNECT              = const(1 << 6)
    _IRQ_PERIPHERAL_DISCONNECT           = const(1 << 7)
    _IRQ_GATTC_SERVICE_RESULT            = const(1 << 8)
    _IRQ_GATTC_CHARACTERISTIC_RESULT     = const(1 << 9)
    _IRQ_GATTC_DESCRIPTOR_RESULT         = const(1 << 10)
    _IRQ_GATTC_READ_RESULT               = const(1 << 11)
    _IRQ_GATTC_WRITE_STATUS              = const(1 << 12)
    _IRQ_GATTC_NOTIFY                    = const(1 << 13)
    _IRQ_GATTC_INDICATE                  = const(1 << 14)

为了节省固件的空间, 这些内容没有包含在:mod:`ubluetooth` 模块，需自行从上面的列表中选择你所
需要的事件码到你的程序中。


广播规则（对外宣传者）
-----------------------------

.. method:: BLE.gap_advertise(interval_us, adv_data=None, resp_data=None, connectable=True)

    在指定时间间隔开始广播(**微**\ 秒). 该间隔将会精确到625us（微妙）。
    将*interval_us*设为``None``以停止广播。

    *adv_data*和*resp_data*可以是任何可实现的缓冲协议 (例如``bytes``, ``bytearray``, ``str``)。
    *adv_data*包含于任何广播里, *resp_data* 是对已激活的扫描仪的回复。

    说明：如果*adv_data* (或*resp_data*)是``None``, 接下来传递给上一个对``gap_advertise``调用
    的值将会被再次使用。
    This allows a broadcaster to resume advertising 这就意味着只要``gap_advertise(interval_us)``
    就可以让广播恢复对外宣传.
    提供一个空的``bytes``以清除广播负载，例如``b''``.


观察者角色（扫描仪）
-----------------------

.. method:: BLE.gap_scan(duration_ms, [interval_us], [window_us])

    持续扫描一段时间(**毫**\ 秒)。

    如果要让开发版一直扫描，请将*duration_ms*设置为``0``.

    如果要停止扫描，将*duration_ms*设置为``None``.

    扫描仪将每*interval_us*微秒运行*window_us*微秒，共*duration_ms*毫秒。
    默认选项分别为 1.28 秒和11.25毫秒(后台扫描).

    对于每个扫描结果，都会引发``_IRQ_SCAN_RESULT``事件。

    当扫描停止时(由于扫描过程结束或者明确地停止)，会引发``_IRQ_SCAN_COMPLETE``事件


外围角色 (GATT服务器)
-----------------------------

一个蓝牙外设已经有一套已经注册好的服务。每个服务可能包含带有一个值的特征，
特征也包含了自带值的描述符。

这些值都存储在本地，并由服务注册期间生成的“值句柄”访问。他们也被远程中央设备读取
或写入。另外，外围设备可以通过连接句柄将特征“通知”到连接的中央设备。

特征和描述有一个20字节的缺省最大值。
任何通过中央单元写入的内容将会被截断到此长度。然而，
任何一次本地写入将会增加最大尺寸，因此如果你想允许从中心向给定特征进行更大的写入,
在注册之后请使用:meth:`gatts_write<BLE.gatts_write>`。
例如：``gatts_write(char_handle, bytes(100))``

.. method:: BLE.gatts_register_services(services_definition)

    使用特定的服务配置外设, 替代任何设备。

    *services_definition*是一个**服务**列表，每一个**服务**是一个包含两
    个元素的元组，包含了一个UUID和一整个列表的**特征**。

    每个**特征** 是一个由两到三个元素的元组t，包含一个UUID，一个**flags**值，
    以及可选的*descriptors*列表。

    每个**描述符**是一个包含两个元素的元组，包含一个UUID和一个**flags**。

    **flags**是一个:data:`ubluetooth.FLAGS_READ`、:data:`bluetooth.FLAGS_WRITE`
    和:data:`ubluetooth.FLAGS_NOTIFY`的按位或组合。

    返回的是是一列表(一个服务一个元素)的元组(每一个元素有值句柄). 
    特征和描述符句柄按定义顺序展平到同一元组中。

    下面的示例注册了两个服务(心跳和Nordic通用异步收发器)::

        HR_UUID = bluetooth.UUID(0x180D)
        HR_CHAR = (bluetooth.UUID(0x2A37), bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY,)
        HR_SERVICE = (HR_UUID, (HR_CHAR,),)
        UART_UUID = bluetooth.UUID('6E400001-B5A3-F393-E0A9-E50E24DCCA9E')
        UART_TX = (bluetooth.UUID('6E400003-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY,)
        UART_RX = (bluetooth.UUID('6E400002-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_WRITE,)
        UART_SERVICE = (UART_UUID, (UART_TX, UART_RX,),)
        SERVICES = (HR_SERVICE, UART_SERVICE,)
        ( (hr,), (tx, rx,), ) = bt.gatts_register_services(SERVICES)

    这里有三个值句柄(``hr``, ``tx``, ``rx``)可用于:meth:`gatts_read <BLE.gatts_read>`, 
    :meth:`gatts_write <BLE.gatts_write>`和:meth:`gatts_notify <BLE.gatts_notify>`.

    **提示：** 注册服务前对外显示必须被停止。

.. method:: BLE.gatts_read(value_handle)

    读取此句柄的本地值(要么是由:meth:`gatts_write <BLE.gatts_write>`编写的，
    要么是由远程中央单元编写的).

.. method:: BLE.gatts_write(value_handle, data)

    写入此句柄的本地值，该值可以被中央单元读取。

.. method:: BLE.gatts_notify(conn_handle, value_handle, [data])

    通知已连接的中心设备此值已更改，并且它应该从该外围设备发出当前值的读取。

    如果指定了*data*，则该值将作为通知的一部分发送到中心，从而避免需要单独的
    读取请求。请注意，这不会更新存储的本地值。

.. method:: BLE.gatts_set_buffer(value_handle, len, append=False)

    设置以字节为单位的值的内部缓冲区大小。这将限制可接收的最大可能写操作。
    默认值是20。

    将*append*设为``True`` 将所有远程写入追加，而不是替换，当前的值。
    大部分*len*字节可以通过这个方式缓冲。
    当你使用:meth:`gatts_read <BLE.gatts_read>`，这个值将在读取后被清除。
    此功能在实现Nordic UART服务时非常有用。


中央处理规则(GATT客户端)
--------------------------

.. method:: BLE.gap_connect(addr_type, addr, scan_duration_ms=2000)

    连接到外设。

    连接成功后, 将引发``_IRQ_PERIPHERAL_CONNECT``事件。

.. method:: BLE.gap_disconnect(conn_handle)

    断开指定的连接句柄。

    断开成功后，将引发``_IRQ_PERIPHERAL_DISCONNECT``事件。

    如果链接句柄，返回``False``，``True``则相反。

.. method:: BLE.gattc_discover_services(conn_handle)

    查询连接的外围设备以获取其服务。

    对于发现的每个服务，都将引发``_IRQ_GATTC_SERVICE_RESULT``事件。

.. method:: BLE.gattc_discover_characteristics(conn_handle, start_handle, end_handle)

    查询连接的外围设备以获取指定范围内的特征。

    对于发现的每个特征，都会引发``_IRQ_GATTC_CHARACTERISTIC_RESULT``事件。

.. method:: BLE.gattc_discover_descriptors(conn_handle, start_handle, end_handle)

    查询连接的外设以查找指定范围内的描述符。

    对于发现的每个外设，都会引发``_IRQ_GATTC_DESCRIPTOR_RESULT``事件。

.. method:: BLE.gattc_read(conn_handle, value_handle)

    查询连接的外设以查找指定范围内的描述符。对连接的外设发出远程读取以
    获取指定的特征或描述符句柄。

    读取成功后, 将会引发``_IRQ_GATTC_READ_RESULT``事件。

.. method:: BLE.gattc_write(conn_handle, value_handle, data)

    为指定的特征或描述符句柄向连接的外围设备发出远程写入。

    写入成功后，将会引发``_IRQ_GATTC_WRITE_STATUS``事件。


class UUID
----------


Constructor
-----------

.. class:: UUID(value)

    创建具有指定**值**的UUID实例。

    **值**可以是：

    - 16位整数 例如 ``0x2908``.
    - 128位UUID字符串 例如 ``'6E400001-B5A3-F393-E0A9-E50E24DCCA9E'``.


常数
---------

.. data:: ubluetooth.FLAG_READ
          ubluetooth.FLAG_WRITE
          ubluetooth.FLAG_NOTIFY
