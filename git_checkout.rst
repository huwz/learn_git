git checkout
============

指令简介
--------

更新工作区中的文件，与索引树中指定版本或者指定文件保持一致。
未指明路径，则 ``git checkout`` 会更新 HEAD 值（保存在 HEAD 文件中）。

.. note:: HEAD 指向当前分支。

常用示例
--------

1. **git checkout <existed_branch>**
  
   准备在 <existed_branch> 分支上工作。
   git 切换到 <existed_branch> 分支的工作区，包括更新 index 和工作区中的其它文件，将 HEAD 指向 <existed_branch>。
   工作区中本地的修改不会丢失，因此可以继续在 <existed_branch> 分支上提交。

   .. note:: 必须提交到索引区中，否则会 checkout 失败。
   参考 :ref:`git_add`。

   举个例子：
  
   .. code-block:: text
  
    branch master:
     echo "hello" > 1.txt
     git add 1.txt
     git checkout modify
    branch modify:
     git commit -m "add file"

   在上面的例子中，主分支没有提交索引区内容，而是由 modify 分支提交。
   切回主分支后发现 ``git log`` 并没有打印刚刚的提交日志。
   说明主分支已经落后 modify 分支一次提交，可以通过 ``git merge modify`` 指令实现同步。

2. **git checkout <non-existed_branch>**

   若远端不存在 ``non-existed_branch`` 分支，则指令执行失败。

3. **git checkout**
   
   git 打印当前分支对远端分支的跟踪信息。
   如果没有跟踪信息，则什么也不做。

4. **git checkout -b|-B <branch> [<start-point>]**
   
   <start-point> 指定 checkout 的分支。
   省略时，表示从当前分支 checkout。
  
   * ``-b``
    
     新建 <branch> 分支，并 checkout 指定分支。

   * ``-B``
    
     如果 ``<branch>`` 不存在，等同于 ``-b``；否则用指定分支替换 ``<branch>``。

     举例：

     * ``git checkout -B master modify`` 将 master 用 modify 替换
     * ``git checkout -B a b`` 如果 a 存在，则用 b 替换 a；否则新建 a，并 checkout b

     重置只替换工作区和版本库，不会影响索引区。
     所以重置后，索引区的内容还可以提交。

关于自动追踪
------------

自动追踪是指本地分支自动关联到服务器端特定的分支上。
有了这种一一对应的关系，在执行同步操作时，可以简化指令。
例如，将本地分支 test 推送到服务器分支 test，通过 ``git push`` 指令完成：

``git push test:origin/test``

如果本地的 test 自动追踪了服务器的 test，则可以将指令简化为 ``git push``。

如何建立这种自动追踪关系呢？
在新建分支的时候加上 ``--track <remote>`` 选项即可。
该选项会在版本配置文件 config 中增加两项内容：``branch.<name>.remote`` 和 ``branch.<name>.merge``。
在前面的示例中，config 文件新增如下内容：

.. code-block:: text
  
   [branch "test"]
       remote = origin
       merge = refs/heads/test

如果新建 test 分支时没有建立追踪，也可以通过修改配置文件来手动创建：

``git config test branch.test.remote orgin``, ``git config test branch.merge refs/heads/test``

一般情况下，新建分支时，如果远端有同名分支，则会自动创建追踪，不用添加 ``--track`` 选项。
涉及到新建分支的指令有：

* ``git checkout -b|-B``
* ``git checkout <non-existed_branch>``
* ``git branch <branch>``

如果需要指定特定追踪的远端分支，则需要借助 ``--track``。

在没有加 ``--track`` 选项的情况下，如何取消自动追踪呢。
添加全局配置项 ``branch.autosetupmerge``，指令为 ``git config branch.autosetupmerge false``。