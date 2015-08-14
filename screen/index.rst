.. screen documentation master file, created by
   sphinx-quickstart on Sat Feb  7 15:02:39 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. toctree::
   :maxdepth: 2


Screen Usage
====================

        *你是不是经常需要远程登录到Linux服务器？你是不是经常为一些长时间运行的任务头疼？还在用 nohup 吗？那么来看看 screen 吧，它会给你一个惊喜！*

        *Screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen中有会话的概念，用户可以在一个screen会话中创建多个screen窗口，在每一个screen窗口中就像操作一个真实的telnet/SSH连接窗口那样。*

---------------
一、screen功能
---------------

- 会话恢复
- 会话共享
- 多窗口

---------------
二、screen参数
---------------

+ screen -ls  *列出正在运行的screen*
+ screen -S name *启动screen的时候以name作为名称*
+ -d *将指定的screen作业离线（Detach）*
+ screen -r name或pid *进入之前断开的screen*
+ screen -d -r name *强抢一个已经存在的screen*
+ screen -x name *进入没有断开的screen，这样可以让一个人操作，另外一个人可以看到他的全部操作*

-------------------------
三、screen 多窗口管理
-------------------------

    每个screen的session中，所有命令都是ctrl+a开头
    
- C-a  c     ==>  在当前screen中创建一个新的shell窗口
- C-a  n     ==>  切换到下一个window
- C-a  p     ==>  切换到上一个window
- C-a  0...9 ==>  切换到第0...9个window
- C-a  [space] ==>  由第0个window循环切换到第9个window
- C-a  C-a   ==>  在两个最近使用的window之间切换
- C-a  x   ==>  锁住当前window，需要密码解锁
- C-a  d   ==>  dettach，离开当前session
- C-a  w  ==>  显示所有窗口列表
- C-a  t   ==>  显示当前时间以及系统load
- C-a  k   ==>  强制关闭当前window


- C-a  S   ==>  水平分屏
- C-a  [TAB]  ==>  下一屏，分屏后需要C-a c 新建窗口后方可使用

-------------------
Indices and tables
-------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

