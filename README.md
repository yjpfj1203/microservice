# microservice 微服务的parent服务，主要使用是将所有service集中在一个service下，可以一起pull，push
命令如下：
## 克隆此仓库
git clone https://xxxxx.git
## 切换到master分支
git checkout master
## 更新submodule代码
git submodule update --init --recursive
## 将所有的submodule切换到master分支
git submodule foreach 'git checkout master'
