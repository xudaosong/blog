React前端工程化
==========

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [create-react-app](#create-react-app)
* [代码组织](#代码组织)
* [编码规范](#编码规范)
	* [eslint代码检查](#eslint代码检查)
	* [通过git hooks强制代码风格检查](#通过git-hooks强制代码风格检查)
		* [本地检查](#本地检查)
		* [远程检查](#远程检查)
	* [React编码规范](#react编码规范)
	* [CSS编码规范](#css编码规范)
* [文档生成](#文档生成)
* [反向代理](#反向代理)
* [数据模拟](#数据模拟)
* [后端接口环境](#后端接口环境)
* [redux去样板代码](#redux去样板代码)
* [命令行工具](#命令行工具)
* [效率工具](#效率工具)
	* [Visual Studio Code](#visual-studio-code)
		* [代码格式化符合eslint的代码风格检查](#代码格式化符合eslint的代码风格检查)

<!-- /code_chunk_output -->

## create-react-app

## 代码组织

## 编码规范
在create-react-app的eslint-config-react-app规范的基础上，添加eslint-config-airbnb的规范

执行步骤：
1. `yarn add eslint-config-airbnb --dev`
2. `.eslintrc.json`文件里修改 `"extends": ["react-app", "airbnb"]`
3. [Visual Studio Code代码格式化符合eslint的代码风格检查](#代码格式化符合eslint的代码风格检查)

### eslint代码检查
   
### 通过git hooks强制代码风格检查
针对痛点：
1. 代码规范落地难
2. 低质量代码带入线上应用
3. 代码格式难统一
4. 代码质量文化难落地
5. 着急上线，本地直接忽略风格检查

#### 本地检查
使用husky + lint-staged + commitizen在本地git commint或push操作时进行代码风格自动fixed、风格检查并对commit message进行校验。

执行步骤：
1. `yarn add husky --dev`
2. `yarn add lint-staged --dev`
3. `yarn add --dev commitizen cz-conventional-changelog`
4. `yarn add --dev @commitlint/config-conventional @commitlint/cli`
5. 修改package.json
	```json
	"scripts": {
		"precommit": "lint-staged",
    	"commit": "git-cz"
	},
	"husky": {
		"hooks": {
			"pre-commit": "yarn precommit",
			"pre-push": "yarn precommit",
      		"commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
		}
	},
	"lint-staged": {
		"src/**/*.{js,jsx}": [
			"eslint --fix",
			"git add"
		]
	},
	"config": {
		"commitizen": {
			"path": "./node_modules/cz-conventional-changelog"
		}
	}
	```
5. 项目根目录下创建commitlint.config.js, 内容如下：
   ```es6
	module.exports = {
		extends: ['@commitlint/config-conventional'],
	};
   ```
6. 使用`yarn commit`提交代码，或者使用插件
7. (可选) Vistual Code Studio安装`Visual Studio Code Commitizen Support`插件，通过`ctrl+shift+p`执行`conventional commit`命令
   
传送门：
- [husky](https://github.com/typicode/husky): 为本地git commint或push等操作添加hooks
- [lint-staged](https://github.com/okonet/lint-staged): 在每次提交时只检查本次提交的文件
- [commitizen](https://github.com/commitizen/cz-cli): 为Git Commit Message添加校验
- [commitlint](https://github.com/conventional-changelog/commitlint)

参考文献：
- [用 husky 和 lint-staged 构建超溜的代码检查工作流](https://segmentfault.com/a/1190000009546913)
- [eslint+husky+prettier+lint-staged提升前端应用质量](https://juejin.im/post/5c67fcaae51d457fcb4078c9)
- [优雅的提交你的 Git Commit Message](https://juejin.im/post/5afc5242f265da0b7f44bee4)
- [Commitizen的安装和使用](https://www.jianshu.com/p/d264f88d13a4)

#### 远程检查

### React编码规范

### CSS编码规范

## 文档生成

## 反向代理

## 数据模拟

## 后端接口环境

## redux去样板代码

## 命令行工具

## 效率工具
### Visual Studio Code
#### 代码格式化符合eslint的代码风格检查
1. 安装esLint + prettier插件 不要装 js-beautify
2. `ctrl+shift+p` 打开命令面板，输入 `Open User Settings`
3. 定位 extensions => Prettier - Code formatter configuration => Prettier: Eslint Integration => 勾选 Use 'prettier-eslint' instead of 'prettier'.