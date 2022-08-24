# Kubeflow 通用前端库

这段代码提供了一个通用的可重用 Angular 组件库，可以在我们不同的 Kubeflow Web 应用程序中使用。该库旨在：
* 在不同的应用程序中实施通用的用户体验
* 减少将更改传播到所有 Web 应用程序所需的开发工作量
* 最大限度地减少我们的 Kubeflow Web 应用程序之间的重复代码

项目通过 [Angular CLI](https://github.com/angular/angular-cli) 版本 8.3.20 生成，这也是构建和运行单元测试所必需的。

## 本地开发
为了在本地开发 Angular 应用程序时使用此库，您需要：
1. 从此源代码构建 `kubeflow` 节点模块
2. 将生成的模块链接到您的全局 npm 模块
3. 在你的应用程序的 npm 模块中链接 `kubeflow` 模块

### 在本地建立库
```bash
# build the npm module
npm run build

# might need sudo, depending on where you global folder lives
# https://nodejs.dev/learn/where-does-npm-install-the-packages
npm link dist/kubeflow
```
### 将其链接到应用程序
```bash
cd ${APP_DIR}
npm install
npm link kubeflow
```

## 运行单元测试

运行 `ng test` 来通过 [Karma](https://karma-runner.github.io) 执行单元测试。

## 贡献者指南

### 单元测试
1. 添加到这个库的任何新组件还应该包括一些基本的单元测试
2. 单元测试应该在任何时候通过

### Git 提交
修改此代码的 Git 提交应以 `web-apps(front)` 为前缀。
