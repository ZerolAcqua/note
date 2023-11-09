---

---

# Progit

## 参考资料
- [pcottle/learnGitBranching: An interactive git visualization and tutorial. Aspiring students of git can use this app to educate and challenge themselves towards mastery of git! (github.com)](https://github.com/pcottle/learnGitBranching)

- [Progit 下载链接](https://github.com/progit/progit2-zh/releases/download/2.1.62/progit.pdf)

### 第一章 起步
#### 版本控制
- 本地版本控制系统
- 集中化的版本控制系统
- 分布式版本控制系统
#### Git 简史
#### Git 是什么？
- 直接记录快照，而非差异比较
    - Git 更像是把数据看作是对小型文件系统的一系列快照
- 近乎所有操作都是本地执行
    - 在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息
- Git 保证完整性
    - 不可能在 Git 不知情时更改任何文件内容或目录内容
- Git 一般只添加数据
    - 很难使用 Git 从数据库中删除数据
- 三种状态
    - **已修改**表示修改了文件，但还没保存到数据库中。 
    - **已暂存**表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
    - **已提交**表示数据已经安全地保存在本地数据库中。
#### 安装与配置
- Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置。
    - **system**  /etc/gitconfig
    - **global**  ~/.gitconfig
    - **local**  /.git/config
- 每一个级别会覆盖上一级别的配置
- 可以使用相关命令查询所有相关配置以及令配置生效的配置文件
#### 获取帮助
### 第二章 Git 基础
#### 获取 Git 仓库
- 将尚未进行版本控制的本地目录转换为 Git 仓库
    - 进入项目目录，之后执行初始化 git init
    - 如果目录下已存在文件应该开始追踪这些文件并进行初始提交
        - git add 进行追踪
        - git commit 进行初始提交
- 从其它服务器克隆一个已存在的 Git 仓库。
    - git clone [url]
    - Git 支持多种数据传输协议
#### 记录每次更新到仓库
- 检查当前文件状态
    - git status 查看哪些文件处于什么状态
- 跟踪新文件
    - git add 以跟踪一个新文件
- 暂存已修改的文件
    - git add 把已跟踪的文件放到暂存区
- 状态简览
    - git status -s 或 git status --short
- 忽略文件
    - .gitignore 的文件，用于忽略文件
        
        [Git 忽略文件.gitignore 详解_ThinkWon 的博客-CSDN 博客_gitignore](https://blog.csdn.net/ThinkWon/article/details/101447866)

        - 所有空行或者以 # 开头的行都会被 Git 忽略。
        - 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
            - 星号（*）匹配零个或多个任意字符
            - [abc] 匹配任何一个列在方括号中的字符
                - 这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c
            - 问号（?）只匹配一个任意字符
            - 如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配。
                - 比如 [0-9] 表示匹配所有 0 到 9 的数字
            - 使用两个星号（\*\*）表示匹配任意中间目录。
                比如 a/\*\*/z 可以 匹配 a/z 、 a/b/z 或 a/b/c/z 等
        - 匹配模式可以以（/）开头（中间亦可）防止递归。
        - 匹配模式可以以（/）结尾指定目录。
        - 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。

    - git check-ignore 可以检查是哪行忽略了文件

- 查看已暂存和未暂存的修改
    - git diff 不加参数查看尚未暂存的文件更新了哪些部分
    - git diff --staged 或 git diff --cached 查看比对已暂存文件与最后一次提交的文件差异
- 提交更新
    - git commit 提交
    - git commit -a 跳过使用暂存区域进行提交
- 移除文件
    - git rm 从暂存区域移除并连带从工作目录中删除指定的文件。如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 -f
    - 另外一种情况是，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 --cached 选项
- 移动文件
    - 要在 Git 中对文件改名，可以使用 git mv file_from file_to
#### 查看提交历史
- git log 回顾提交历史
    - -p 或 --patch 选项用于显示每次提交所引入的差异
    - -n 选项来只显示最近的 n 次提交
    - --stat 选项用于显示每次提交的简略统计信息
    - --pretty 选项以指定不同于默认格式的方式展示提交历史
    - 当 oneline 或 format 与另一个 log 选项 --graph 结合使用时尤其有用。 这个选项添加了一些 ASCII 字符串来形象地展示你的分支、合并历史
- 限制输出长度
    - --since 按时间限制
    - --author 按作者限制
    - --grep 按提交说明中的关键字限制
    - -S 按那些添加或删除了该字符串的提交限制
    - --no-merges 过滤合并提交
#### 撤消操作
- git commit --amend 这个命令会将暂存区中的文件提交。如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你所修改的只是提交信息。
> 当你在修补最后的提交时，与其说是修复旧提交，倒不如说是完全用一个新的提交替换旧的提交
    - git commit -m 'initial commit'
    - git add forgotten_file
    - git commit --amend
    - 最终你只会有一个提交——第二次提交将代替第一次提交的结果
- 取消暂存的文件
    - git reset HEAD [file] 取消暂存
- 撤消对文件的修改
    - git checkout -- [file] 撤消之前所做的修改
    > 工作区的相应文件会被最近的提交覆盖
#### 远程仓库的使用
- 查看远程仓库
    - git remote 它会列出你指定的每一个远程服务器的简写
        - 可以指定选项 -v，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL
- 添加远程仓库
    - git remote add [shortname] [url] 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写
    - git fetch pb
- 推送到远程仓库
    - git push [remote] [branch] 
        - 只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。否则，你必须先抓取他们的工作并将其合并进你的工作后才能推送。
- 查看某个远程仓库
    - git remote show [remote]
#### 远程仓库的重命名与移除
- git remote rename 修改一个远程仓库的简写名
- git remote remove 或 git remote rm 移除一个远程仓库
#### 打标签
- 列出标签
    - git tag 列出已有的标签
        - -l 或 --list 可以使用通配方式进行筛选
- 创建附注标签
    - git tag -a [tag] -m [message] 
        - 可使用 git show [tag] 看到标签信息和与之对应的提交信息。
- 创建轻量标签
    - git tag [tag]
        - 使用 git show [tag] 不会看到额外的标签信息。只会显示出提交信息。
- 后期打标签
    - git tag -a [tag] [checksum] 可以在对应的提交上打上标签
- 共享标签
    - git push origin [tagname] 以显式地推送标签到共享服务器上
        - git push origin --tags 则可以把所有不在远程仓库服务器上的标签全部传送到那里
- 删除标签
    - git tag -d [tagname] 删除本地仓库的标签
    - 同步更新远程仓库有两种方法
        - git push [remote] :refs/tags/[tagname]  
        - git push origin --delete [tagname] 
- 检出标签
    - git checkout 
        - 会使你的仓库处于分离头指针（detached HEAD）的状态
            - 在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。 
            - 因此，如果你需要进行更改，比如你要修复旧版本中的错误，那么通常需要创建一个新分支。
#### Git 别名
- 可以通过 git config 文件来轻松地为每一个命令设置一个别名
    - git config --global alias.ci commit
        - git ci 会等价于 git commit
    - git config --global alias.last 'log -1 HEAD'
        - git last 可以查看最后一次提交
- 在命令前面加入 ! 号，可执行外部命令
    - git config --global alias.visual '!gitk'
### 第三章 Git 分支
#### 分支简介
- 文件用 blob 对象保存
- 这些子文件 blob 的校验和保存为树对象
- 提交对象则保存树对象的指针、父对象指针，以及提交信息
> 我其实很早之前就想过这个问题了，每次提交都需要存储一份快照，会不会存储空间不够呢？下面是问题的回答：[git 每一次暂存都会保存一次快照吗？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/282558019)
- 分支创建
    - git branch testing 创建一个 testing 分支
    - 分支创建是通过创建一个分支指针实现的，而判断当前位于哪个分支是利用 Head 这一特殊指针完成的
    - git log --oneline --decorate 可查看各分支所指的对象
- 分支切换
    - git checkout testing 切换到 testing 分支
        - 在切换分支时，工作目录里的文件会被改变。如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。 如果 Git 不能干净利落地完成这个任务，它将禁止切换分支。
        - git checkout -b [newbranchname] 创建新分支的同时切换过去
#### 分支的新建与合并
- 新建分支
    - git branch testing 创建一个 testing 分支
    - git checkout testing 切换到 testing 分支
    - 在切换分支前保持好一个干净的状态，能让你顺利地进行分支切换
        - 使用暂存（stashing）和修补提交（commit amending）可以绕过这个问题
        - 合并分支：快进
            - git checkout master
            - git merge hotfix
            - 合并的分支 hotfix 是 master 的直接后继，则合并时只会简单地移动指针，即快进
        - 删除分支
            - git branch -d hotfix
- 分支的合并
    - 合并分支：三方合并
        - 开发历史从一个更早的地方开始分叉开来（diverged）。master 分支所在提交并不是 iss53 分支所在提交的直接祖先
        - 三方合并会创建一个新的提交——合并提交
            - git checkout master
            - git merge iss53
        - 合并后就可以删掉分支了
            - git branch -d iss53
- 遇到冲突时的分支合并
    - 如果在两个不同的分支中，同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们
    - 合并冲突后，使用 git status 命令来查看那些因包含合并冲突而处于未合并 （unmerged）状态的文件
    - 在解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决。
    - git status 以确认所有的合并冲突都已被解决
    - git commit 来完成合并提交
#### 分支管理
- git branch 直接使用可显示所有分支的列表
    - --merged 和 --no-merged 选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支
    - 提供一个附加的参数来查看其它分支的合并状态而不必检出它们，如
        - git branch --no-merged master
#### 分支开发工作流
- 长期分支：流水线式
    - master
    - develop/next
    - topic
    - proposed
- 主题分支：注重特定主题
#### 远程分支
- 远程引用是对远程仓库的引用（指针），包括分支、标签等等。
    - git ls-remote [remote] 获得远程引用的完整列表
    - git remote show [remote] 获得远程分支的更多信息
- 与给定的远程仓库同步数据
    - git fetch [remote]
    - 本地与远程的工作可以分叉
    - 也可以添加另一个远程仓库
- 推送
    - git push [remote] [branch]
        - 下一次其他协作者从服务器上抓取数据时，他们会在本地生成一个远程分支，但是本地不会自动生成一份可编辑的副本（拷贝）
        - git merge origin/serverfix 将这些工作合并到当前所在的分支
        - git checkout -b serverfix origin/serverfix 可以将其建立在远程跟踪分支之上，以在其他人的本地分支工作
- 跟踪分支
    - 当克隆一个仓库时，它通常会自动地创建一个跟踪 origin/master 的 master 分支
    - 运行 git checkout -b [branch] [remote]/[branch] 进行跟踪
        - 或是简化方式 git checkout --track origin/serverfix
        - 使用 -u 或 --set-upstream-to 选项运行 git branch 来显式地设置上游分支
        - 设置好跟踪分支后，可以通过简写@{upstream}或@{u]来引用它的上游分支
            - git merge @{u} 来取代 git merge origin/master
        - git branch 的 -vv 选项用于查看设置的所有跟踪分支
            - 需要重点注意的一点是这些数字的值来自于你从每个服务器上最后一次抓取的数据
            - git fetch --all; git branch -vv 可获取最新的
- 拉取
    - git pull 在大多数情况下它的含义是一个 git fetch 紧接着一个 git merge 命令
- 删除远程分支
    - 使用带有 --delete 选项的 git push 命令来删除一个远程分支
        - git push origin --delete serverfix
        - 基本上这个命令做的只是从服务器上移除这个指针。 Git 服务器通常会保留数据一段时间直到垃圾回收运行，所 以如果不小心删除掉了，通常是很容易恢复的。
#### 变基
- 变基的基本操作
    - 可以使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。
    - 可以检出 experiment 分支，然后将它变基到 master 分支上
        - git checkout experiment
        - git rebase master
    - 再回到 master 分支，进行一次快进合并
        - git checkout master
        - git merge experiment
    - 这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁
    - 这样的话，该项目的维护者就不再需要进行整合工作
- 更有趣的变基例子
    - 在对两个分支进行变基时，所生成的“重放”并不一定要在目标分支上应用
        - git rebase --onto master server client
            - 取出 client 分支，找出它从 server 分支分歧之后的补丁， 然后把这些补丁在 master 分支上重放一遍，让 client 看起来像直接基于 master 修改一样
- 变基的风险
    - <u>如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。</u>
- 用变基解决变基
    - 如果团队中的某人强制推送并覆盖了一些 你所基于的提交，你需要做的就是检查你做了哪些修改，以及他们覆盖了哪些修改
        - git rebase teamone/master 会：
            - 检查哪些提交是我们的分支上独有的
            - 检查其中哪些提交不是合并操作的结果
            - 检查哪些提交在对方覆盖更新时并没有被纳入目标分支
            - 把查到的这些提交应用在 teamone/master 上面
        - 要想上述方案有效，还需要对方在变基时确保 C4' 和 C4 是几乎一样的。 
        - 否则变基操作将无法识别，并新建另一个类似 C4 的补丁
    - 如果你只对不会离开你电脑的提交执行变基，那就不会有事。 
    - 如果你对已经推送过的提交执行变基，但别人没有基于它的提交，那么也不会有事。 
    - 如果你对已经推送至共用仓库的提交上执行变基命令，并因此丢失了一些 别人的开发所基于的提交， 那你就有大麻烦了，你的同事也会因此鄙视你。
    - 如果你或你的同事在某些情形下决意要这么做，请一定要通知每个人执行 git pull --rebase 命令，这样尽管不能避免伤痛，但能有所缓解。
- 变基 vs. 合并
    - 总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。
### 第四章 服务器上的 Git
一个远程仓库通常只是一个裸仓库（bare repository）——即一个没有当前工作目录的仓库。因为该仓库仅仅作为合作媒介，不需要从磁盘检查快照；存放的只有 Git 的资料。简单的说，裸仓库就是你工程目录内的 .git 子目录内容，不包含其他资料。
#### 协议
- Git 可以使用四种不同的协议来传输资料：本地协议（Local），HTTP 协议，SSH（Secure Shell）协议及 Git 协议
- 本地协议
    - 最基本的就是本地协议（Local protocol），其中的远程版本库就是同一主机上的另一个目录
    - 如果使用共享文件系统，就可以从本地版本库克隆（clone）、推送（push）以及拉取（pull）。像这样去克隆一个版本库或者增加一个远程到现有的项目中，使用版本库路径作为 URL。
    - git clone /srv/git/project.git
        - 指定路径，Git 尝试使用硬链接（hard link）或直接复制所需要的文件
    - git clone file:///srv/git/project.git
        - Git 会触发平时用于网路传输资料的进程，以取得一个没有外部参考（extraneous references）或对象 （object）的干净版本库副本
    - 要增加一个本地版本库到现有的 Git 项目，可以执行
        - git remote add local_proj /srv/git/project.git
    - 本地协议的优缺点
        - 简单，并且直接使用了现有的文件权限和网络访问权限。如果你的团队已经有共享文件系统，建立版本库会十分容易。
        - 共享文件系统比较难配置，并且比起基本的网络连接访问，这不方便从多个位置访问。如果你想从家里推送内容，必须先挂载一个远程磁盘，相比网络连接的访问方式，配置不方便，速度也慢。
        - 这个协议并不保护仓库避免意外的损坏。
- HTTP 协议
    - 新版本的 HTTP 协议一般被称为智能 HTTP 协议， 旧版本的一般被称为哑 HTTP 协议。我们先了解一下新的智能 HTTP 协议
    - 智能 HTTP 协议
        - 智能 HTTP 的运行方式和 SSH 及 Git 协议类似，只是运行在标准的 HTTP/S 端口上并且可以使用各种 HTTP 验证机制， 这意味着使用起来会比 SSH 协议简单的多，比如可以使用 HTTP 协议的用户名/密码授权，免去设置 SSH 公钥。
    - 哑（Dumb）HTTP 协议
        - 如果服务器没有提供智能 HTTP 协议的服务，Git 客户端会尝试使用更简单的“哑” HTTP 协议。
        - 哑 HTTP 协议里 web 服务器仅把裸版本库当作普通文件来对待，提供文件服务。
        - 基本上，只需要把一个裸版本库放在 HTTP 根目录，设置一个叫做 post-update 的挂钩就可以了
    - 优点
        - 不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息，这两个简便性让终端用户使用 Git 变得非常简单。
        - 相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用 Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。
        - HTTPS 协议被广泛使用
    - 缺点
        - 在一些服务器上，架设 HTTPS 协议的服务端会比 SSH 协议的棘手一些。
- SSH 协议
    - 架设 Git 服务器时常用 SSH 协议作为传输协议。因为大多数环境下服务器已经支持通过 SSH 访问 
    - 即使没有支持也很容易架设。SSH 协议也是一个验证授权的网络协议
    - 因为其普遍性，架设和使用都很容易。
    - git clone ssh://[user@]server/project.git
    - git clone [user@]server:project.git
    - 优点
        - SSH 架设相对简单
        - 通过 SSH 访问是安全的
        - SSH 协议很高效
    - 缺点
        - SSH 协议的缺点在于它不支持匿名访问 Git 仓库。如果你使用 SSH，那么即便只是读取数据，使用者也必须通过 SSH 访问你的主机
- Git 协议
    - 它监听在一个特定的端口（9418），类似于 SSH 服务，但是访问无需任何授权
    - 优点
        - Git 协议是 Git 使用的网络传输协议里最快的。如果你的项目有很大的访问量，或者你的项目很庞大并且 不需要为写进行用户授权，架设 Git 守护进程来提供服务是不错的选择。
    - 缺点
        - Git 协议缺点是缺乏授权机制
        - Git 协议也许也是最难架设的
        - 它还要求防火墙开放 9418 端口
#### 在服务器上搭建 Git
- 在开始架设 Git 服务器前，需要把现有仓库导出为裸仓库——即一个不包含当前工作目录的仓库。这通常是很简单的。为了通过克隆你的仓库来创建一个新的裸仓库，你需要在克隆命令后加上 --bare 选项。按照惯例，裸仓库的目录名以 .git 结尾
    - git clone --bare my_project my_project.git
    - 整体上相当于 cp -Rf my_project/.git my_project.git
- 把裸仓库放到服务器上
    - 假设一个域名为 git.example.com 的服务器已经架设好，并可以通过 SSH 连接， 你想把所有的 Git 仓库放在 /srv/git 目录下。假设服务器上存在 /srv/git/ 目录，你可以通过以下命令复制你的裸仓库来创建一个新仓库
        - scp -r my_project.git user@git.example.com:/srv/git
    - 此时，其他可通过 SSH 读取此服务器上 /srv/git 目录的用户，可运行以下命令来克隆你的仓库。
        - git clone user@git.example.com:/srv/git/my_project.git
    - 如果一个用户，通过使用 SSH 连接到一个服务器，并且其对 /srv/git/my_project.git 目录拥有可写权限，那么他将自动拥有推送权限。
    - 到该项目目录中运行 git init 命令，并加上 --shared 选项， 那么 Git 会自动修改该仓库目录的组权限为可写。注意，运行此命令的工程中不会摧毁任何提交、引用等内容。
        - ssh user@git.example.com
        - cd /srv/git/my_project.git
        - git init --bare --shared
- 小型安装
    - 如果设备较少或者你只想在小型开发团队里尝试 Git ，那么一切都很简单。 架设 Git 服务最复杂的地方在于用户管理。如果需要仓库对特定的用户可读，而给另一部分用户读写权限，那么访问和许可安排就会比较困难。
    - SSH 安装
        - 如果你有一台所有开发者都可以用 SSH 连接的服务器，架设你的第一个仓库就十分简单了， 因为你几乎什么都不用做（正如我们上一节所说的）。如果你想在你的仓库上设置更复杂的访问控制权限，只要使用服务器操作 系统的普通的文件系统权限就行了。
        - 如果需要团队里的每个人都对仓库有写权限，又不能给每个人在服务器上建立账户，那么提供 SSH 连接就是唯一的选择了。
            - 给团队里的每个人创建账号
            - 在主机上建立一个 git 账户
            - 让 SSH 服务器通过某个 LDAP 服务，或者其他已经设定好的集中授权机制，来进行授权
#### 生成 SSH 公钥
- 默认情况下，用户的 SSH 密钥存储在其 ~/.ssh 目录下
- 我们需要寻找一对以 id_dsa 或 id_rsa 命名的文件，其中一个带有 .pub 扩展名。.pub 文件是你的公钥，另一个则是与之对应的私钥。如果找不到这样的文件（或者根本没有 .ssh 目录），你可以通过运行 ssh-keygen 程序来创建它们。在 Linux/macOS 系统中，ssh-keygen 随 SSH 软件包提供；在 Windows 上，该程序包含于 MSysGit 软件包中
#### 配置服务器
- 使用 authorized_keys 方法来对用户进行认证
    - 创建一个操作系统用户 git，并为其建立一个 .ssh 目录
        - sudo adduser git
        - su git
        - cd
        - mkdir .ssh
        - chmod 700 .ssh
        - touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
    - 为系统用户 git 的 authorized_keys 文件添加一些开发者 SSH 公钥，并将它们保存在临时文件中
        - cat /tmp/id_rsa.john.pub
        
    - 将这些公钥加入系统用户 git 的 .ssh 目录下 authorized_keys 文件的末尾
        - cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
        - cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
        - cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys
    - 为开发者新建一个空仓库。使用带 --bare 选项的 git init 命令
        - cd /srv/git
        - mkdir project.git
        - cd project.git
        - git init --bare
        > Initialized empty Git repository in /srv/git/project.git/
    - 接着，John、Josie 或者 Jessica 中的任意一人可以将他们项目的最初版本推送到这个仓库中
- 通过这种方法，你可以快速搭建一个具有读写权限、面向多个开发者的 Git 服务器。
- 需要注意的是，目前所有（获得授权的）开发者用户都能以系统用户 git 的身份登录服务器从而获得一个普通 shell。如果你想对此加以限制，则需要修改 /etc/passwd 文件中（git 用户所对应）的 shell 值。
- 借助一个名为 git-shell 的受限 shell 工具，你可以方便地将用户 git 的活动限制在与 Git 相关的范围内。该工具随 Git 软件包一同提供。如果将 git-shell 设置为用户 git 的登录 shell（login shell），那么该用户便不能获得此服务器的普通 shell 访问权限。若要使用 git-shell，需要用它替换掉 bash 或 csh，使其成为该用户的登录 shell。
    - cat /etc/shells # see if git-shell is already in there. If not...
    - which git-shell # make sure git-shell is installed on your system.
    - sudo -e /etc/shells # and add the path to git-shell from last command
    - 然后使用 chsh [username] -s [shell] 命令修改任一系统用户的 shell
        - sudo chsh git -s $(which git-shell)
    - 可编辑 authorized_keys 文件并在所有想要限制的公钥之前添加以下选项
        - no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty

#### Git 守护进程
#### Smart HTTP
#### GitWeb
- 如果你对项目有读写权限或只读权限，你可能需要建立起一个基于网页的简易查看器。 Git 提供了一个叫做 GitWeb 的 CGI 脚本来做这项工作
#### GitLab
- 如果你正在寻找一个更现代，功能更全的 Git 服务器，这里有几个开源的解决方案可供你选择安装。因为 GitLab 是其中最出名的一个，我们将它作为示例并讨论它的安装和使用。这比 GitWeb 要复杂的多并且需要更多的维护，但它的确是一个功能更全的选择。
#### 第三方托管的选择
- 如果不想设立自己的 Git 服务器，你可以选择将你的 Git 项目托管到一个外部专业的托管网站。这带来了一些好处：一个托管网站可以用来快速建立并开始项目，且无需进行服务器维护和监控工作。即使你在内部设立并且运行了自己的服务器，你仍然可以把你的开源代码托管在公共托管网站——这通常更有助于开源社区来发现和帮助你。
- 现在，有非常多的托管供你选择，每个选择都有不同的优缺点。 欲查看最新列表，请浏览 Git 维基的 GitHosting 页面 https://git.wiki.kernel.org/index.php/GitHosting
#### 第五章 分布式 Git
####  分布式工作流程
- 集中式工作流
- 集成管理者工作流
- 主管与副主管工作流
#### 向一个项目贡献
- 影响因素：活跃贡献者的数量、项目使用的工作流程、提交权限
- 提交准则
    - 提交不应该包含任何空白错误
        - git diff --check
    - 尝试让每一个提交成为一个逻辑上的独立变更集
        - 使用暂存区域将你的工作最少拆分为每个问题一个提交，并且为每一个提交附带一个有用的信息
    - 如果其中一些改动修改了同一个文件，尝试使用 git add --patch 来部分暂存文件
    - 有一个创建优质提交信息的习惯会使 Git 的使用与协作容易的多
- 私有小型团队
    - 可以采用一个类似使用 Subversion 或其他集中式的系统时会使用的工作流程。依然可以得到像 离线提交、非常容易地新建分支与合并分支等高级功能，但是工作流程可以是很简单的；主要的区别是合并发生在客户端这边而不是在提交时发生在服务器那边。
- 私有管理团队
    - 在接下来的场景中，你会看到大型私有团队中贡献者的角色。你将学到如何在这种工作环境中工作，其中小组基于特性进行协作，而这些团队的贡献将会由其他人整合。
- 派生的公开项目
    - --squash 选项接受被合并的分支上的所有工作，并将其压缩至一个变更集， 使仓库变成一个真正的合并发生的状态，而不会真的生成一个合并提交。
- 通过邮件的公开项目 
#### 维护项目
- 在主题分支中工作
    - 如果你想向项目中整合一些新东西，最好将这些尝试局限在主题分支。项目的维护者一般还会为这些分支附带命名空间
        - git branch sc/ruby_client master
- 应用来自邮件的补丁 
- 检出远程分支
    - 可以将其添加为一个远程分支，并在本地检出
    - Git 提供了一种比较便捷的方式：三点语法。以对该分支的最新提交与两个分支的共同祖先进行比较
        - git diff master...contrib
- 将贡献的工作整合进来
    - 合并工作流
        - 将所有的工作直接合并到 master 分支
        - 两阶段合并循环
    - 大项目合并工作流
        - Git 项目包含四个长期分支：master、next，用于新工作的 pu（proposed updates）和用于维护性向后移植工作（maintenance backports）的 maint 分支。
    - 变基与拣选工作流
    - Rerere
    - 为发布打标签
    - 生成一个构建号
        - Git 中不存在随每次提交递增的“v123”之类的数字序列，如果你想要为提交附上一个可读的名称， 可以对其运行 git describe 命令。
            - 默认情况下，git describe 命令需要有注解的标签（即使用 -a 或 -s 选项创建的标签）
            - 如果你想使用轻量标签（无注解的标签），请在命令后添加 --tags 选项。
    - 准备一次发布
        - 创建一个最新的快照归档
            - 使用 git archive 命令完成此工作
            - 也可以以类似的方式创建一个 zip 压缩包，但此时你应该向 git archive 命令传递 --format=zip
            - 使用 git shortlog 命令可以快速生成一 份包含从上次发布之后项目新增内容的修改日志（changelog）类文档
### 第六章 Github
#### 账户的创建和配置 
#### 对项目做出贡献
- 派生项目 
- GitHub 流程
    - 派生一个项目
    - 从 master 分支创建一个新分支
    - 提交一些修改来改进项目
    - 将这个分支推送到 GitHub 上
    - 创建一个拉取请求
    - 讨论，根据实际情况继续修改
    - 项目的拥有者合并或关闭你的拉取请求
    - 将更新后的 master 分支同步到你的派生中
- 拉取请求的进阶用法
- GitHub 风格的 Markdown
- 让你的 GitHub 公共仓库保持更新
#### 维护项目
- 创建新的版本库
- 添加合作者
- 管理合并请求
- 在合并请求上进行合作
- 合并请求引用
- 提醒和通知 
- 特殊文件
    - README
    - CONTRIBUTING
- 项目管理
    - 改变默认分支
    - 移交项目
#### 管理组织
- 组织的基本知识
- 团队
- 审计日志
#### 脚本 GitHub
- 服务与钩子
    - 服务
    - 钩子
- GitHub API
    - 基本用途
    - 在一个问题上评论
    - 修改 Pull Request 的状态
    - Octokit
### 第七章 Git 工具
#### 选择修订版本
- 单个修订版本
    - 简短的 SHA-1
        - git show [SHA-1]
        - SHA-1 碰撞的几率非常小
    - 分支引用
        - 使用名叫 rev-parse 的 Git 探测工具，获取分支名称指向的提交的 SHA-1
            - git rev-parse topic1
    - 引用日志
        - 使用 git reflog 来查看引用日志
        - 查看仓库中 HEAD 在 第 n 次前的所指向的提交
            - git show HEAD@{n}
        - 查看 master 分支在昨天的时候指向了哪个提交
            - git show master@{yesterday}
        - 运行 git log -g 来查看类似于 git log 输出格式的引用日志信息
        - 引用日志只存在于本地仓库，它只是一个记录你在 自己 的仓库里做过什么的日志
    - 祖先引用
        - 在引用的尾部加上一个 ^（脱字符）， Git 会将其解析为该引用的上一个提交。
        - 在 Windows 的 cmd.exe 中，^ 是一个特殊字符，因此需要区别对待。 你可以双写它或者将提交引用放在引号中
            - git show HEAD^      # 在 Windows 上无法工作
            - git show HEAD^^   # 可以
            - git show "HEAD^"  # 可以
        - 也可以在 ^ 后面添加一个数字来指明想要 哪一个父提交，这个语法只适用于合并的提交。
        - 另一种指明祖先提交的方法是 ~（波浪号）
            - HEAD~2 代表“第一父提交的第一父提交”
                - HEAD~3 与 HEAD\~\~\~

- 提交区间
    - 双点
        - 可以让 Git 选出在一个分支中而不在另一个分支中的提交
            - git log master..experiment
    - 多点
        - 以下三个命令等价
            - git log refA..refB
            - git log ^refA refB
            - git log refB --not refA
        - 所以可以使用多个引用
            - git log refA refB ^refC
            - git log refA refB --not refC
    - 三点
        - 这个语法可以选择出被两个引用 之一 包含但又不被两者同时包含的提交
            - git log master...experiment
                - log 命令的一个常用参数是 --left-right，它会显示每个提交到底处于哪一侧的分支。
                    - git log --left-right master...experiment

#### 交互式暂存 
- 暂存与取消暂存文件
- 暂存补丁
#### 贮藏与清理
- 贮藏工作
    - 想要切换分支，但是还不想要提交之前的工作
        - 运行 git stash 或 git stash push
    - 查看贮藏的东西，可以使用 git stash list
    - 重新应用贮藏的工作
        - git stash apply
        - 文件的改动被重新应用了，但是之前暂存的文件却没有重新暂存。想要那样的话，必须使用 --index 选项来运行 git stash apply 命令，来尝试重新应用暂存的修改。
    - 运行 git stash drop 加上将要移除的贮藏的名字来移除
        - 运行 git stash pop 来应用贮藏然后立即从栈上扔掉它
- 贮藏的创意性使用
    - git stash 命令的 --keep-index 选项，它告诉 Git 不仅要贮藏所有已暂存的内容，同时还要将它们保留在索引中
    - 默认情况下，git stash 只会贮藏已修改和暂存的已跟踪文件。如果指定 --include-untracked 或 -u 选项，Git 也会贮藏任何未跟踪文件。
    - 在贮藏中包含未跟踪的文件仍然不会包含明确忽略的文件。要额外包含忽略的文件，请使用 --all 或 -a 选项
    - 指定了 --patch 标记，Git 不会贮藏所有修改过的任何东西，但是会交互式地提示哪些改动想要贮藏、哪些改动需要保存在工作目录中
- 从贮藏创建一个分支
    - git stash branch [new branchname] 以你指定的分支名创建一个新分支，检出贮藏工作时所在的提交，重新在那应用工作，然后在应用成功后丢弃贮藏
- 清理工作目录
    - git clean
    - 如果只是想要看看它会做什么，可以使用 --dry-run 或 -n 选项来运行命令
    - 如果也想要移除 .gitignore 了的那些文件，例如为了做一次完全干净的构建而移除所有由构建生成的 .o 文件， 可以给 clean 命令增加一个 -x 选项
    - 如果不知道 git clean 命令将会做什么，在将 -n 改为 -f 来真正做之前总是先用 -n 来运行它做双重检查。另一个小心处理过程的方式是使用 -i 或 “interactive” 标记来运行它。
#### 签署工作 
#### 搜索
- git grep 命令
- git log 日志搜索
#### 重写历史
- 修改最后一次提交
    - git commit --amend 修改最近一次提交的提交信息
        - 修改提交的实际内容，只需要补上修改，暂存它们，然后使用此命令
        - 修正会改变提交的 SHA-1 校验和。它类似于一个小的变基
            - 如果已经推送了最后一次提交就不要修正它
- 修改多个提交信息
    - 使用变基工具来变基一系列提交
    - git rebase 增加 -i 选项来交互式地运行变基。 必须指定想要重写多久远的历史，这可以通过告诉命令将要变基到的提交来做到
    - git rebase -i HEAD~3
        - 不要涉及任何已经推送到中央服务器的提交——这样做会产生一次变更的两个版本，因而使他人困惑
        - 交互式变基给你一个它将会运行的脚本，你需要修改脚本来让它停留在你想修改的变更上
- 重新排序提交
    - 修改脚本顺序
- 压缩提交
    - 修改脚本，使用 squash
- 拆分提交
- 核武器级选项：filter-branch
    - git filter-branch 有很多陷阱，不再推荐使用它来重写历史。请考虑使用 git-filter-repo，它是一个 Python 脚本
    - 从每一个提交中移除一个文件
    - 使一个子目录做为新的根目录
    - 全局修改邮箱地址
#### 重置揭密
- 三颗树
    - Git 作为一个系统，是以它的一般操作来管理并操纵这三棵树的
        - HEAD
        - Index
        - Working Directory
    - HEAD
        - HEAD 是当前分支引用的指针，它总是指向该分支上的最后一次提交。这表示 HEAD 将是下一次提交的父结点。通常，理解 HEAD 的最简方式，就是将它看做该分支上的最后一次提交的快照。
        - 查看快照
            - git cat-file -p HEAD
            - git ls-tree -r HEAD
    - 索引
        - 索引是你的预期的下一次提交。我们也会将这个概念引用为 Git 的“暂存区”。
        - Git 将上一次检出到工作目录中的所有文件填充到索引区，它们看起来就像最初被检出时的样子。 之后你会将其中一些文件替换为新版本，接着通过 git commit 将它们转换为树来用作新的提交
        - 查看索引
            - git ls-files -s
    - 工作目录
        - 另外两棵树以一种高效但并不直观的方式，将它们的内容存储在 .git 文件夹中。工作目录会将它们解包为实际的文件以便编辑。你可以把工作目录当做沙盒。在你将修改提交到暂存区并记录到历史之前，可以随意更改。
- 工作流程
- 重置的作用
    - 第一步：移动 HEAD（--soft）
        - 只移动 HEAD 到某个提交
    - 第二步：更新索引（--mixed）
        - 相比上一步，会用 HEAD 内容更新索引区
    - 第三步：更新工作目录（--hard）
        - 相比上一步，还会更新工作区
        - --hard 标记是 reset 命令唯一的危险用法，它也是 Git 会真正地销毁数据的仅有的几个操作之一
        - Git 数据库中的一个提交内还留有该文件的 v3 版本， 我们可以通过 reflog 来找回它
- 通过路径来重置
    - git reset file.txt
        - 相当于取消暂存的作用
- 压缩
    - 使用 reset 来轻松快速地将它们压缩成单个提交
- 检出
    - 不带路径
        - 不同于 reset --hard，checkout 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢
            - checkout 会在工作目录中先试着简单合并一下，这样所有还未修改过的文件都会被更新。而 reset --hard 则会不做检查就全面地替换所有东西
        - checkout 只移动 HEAD ，reset 会同时移动分支指针
    - 带路径
        - git checkout [branch] file，它就像 git reset --hard [branch] file 一样，也会覆盖工作目录中对应的文件，所以不是 WD 安全的
#### 高级合并
- 合并冲突
    - 中断一次合并
        - git merge --abort 选项会尝试恢复到你运行合并前的状态。但当运行命令前，在工作目录中有未储藏、未提交的修改时它不能完美处理
    - 忽略空白
        - -Xignore-all-space 或 -Xignore-space-change 选项。 第一个选项在比较行时完全忽略空白修改，第二个选项将一个空白符与多个连续的空白字符视作等价的
    - 手动文件再合并
        - git show 命令与一个特别的语法，你可以将冲突文件的这些版本释放出一份拷贝
            - git show :1:hello.rb > hello.common.rb
            - git show :2:hello.rb > hello.ours.rb
            - git show :3:hello.rb > hello.theirs.rb
        - 既然在我们的工作目录中已经有这所有三个阶段的内容，我们可以手工修复它们来修复空白问题，然后使用鲜为人知的 git merge-file 命令来重新合并那个文件。
    - 检出冲突
    - 合并日志
    - 组合式差异格式
- 撤消合并
    - 修复引用
        - git reset --hard HEAD~
        - 这个方法的缺点是它会重写历史，在一个共享的仓库中这会造成问题的
    - 还原提交
        - git revert -m 1 HEAD
            - 如果你尝试再次合并 topic 到 master Git 会感到困惑
            - topic 中并没有东西不能从 master 中追踪到达。更糟的是，如果你在 topic 中增加工作然后再次合并，Git 只会引入被还原的合并之后的修改。
            - 解决这个最好的方式是撤消还原原始的合并 git revert ^M
    - 其他类型的合并
        - 我们的或他们的偏好
        - 子树合并
#### Rerere 
#### 使用 Git 调试
- 文件标注
    - git blame
- 二分查找
#### 子模块
- 开始使用子模块
    - 使用 git submodule add 命令后面加上想要跟踪的项目的相对或绝对 URL 来添加新的子模块
        - git status 中输出的目录
            - .gitmodules 用于记录项目 URL 与已经拉取的本地目录之间的映射
            - 另一个是项目文件夹记录
- 克隆含有子模块的项目
    - 克隆这样的项目时，默认会包含该子模块目录，但其中还没有任何文件
    - 必须运行两个命令
        - git submodule init 用来初始化本地配置文件
        - git submodule update 从该项目中抓取所有数据并检出父项目中列出的合适的提交
    - 更简单一点的方式
        - git clone --recurse-submodules
            - 自动初始化并更新仓库中的每一个子模块，包括可能存在的嵌套子模块
        - 如果你已经克隆了项目但忘记了 --recurse-submodules
            - git submodule update --init 将 git submodule init 和 git submodule update 合并成一步。
            - 如果还要初始化、抓取并检出任何嵌套的子模块
                - 使用简明的 git submodule update --init --recursive。
- 在包含子模块的项目上工作
    - 从子模块的远端拉取上游修改
        - 在子目录中手动抓取合并
        -  git submodule update --remote 
            - Git 将会进入子模块然后抓取并更新
        - git submodule update --remote
            - Git 默认会尝试更新 所有 子模块， 所以如果有很多子模块的话，你可以传递想要更新的子模块的名字
    - 从项目远端拉取上游更改
        - git pull 命令会递归地抓取子模块的更改
            - 但它不会更新子模块
            - 使用 git submodule update 更新
            - 为安全起见，如果 MainProject 提交了你刚拉取的新子模块，那么应该在 git submodule update 后面添加 --init 选项
                - 如果子模块有嵌套的子模块，则应使用 --recursive 选项。
        - 子模块的 URL 发生了改变
            - 复制新的 URL： git submodule sync
            - 从新 URL 更新子模块：git submodule update --init --recursive
- 在子模块上工作
    - 进入每个子模块并检出其相应的工作分支
    - 然后尝试用 “merge” 选项来更新子模块
        - git submodule update --remote --merge
    - 本地做了更改时上游也有一个改动时
        - git submodule update --remote --rebase
        - 如果忘记 --rebase 或 --merge， Git 会将子模块更新为服务器上的状态。并且会将项目重置为一个游离的 HEAD 状态。
            - 只需回到目录中再次检出你的分支（即还包含着你的工作的分支）然后手动地合并或变基 origin/stable（或任何一个你想要的远程分支）就行了
    - 发布子模块改动
        - git push 命令接受可以设置为 “check” 或 “on-demand” 的 --recurse-submodules 参数
            - 任何提交的子模块改动没有推送，那么 “check” 选项会直接使 push 操作失败
            - “on-demand” ，会尝试为你进入并推送子模块。如果子模块因为某些原因推送 失败，主项目也会推送失败
    - 合并子模块改动
    - 子模块技巧
        - 子模块遍历
            - foreach 子模块命令
        - 有用的别名
            - git config alias.sdiff '!'"git diff && git submodule foreach 'git diff'"
            - git config alias.spush 'push --recurse-submodules=on-demand'
            - git config alias.supdate 'submodule update --remote --merge'
        - 子模块的问题
            - 切换分支
                - 使用 Git 2.13 以前的版本时，在有子模块的项目中切换分支可能会造成麻烦。 如果你创建一个新分支，在其中添加一个子模块，之后切换到没有该子模块的分支上时，你仍然会有一个还未跟踪的子模块目录。
                - 新版的 Git（>= 2.13）通过为 git checkout 命令添加 --recurse-submodules 选项简化了所有这些步骤
            - 从子目录切换到子模块
#### 打包替换 
#### 凭证存储 
### 第八章 自定义 Git (略)
### 第九章 Git 与其他系统 (略) 
### 第十章 Git 内部原理
#### 底层命令与上层命令
- 当在一个新目录或已有目录执行 git init 时，Git 会创建一个 .git 目录。这个目录包含了几乎所有 Git 存储和操作的东西。如若想备份或复制一个版本库，只需把这个目录拷贝至另一处即可。
- 新初始化的 .git 目录的典型结构如下
    - config
        - 文件包含项目特有的配置选项
    - description
        - 文件仅供 GitWeb 程序使用
    - info/
        - 目录包含一个全局性排除文件
    - hooks/
        - 目录包含客户端或服务端的钩子脚本
    - HEAD
    - （尚待创建的）index
    - objects/
    - refs/
#### Git 对象
- 数据对象
    - git hash-object 会接受你传给它的东西，而它只会返回可以存储在 Git 仓库中的唯 一键
        - 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名
    - git cat-file 命令从 Git 那里取回数据
        - 指定 -p 选项可指示该命令自动判断内容的类型
        - git cat-file -t 命令，可以让 Git 告诉我们其内部存储的任何对象类型
        - 在这个简单的版本控制系统 中，文件名并没有被保存——我们仅保存了文件的内容，即数据对象
- 树对象
    - 一个树对象包含了一条或多条树对象记录
        - master^{tree} 语法表示 master 分支上最新的提交所指向的树对象
- 提交对象
    - 一个顶层树对象，代表当前项目快照
    - 可能存在的父提交
    - 之后是作者/提交者信息，外加一个时间戳
    - 留空一行，最后是提交注释。
- 对象存储
    - Git 首先会以识别出的对象的类型作为开头来构造一个头部信息。接着 Git 会在 头部的第一部分添加一个空格，随后是数据内容的字节数，最后是一个空字节。
    - Git 会将上述头部信息和原始数据拼接起来，并计算出这条新内容的 SHA-1 校验和
    - Git 会通过 zlib 压缩这条新内容。
#### Git 引用
- 用名字指针来替代原始的 SHA-1 值
    - 一个指向某一系列提交之首的指针或引用。这基本就是 Git 分支的本质。
    - 当运行类似于 git branch [branch] 这样的命令时，Git 实际上会运行 update-ref 命令， 取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新引用中。
- HEAD 引用
    - HEAD 文件通常是一个符号引用（symbolic reference），指向目前所在的分支。所谓符号引用，表示它是一个指向其他引用的指针
- 标签引用
    - 标签对象（tag object） 非常类似于一个提交对象
        - 它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。
        - 标签对象通常指向一个提交对象，而不是一个树对象。
        - 它像是一个永不移动的分支引用
    - 轻量标签
        - 一个固定的引用
    - 附注标签
        - 创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象
        - 标签对象并非必须 指向某个提交对象；你可以对任意类型的 Git 对象打标签。
- 远程引用
    - 远程引用和分支（位于 refs/heads 目录下的引用）之间最主要的区别在于，远程引用是只读的
#### 包文件
- “松散（loose）”对象格式
- 打包成包文件（packfile）”二进制文件，以节省空间和提高效率
- git gc 命令让 Git 对对象进行打包
- 第二个版本完整保存了文件内容，而原始的版本反而是以差异方式保存的——这是因为大部分情况下需要快速访问文件的最新版本
#### 引用规范
- 引用规范的格式由一个可选的 + 号和紧随其后的 [src]:[dst] 组成， 其中 [src] 是一个模式（pattern），代表远程版本库中的引用；[dst] 是本地跟踪的远程引用的位置。+ 号告诉 Git 即使在不能快进的情况下也要（强制）更新引用
- 引用规范拉取
    - 我们不能在模式中使用部分通配符，但我们可以使用命名空间（或目录）来达到类似目的
    - 命令行
        - git fetch origin master:refs/remotes/origin/mymaster
    - 配置文件
        - fetch = +refs/heads/master:refs/remotes/origin/master
- 引用规范推送
    - 命令行
        - git push origin master:refs/heads/qa/master
    - 配置文件
        - push = refs/heads/master:refs/heads/qa/master
- 删除引用
    - git push origin :topic
    - git push origin --delete topic
#### 传输协议 
#### 维护与数据恢复
- 维护
    - git gc --auto
        - 收集所有松散对象并将它们放置到包文件中， 将多个包文件合并为一个大的包文件，移除与任何提交都不相关的陈旧对象
        - 打包你的引用到一个单独的文件
            - .git/packed-refs
            - ^ 开头，表示它上一行的标签是附注标签，^ 所在的那一行是附注标 签指向的那个提交。
- 数据恢复
    - 硬重置后，之后的提交已经丢失了。需要找出最后一次提交的 SHA-1 然后增加一个指向它的分支。
    - git reflog
        - 会默默地记录每一次改变 HEAD 时它的值
        - git log -g 以标准日志的格式输出引用日志
        - 通过创建一个新的分支指向这个提交来恢复它
    - 引用日志丢失
        - git fsck -full
            - 会显示出所有没有被其他对象指向的对象
- 移除对象
    - 这个操作对提交历史的修改是破坏性的，
        - 在导入仓库后，在任何人开始基于这些提交工作前执行这个操作，那么将不会有任何问题
        - 否则， 必须通知所有的贡献者他们需要将他们的成果变基到你的新提交上。
    - 查询包大小
        - git gc
        - git count-objects -v
    - 查找要删除的大文件
        - git verify-pack -v
            - 列出文件信息
        - git rev-list --objects
            - 查找文件 SHA-1
    - 从过去所有的树中移除这个文件
        - 查看哪些提交对这个文件产生改动
            - git log --oneline --branches -- git.tgz
        - 重写某次提交之后的所有提交
            - git filter-branch
            - 历史中将不再包含对那个文件的引用。但引用日志和你在 .git/refs/original 通过 git filter-branch 添加的新引用中还存有对这个文件的引用，所以你必须移除它们然后重新打包数据库。在重新打包前需要移除任何包含指向那些旧提交的指针的文件
#### 环境变量 
- 全局行为
- 版本库位置
- 路径规则
- 提交
- 网络
- 比较和合并
- 调试
- 其它

