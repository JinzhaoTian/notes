

> 参考：[官方文档](https://docs.conda.io/en/latest/)，[Anaconda](https://anaconda.org/anaconda/conda)

Conda 是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。 Conda 是为 Python 程序创建的，适用于 Linux，OS X 和Windows，也可以打包和分发其他软件。

### 命令

* 创建环境：`conda create -n [name] python=3.8`

* 删除环境：`conda remove -n [name] --all`

* 复制环境：`conda create -n [newenv] --clone [oldenv]`

* 激活环境：`source activate [name]`

* 退出环境：`conda deactivate`

* 显示所有环境： `conda env list`，`conda info -e`

  

* 显示已安装的包：`conda list`

* 搜索包：`conda search [name]`

* 安装：`conda install [name]`

* 卸载：`conda uninstall [name]` 

* 删除没有用的包：`conda clean -p`

  

* 添加新镜像源：`conda config --add channels [channelsurl]`

  * -channels:  
    * `- https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/`
    * `- https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/`



### 与pip的区别

**pip**

> - **pip专门管理Python包**
> - **编译源码中的所有内容**。 **（源码安装）**
> - 由核心Python社区所支持（即，Python 3.4+包含可自动增强pip的代码）。



**conda**

> * conda本身是用Python编写的，但你也可以为C库或R软件包或任何其他软件包提供conda软件包。
> * 安装的是二进制文件。 有一个名为conda build的工具，它可以从源代码构建软件包，但conda install本身会安装已经构建的conda软件包中的东西。
> * Conda是Anaconda的包管理器，由Continuum Analytics提供的Python发行版，但它也可以在Anaconda之外使用。 您可以使用现有的Python安装，通过pip安装它（尽管除非您有充分理由使用现有安装，否则不建议这样做）。



## 安装

### Linux


```
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
```

```
bash Anaconda3-2024.02-1-Linux-x86_64.sh
```