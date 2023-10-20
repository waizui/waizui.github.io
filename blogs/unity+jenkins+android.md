# Jenkins自动化构建Unity+Android混合项目

## 背景

* 有的时候我们需要将Unity工程导出Android工程，再与其它的Android工程混合开发。这样的开发方式在自动化构建时会产生一个问题:
不能使用Unity内置的Apk导出功能，构建过程包含多种互相独立的步骤，这意味着要自动构建时要使用其它方式来粘合构建过程。


## 问题
要实现混合开发的的自动构建过程，需要解决几个问题:
* Unity打包自动化打包并导出Android工程的功能模块。
* 自动更新Unity工程仓库跟Android仓库。
* 库文件跟资源文件的拷贝。
* 目标Android工程的自动化打包APK功能。
* 打包的清理跟后续操作。


## 方案

* 自动化工具选择Jenkis,Jenkins的安装和部署可以参考:[安装Jenkins](https://www.cnblogs.com/dotnet261010/p/11495762.html)
* 构建过程使用Python脚本粘合
* Android构建使用Gradle工具 [Gradle安装](https://www.w3cschool.cn/gradle/ctgm1htw.html)


## 构建步骤
以window平台为例子

1. 第一步是更新Unity工程和目标Android工程，这一步以git为例。使用Python中的GitPython模块，可以实现分支拉取和切换。
其中的核心函数是操作GitPython，更新git仓库和切换分支。

```Python
    from git import Repo
    ...
    def pull_remote(self):
        print("start update git repo")
        # 获取指定地址的仓库对象
        repo = self.get_repo()
        # 获取目标分支名
        branchStr = self.get_cur_branch()
        head = repo.head

        try:
            #检查本地分支
            branch = repo.branches[branchStr]
        except IndexError as e:
            # 没有检出的话需要检出
            remoteHead = repo.remote().refs[branchStr]
            remoteHead.checkout(force=True, b=branchStr)

        branch = repo.branches[branchStr]
        try:
            #要丢弃所有更改
            head.reset(working_tree=True)
            repo.git.clean("-df")
            #尝试checkout
            branch.checkout()
            repo.remote().pull()
        except Exception as e:
            print("error on update repo")
            exit(1)

        print("update git repo finished")
```
#
2. 在Unity编辑器中，需要编写自动化打包的模块，这个模块同时要包含一个读取命令行参数的静态打包入口，
这样才可以使用Unity的命令行启动方式来构建。比如在unity中声明了一个静态打包函数:
```C#
namespace XXX {
    public static void CommandLineCompileBuild(){
        ...
    }
}
```
那么可以在cmd或者shell中使用如下语句调用该函数

```Shell
    xxx/xxx/unity.exe -projectPath your-unityproject-path -executeMethod XXX.CommandLineCompileBuild 
```
其中-projectPath参数指定要打包的工程的绝对路径

在真正开始启动Unity打包之前,一般会有一段预处理的工作，比如清除中间文件，生成代码等，这些也可以在python脚本中操作。
其中最重要的是关闭正在占用该工程的Unity进程。核心函数是通过查找Unity的LockFile文件的持有进程来关闭该Unity进程。

```Python
    import psutil
    from distutils.dir_util import copy_tree, remove_tree

    ...

    def close_unity_process(self):
        # 获取工程Root路径
        projPath = self.get_unity_project_path()
        tempDir = (projPath + r"\Temp")
        lockFile = tempDir + r"\UnityLockfile"
        exists = os.path.exists(lockFile)
        if not exists:
            return

        unity_name = 'Unity.exe'
        for proc in psutil.process_iter():
            try:
                if proc.name() != unity_name:
                    continue

                open_files = proc.open_files()
                if not open_files:
                    continue

                found = False
                for open_file in open_files:
                    if os.path.abspath(open_file.path) != lockFile:
                        continue

                    found = True
                    print(f'Kill open Unity process: {proc.pid}')
                    sys.stdout.flush()
                    proc.kill()
                    time.sleep(10)
                    break

                if found:
                    break

            except psutil.NoSuchProcess as err:
                print("****", err)
        if os.path.isdir(tempDir):
            remove_tree(tempDir)

```

启动Unity并且打包的过程，可以使用python中提供的os.system函数,该通过cmd启动一个新进程，并且阻塞等待进程退出，
返回进程的返回值。例如：

```Python
    import os

    ...

    statusCode = os.system("xxx/xxx/unity.exe -projectPath your-unityproject-path -executeMethod XXX.CommandLineCompileBuild")
    if statusCode != 0:
        print("error while building:{} ".format(statusCode))
        exit(statusCode)

```
等待Unity输出完工程以后，需要使用Gradle进行打包编译(IL2Cpp)，在pyhon中可以调用Gradlew工具进行打包，需要提前安装好gradle。


```Python

    ...

    def build_gradle(self, asPath):
        # gradle 运行文件的全路径
        gradleExePath = "path to gradle executable file"
        # 切换到传入的Android工程目录
        os.chdir(asPath)
        # 首先build gradle wrapper 产生gradlew
        code = os.system("{} wrapper".format(gradleExePath))
        if code != 0:
            exit(1)

        isRelease = self.isRelease()
        cmd = "assembleRelease" if isRelease else "assembleDebug"
        # 使用gradlew assembleXXX 打包
        code = os.system("gradlew {}".format(cmd))
        if code != 0:
            exit(1)
```


打包完成后，可以根据项目的实际情况，复制Gradle打包输出的资源和库文件到目标Android工程中。这一步可能会遇到Gradle的各种参数不对，
比如Unity输出的SDK，NDK地址格式错误等等，这些需要在脚本里显式的修复，不然无法进行下一步。

最后在目标Android工程中，再次执行上面的Gradle打包，得到最终的包体。需要注意的是，如果输出release包，需要额外的资源对齐和签名操作，
如果嫌麻烦可以将jks文件密码写在build.gradle里，这样在assembleRelease时会自动进行这些操作。不过这样做的话需要严格限制此Android工程的权限，
或者不要将build.gradle放在版本控制里，不然每个人都能看到密码。

#

3. 在Jenkins中，需要连接上面写好的脚本，设置参数，周期等。
*  首先需要设置一些环境变量，在Dashboard界面中打开Jenkins的Manage Jenkins选项，再找到Configure System，找到Environment variables.
添加下面下面几个变量。
    1. ANDROID_HOME, 指向Android sdk的根文件夹。
    2. GIT_PYTHON_GIT_EXECUTABLE, 指向git运行文件的全路径，比如"C:\Program Files\Git\bin\git.exe"，GitPython模块依赖这个变量。
    3. GRADLE_USER_HOME, 指向gradle的bin文件夹，如果没有，需要先安装Gradle。
    4. JAVA_HOME, 指向JdK的根文件夹。   
* 接下来在Dashboard新建一个Item，选择freestyle。
    1. 在构建中，选择"Execute Windows batch command"，然后根据项目实际情况，输入执行打包脚本的命令。跟直接在cmd中运行是一样的。
    如果需要命令行参数，可以用"%变量名%"这样的语法来引用在Jenkins定义的变量。这个变量定义可以在Item中选择"This is parameterized"
    选项来添加。
    2. 在Item的构建环境这个选项中，选择"Use secret text(s) or file(s)"来配置git仓库的用户名和密码。选择新增然后选择"Git Username and Password"
    选项，然后输入用户名和密码。
    3. 最后就是周期性自动构建，选择构建触发器中的"Build Periodically",在日程表中输入指定的语法规则就可以按照规则来周期性构建。
    最简单的规则比如"0 9,13,19  * * *" 代表在每天的9点，13点和19点进行构建。
    4. 打包完成后的文件清理，也可以用Python执行，在Item中再按1的步骤添加一个"Execute Windows batch command"即可。这些command将会依次执行。


#
4. 如果一切顺利，那么自动化打包输出就完成了。打包会自动拉取最新的git提交来打包，如果需要不同分支，或者其它的打包策略，只需要稍加配置就能实现。