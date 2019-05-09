# 使用verdaccio搭建npm私服

## 搭建私服的好处

1. 模块复用性角度考虑
   
   多个项目之间有重复的共有模块，当需要修改模块，通过简单的统一的配置就可以实现；提炼后的组件有专门的地址可以用来查看，方便使用，在后期项目的引用中也能节约开发成本

2. 公司技术沉淀角度考虑
   
   知识的沉淀，在公司业务相关的应用上尤佳；

3. 开发效率角度考虑：
   
   使私有公共业务或组件模块能以共有包一样的管理组织方式，保持一致性，提高开发效率；

4. 版本角度的考虑
   
   相当于一个容器，统一管理需要的包，保持版本的唯一；

5. 安全性角度考虑
   
   如果我们想要一个公共组件库，那么把组件放到我们私有库中，只有内网可以访问，这样可以避免组件中业务的泄露；

## 为什么使用verdaccio搭建npm私服

目前可以通过以下方式搭建私服

1. 阿里的 cnpmjs.org
2. Maven/nexus 3
3. sinopia
4. verdaccio

cnpm和nexus部署私服较为复杂，cnpm需要安装数据库。
sinopia是verdaccio的前身，由于sinopia作者已弃坑，多年失修，因此排除了sinopia。

verdaccio的优势：搭建简单方便，不需要数据库

## 搭建私服
1. 服务器要求
   服务器能访问外网，并确保内网的所有同学都能访问
2. 安装node/python/yarn软件
   - node建议安装LTS最新的版本
   - yarn建议安装最新版本
   - python建议安装2.x或3.x的最新版本，需要设置到环境变量
3. 安装verdaccio/pm2插件
   ```
   yarn global add verdaccio
   yarn global add pm2
   ```
4. 修改verdaccio配置
   window系统打开C:\Users\用户名\AppData\Roaming\verdaccio\config.yaml文件，如果是linux系统请找到对应的config.yaml，做下面修改：
   - 把uplinks里的https://registry.npmjs.org/ 改为 https://registry.npm.taobao.org/
   - 添加地址监听，确保能通过远程地址访问
       ```
       #listen port
       listen: 0.0.0.0:4873
       ```
5. 启动服务
   使用pm2进程守护启动服务: `pm2 start verdaccio`
6. 检查安装
   在浏览器打开http://远程ip地址:4873 ,能正常访问即可

## 使用私服
1. 安装node/yarn
	node建议安装LTS最新的版本
	yarn建议安装最新版本

2. 安装nrm插件
	`yarn global add nrm`

3. 通过nrm注册仓库
	`nrm add privateNpmName http://私服ip地址:4873`
    privateNpmName: 可修改为有意义的名称,比如公司名

4. 切换到私服仓库
	`nrm use privateNpmName`

5. 注册账号，需输入用户名、密码和邮箱
	`npm adduser --registry http://私服ip地址:4873`

6. 在需要发包的项目里设置package.json的name为：@companyName/***，如：@leedarson/***，注意版本号的管理。
	
7. 项目发布
	在项目根目录执行：`yarn publish`

8. 确认发布成功
   访问：http://私服ip地址:4873，确认仓库已发布