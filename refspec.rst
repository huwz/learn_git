Refspec
=======

.. note:: 本章是对《progit-en.pdf》中的 ``The Refspec`` 章节的翻译

纵观本书，我们已经用了好几个简单映射的例子，即将远端分支映射到本地。
其实这种映射可以更复杂。
假定你已经添加了一个服务器远端（实质是在 ``refs/`` 下增加一个文件夹）：

``git remote add origin https://github.com/schacon/simplegit-progit``

该指令在配置文件 ``.git/config `` 中添加一个节点。
指明服务器远端的名称（``origin``），远程仓库的 URL 以及拉取的 ``refspec``：

.. code-block:: text

    [remote "origin"]
        url = https://github.com/schacon/simplegit-progit
        fetch = +refs/heads/*:refs/remotes/origin/*

``refspec`` 的格式为一个可选的 ``+`` 号，后面跟上 ``<src>:<dst>``。
其中 <src> 表示远端引用 [1]_ 的文件模式 [2]_，<dst> 表示本地存储远端引用的文件模式。
``+`` 要求 Git 对非快速前移 [3]_ 同样执行拉取并更新引用。

.. note:: 拉取是指下载远端分支，并合并到本地分支

默认情况下，添加远端是通过指令 ``git remote add`` 完成的（没有任何参数）。
Git 会拉取服务器的 ``refs/heads/`` 目录下的所有引用，
并将它们写入到本地的 ``refs/remotes/origin/`` 下对应的引用文件中。
因此，如果服务器上有 master 分支，可以通过以下指令获取该分支的提交日志：

.. code-block:: text

    git log origin/master
    git log remotes/origin/master
    git log refs/remotes/origin/master

这些指令完全等价，因为 Git 会将它们都扩展为 ``refs/remotes/origin/master``。

.. note:: 题外话

 progit-en.pdf 中提到分支和引用的概念，
 两者是有区别的：
 分支是工作的支线任务，它是完整的 Git 工程。
 它可以用一个字符名称来表示，比如 master。
 引用是每次提交生成的一个唯一 ID。
 引用和分支的关系是前者是后者的指针。
 每一次引用都指向某个历史上的分支。
 头指针其实也是引用。
 我们在 Git 指令中使用分支名，其实最终会转化为分支的引用。

如果你想让 Git 只拉取主分支，不包含服务器其它分支，可以将 ``fetch`` 项修改为：

``fetch = +refs/heads/master:refs/remotes/origin/master``

这是远端默认的 ``refspec`` 值。
如果某个时刻你想做某件事，你也可以通过命令行指定 ``refspec``。
例如将远端的主分支拉取到本地的 ``origin/mymaster`` 引用上，
你可以执行指令：

``git fetch origin master:refs/remotes/origin/mymaster``

.. note:: 这个指令表示你可以修改主分支的名称

你可以在命令行中指定多个 ``refspecs``，用于拉取多个分支：

``git fetch origin master:refs/remotes/origin/mymaster \
     topic:refs/remotes/origin/topic``

在这个例子中，主分支 mymaster 的拉取操作被拒绝了，因为它不是一个快速前移的引用。
你可以在 ``refspec`` 加上 ``+`` 防止出现这个问题。

你也可以在配置文件中指定多个 ``refspecs``。
如果要求总是拉取主分支和 ``experiment`` 分支，可以添加以下两行：

.. code-block:: text

    [remote "origin"]
        url = https://github.com/schacon/simplegit-progit
        fetch = +refs/heads/master:refs/remotes/origin/master
        fetch = +refs/heads/experiment:refs/remotes/origin/experiment

在匹配模式中，不能使用部分匹配（glob），因此以下设置无效：

``fetch = +refs/heads/qa*:refs/remotes/origin/qa*

但是，可以用命名空间（或者目录）实现这样的效果。
如果你有一个 ``QA`` 团队将一系列分支上传到了远端。
你只想获取主分支和 ``QA`` 分支，你可以在配置文件中这样设置：

.. code-block:: text

    [remote "origin"]
        url = https://github.com/schacon/simplegit-progit
        fetch = +refs/heads/master:refs/remotes/origin/master
        fetch = +refs/heads/qa/*:refs/remotes/origin/qa/*    

如果你需要处理复杂的工作流程：
QA 团队，开发人员上传分支，项目整合团队进行上传和联调。
你可以分别建立命名空间简单实现。

推送引用
--------

你可以使用上文提到的方式获取命名空间中的引用。
但是 QA 团队如何优先获取 ``qa`` 空间中的分支呢？
可以设置 ``push`` 的 ``refspecs`` 实现这个目标。

QA 团队要将他们的主分支 master 推送到远端的 ``qa/master`` 上，可以执行：

``git push origin master:refs/heads/qa/master``

要想在执行 ``git push origin`` 时，让 Git 自动以上指令，可以在配置文件中增加一个 ``push``:

.. code-block:: text

    [remote "origin"]
        url = https://github.com/schacon/simplegit-progit
        fetch = +refs/heads/master:refs/remotes/origin/master
        push = refs/heads/master:refs/remotes/origin/qa/master    

这样设置之后，在使用 ``git push origin`` 时，会自动将本地的 master 分支推送到远端的 ``qa/master`` 分支。

删除引用
--------

你也可以使用 ``refspec`` 删除远端的引用：

``git push origin :topic``

``refspec`` 的格式是 ``<src>:<dst>``。
省略 <src> 部分，主要是让远端的 ``topic`` 分支消失，即删除。

.. [1] 这里的引用是指分支的引用文件。
 例如分支 ``test`` 会在 ``.git/refs/heads/`` 中生成一个 test 文件。
 test 文件存储的是该分支的最近一次提交 ID 号。
 通过该 ID， 在索引区（``index`` 文件）中找到对应的工作区快照（或修改内容）。
 应用快照内容，即可将本地工作区恢复到 ID 号对应的版本。
 这也是为什么进行 checkout 的时候，所有分支可以各自拥有不同的工作进度。
.. [2] 文件模式指文件名匹配模式，使用通配符可以匹配多个文件。
.. [3] 快速前移操作是通过 ``git merge`` 完成的。
 举个例子，master 分支和 test 分支进度一致。
 在 test 分支上做一次新的提交，再切换到 master 分支。
 master 分支已经落后 test 分支一次提交。
 如果不对 master 做任何修改，直接合并 test 分支，
 Git 只要向前移动头指针，指向 test 新的提交即可。