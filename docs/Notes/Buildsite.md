


1.  创建虚拟环境
```
conda create -n venv python=3.9.5
conda activate venv
```

2.  安装所需库
    - `pip install mkdocs`
    - `pip install mkdocs-material`


3.  创建网站
    - `mkdir xvliote`
    - `cd xvliote`
    - `mkdocs new mysite`

4.  提交

    - 在包含`.yml`的文件中进行`git`操作
    - `git init`  
    - `git add .`
    - `git commit -m "first commit"`
    - `git remote add origin git@github.com:xvliote/xvliote.github.io.git`
    - `git push -u origin main`
    - `mkdocs gh-deploy`

5.  修改提交
    - `git status ` 
    - `git add .`
    - `git commit -m "xxxxx"`
    - `git push -u origin main`
    - `mkdocs gh-deploy`










要在 CentOS 7 上下载并安装 GCC 11，您可以按照以下步骤操作：

启用 SCL（软件集合）存储库：
sudo yum install centos-release-scl
安装 GCC 11 和 G++：
sudo yum install devtoolset-11-gcc devtoolset-11-gcc-c++
切换到使用 GCC 11 的 shell：
scl enable devtoolset-11 bash
验证安装：
gcc --version
g++ --versionå