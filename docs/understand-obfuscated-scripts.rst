.. _了解加密脚本:

了解加密脚本
============

.. _全局密钥箱:

全局密钥箱
----------

全局密钥箱是存放在用户主目录的一个文件 :file:`.pyarmor_capsule.zip` 。当 PyArmor
加密脚本或者生成加密脚本的许可文件的时候，都需要从这个文件中读取数据。

所有的试用版本使用同一个密钥箱 `公共密钥箱` ，这个密钥箱使用 1024 位 RSA 密钥对。

而对于正式版本，每一个用户都会收到一个专用的 `私有密钥箱` ，这种密钥箱使用 2048
位 RSA 密钥对。

通过密钥箱是无法恢复加密后的脚本，所以即便都使用 `公共密钥箱` ，加密后的脚本也是
安全的，但是使用相同的密钥箱为加密脚本生成的许可是通用的。使用 `公共密钥箱` 是无
法真正的限制加密脚本的使用期限或者绑定到某一台指定设备上，因为别人同样可以使用
`公共密钥箱` 为你的加密脚本生成合法的许可。

运行加密脚本并不需要密钥箱，只有在加密脚本和为加密脚本生成许可的时候才会用到。

加密后的脚本
------------

和原来的脚本相比，被 PyArmor 加密后的脚本需要额外的运行辅助文件，下面是加密后在
输出目录 `dist` 下的所有文件清单::

    myscript.py
    mymodule.py

    pytransform/
        __init__.py
        _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
        pytransform.key
        license.lic

被加密的脚本也是一个普通的 Python 脚本，模块 `dist/mymodule.py` 加密后会是这样::

    __pyarmor__(__name__, __file__, b'\x06\x0f...')

而主脚本 `dist/myscript.py` 被加密后则会是这样::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'\x0a\x02...')

.. _主脚本:

主脚本
~~~~~~

在 PyArmor 中，主脚本并不一定是启动脚本，而是在运行 Python 解释器之后，第一个被
执行（导入）的加密脚本。例如，如果只有一个 Python 包的被加密，那么这个包的
`__init__.py` 就是主脚本。

.. _引导代码:

引导代码
--------

主脚本的前两行就是 `引导代码` ，它一般出现在主脚本中::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()

对于被加密的 Python 包，其主脚本是 `__init__.py` ，那么引导代码会使用相对导入的
方式::

    from .pytransform import pyarmor_runtime
    pyarmor_runtime()

如果加密脚本的时候指定了运行时刻路径，引导代码会是这样的形式::

    from pytransform import pyarmor_runtime
    pyarmor_runtime('/path/to/runtime')

.. _运行辅助包:

运行辅助包
----------

和加密脚本一起生成的目录 `pytransform` 是一个 Python 的包，叫做 `运行辅助包` 。
它是运行加密脚本的唯一依赖，只要它在任何一个 Python 路径下面，加密脚本就可以像普
通脚本一样被正常执行。

通常情况下面运行辅助包和加密脚本在相同目录下，但是也可以在其他任何路径，只要可以
使用 `import pytransform` 能够正常导入就可以。并且使用相同 :ref:`全局密钥箱` 加
密的脚本都可以共用这个包。 你完全可以把这个包放到 Python 的系统库路径下面，这样
加密后的脚本目录结构就和原来完全一样。

运行辅助包里面有四个文件::

    pytransform/
        __init__.py                  Python 模块文件
        _pytransform.so/.dll/.lib    动态链接库，核心功能的实现
        pytransform.key              数据文件
        license.lic                  加密脚本的许可文件


在 v5.7.0 之前, 运行辅助包是另外一种存放形式 `运行辅助文件`

.. _运行辅助文件:

运行辅助文件
~~~~~~~~~~~~
它们不是以 Python 包的形式存在，而是四个单独文件::

    pytransform.py               Python 模块文件
    _pytransform.so/.dll/.lib    动态链接库，核心功能的实现
    pytransform.key              数据文件
    license.lic                  加密脚本的许可文件

很明显，使用运行辅助包的形式使得加密后的脚本目录结构更清晰。

.. _加密脚本的许可文件:

加密脚本的许可文件
------------------

运行辅助文件中的 `license.lic` 作用比较特殊，它包含着对加密脚本的运行许可信息。
在加密脚本的同时会在输出目录下面生成一个默认许可文件，该文件允许加密脚本运行在任
何机器并且永不过期。

如果需要为加密脚本设置新的许可，例如设置有效期，限制加密脚本在特定机器上运行，需
要运行命令 :ref:`licenses` 生成新的相应的许可文件，然后用新生成的 `license.lic`
覆盖原来的许可文件。

.. note::

    PyArmor 的安装目录下面也有一个 `license.lic` ，这个文件主要是设置 PyArmor 自
    身的许可，这个许可是由我来发布的，：）

使用加密脚本的基本原则
----------------------

* 加密后的脚本也是一个正常的 Python 脚本，它可以无缝替换原来的脚本

* 唯一的改变是， `引导代码`_ 必须被首先执行，加密脚本才能正常运行，否则会报错。

* `运行辅助包`_ 必须在任何 Python 路径下面，确保 `引导代码`_ 能被正确导入。

* `引导代码`_ 会使用 `ctypes` 装载动态库 `_pytransform.so/.dll/.dylib` 。动态库
  是平台相关的，所有预编译的动态库列表在这里 :ref:`支持的平台列表`

* 默认情况下， `引导代码`_ 会在 `运行辅助包`_ 里面搜索动态库 `_pytransform` ，具
  体装载过程可以查看函数 `pytransform._load_library` 的源代码。

* 如果动态库 `_pytransform` 没有在默认位置，需要修改 `引导代码`_ ,设置运行时刻路
  径::

    from pytransform import pyarmor_runtime
    pyarmor_runtime('/path/to/runtime')

  运行辅助文件 `license.lic` 和 `pytransform.key` 必须也在这个目录下面

* 如果在代码中动态创建新的执行环境，例如 `multiprocssing.Process`, `os.exec`,
  `subprocess.Popen` 等, 要确保 `引导代码`_ 在新的执行环境被首先执行，否则加密脚
  本会报错。

更多详细的信息，可以参考 :ref:`如何加密脚本` 和 :ref:`如何运行加密脚本`

.. _加密脚本和原脚本的区别:

加密脚本和原脚本的区别
----------------------

加密脚本和原来的脚本相比，存在下列一些的不同:

* 运行加密脚本的 Python 主版本和加密脚本使用的 Python 主版本应该要一致，
  因为加密的脚本实际上已经是 `.pyc` 文件，如果主版本不一致，有些指令无
  法识别或者会出错。尤其是 Python3.6，在这个版本引入了新的指令系统，所
  以和 Python3.5 以及之前的版本完全不同。

* 执行加密角本的 Python 不能是调试版，准确的说，不能是设置了
  Py_TRACE_REFS 或者 Py_DEBUG 生成的 Python

* 使用 ``sys.settrace``, ``sys.setprofile``, ``threading.settrace`` 和
  ``threading.setprofile`` 设置的回调函数在加密脚本中将被忽略

* 代码块的属性 ``__file__`` 在加密脚本是 ``<frozen name>`` ，而不是文件
  名称，在异常信息中会看到文件名的显示是 ``<frozen name>``

  需要注意的是模块的属性 ``__file__`` 还和原来的一样，还是文件名称。加
  密下面的脚本并运行，就可以看到输出结果的不同::

      def hello(msg):
          print(msg)

      # The output will be 'foo.py'
      print(__file__)

      # The output will be '<frozen foo>'
      print(hello.__file__)

.. include:: _common_definitions.txt
