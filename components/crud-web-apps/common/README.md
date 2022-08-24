# CRUD web 应用通用代码

由于我们的 CRUD web 应用程序（如 Jupyter、Tensorboards 和 Volumes UI）类似地使用 Angular 和 Python/Flask 构建，因此我们考虑将常见代码抽象到模块和库中。

目录包含：
1. 基础后端的 Python 包。每个提到的应用程序都应该扩展这个后端。
2. 一个 Angular 库，其中包含这些应用程序将共享的通用前端代码。

## 后端

后端将公开一个基本后端，该后端将负责：
* 服务单页应用程序
* 添加 liveness/readiness 探针
* 基于 `kubeflow-userid` 头的身份验证
* 使用 SubjectAccessReviews 授权
* 统一日志
* 异常处理
* 日期、yaml 文件解析等的常用辅助函数。

### 支持的 ENV 变量

以下是可以为使用此基本应用程序的任何 Web 应用程序设置的 ENV var 列表。
此列表不完整，我们将在未来添加更多变量。

| ENV Var | 描述 |
| - | - |
| CSRF_SAMESITE | 控制 [SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#SameSite) CSRF cookie 值 |

### 如何使用

为了在开发过程中使用此代码，可以在 `pip install` 时使用 `-e` [标识](https://pip.pypa.io/en/stable/reference/pip_install/#install-editable)。比如：

```bash
# I'm currently in /components/crud-web-apps/volumes/backend
cd ../../common/backend && pip install -e .
```

这将安装包的所有依赖项，您现在可以在当前 Python 环境中包含来自 `kubeflow.kubeflow.crud_backend` 的代码。

为了构建 Docker 映像并使用此代码，您可以构建一个 wheel，然后安装它：
```dockerfile
### Docker
FROM python:3.7 AS backend-kubeflow-wheel

WORKDIR /src
COPY ./components/crud-web-apps/common/backend .

RUN python3 setup.py bdist_wheel

...
# Web App
FROM python:3.7

WORKDIR /package
COPY --from=backend-kubeflow-wheel /src .
RUN pip3 install .
...
```
## 前端

通用 Angular 库包含以下通用代码：
* 与看板中心通信来处理命名空间选择
* 进行 http 调用并处理错误
* 显示错误和警告
* 轮询机制
* 跨不同 Web 应用程序的通用样式
* 处理表格

### 如何使用
```bash
# build the common library
cd common/frontend/kubeflow-common-lib
npm i
npm run build

# for development link the created module to the app
cd dist/kubeflow
npm link

# navigate to the corresponding app's frontend
cd crud-web-apps/volumes/frontend
npm i
npm link kubeflow

```
### 常见错误
```
NullInjectorError: StaticInjectorError(AppModule)[ApplicationRef -> NgZone]:
  StaticInjectorError(Platform: core)[ApplicationRef -> NgZone]:
    NullInjectorError: No provider for NgZone!
```

你也需要添加 `"preserveSymlinks": true` 到前端应用 `angular.json` 文件的 `projects.frontend.architect.build.options` 选项。

### Docker

```dockerfile
# --- Build the frontend kubeflow library ---
FROM node:12 as frontend-kubeflow-lib

WORKDIR /src

COPY ./components/crud-web-apps/common/frontend/kubeflow-common-lib/package*.json ./
RUN npm install

COPY ./components/crud-web-apps/common/frontend/kubeflow-common-lib/ .
RUN npm run build

...
# --- Build the frontend ---
FROM node:12 as frontend
RUN npm install -g @angular/cli

WORKDIR /src
COPY ./components/crud-web-apps/volumes/frontend/package*.json ./
RUN npm install
COPY --from=frontend-kubeflow-lib /src/dist/kubeflow/ ./node_modules/kubeflow/

COPY ./components/crud-web-apps/volumes/frontend/ .

RUN npm run build -- --output-path=./dist/default --configuration=production

```

### 国际化
国际化使用 [ngx-translate](https://github.com/ngx-translate/core) 实现。

这是基于浏览器的语言。如果浏览器检测到应用程序中未实现的语言，它将默认为英语。

i18n 资产文件位于每个应用（jupyter, volumes 及 tensorboard）的 `frontend/src/assets/i18n` 目录。一个语言需要一个文件。公共项目在每个资产中都有重复。

翻译资产文件设置在 `app.module.ts` 中，不需要修改。
翻译默认语言设置在 `app.component.ts`。

对于添加的每种语言，都需要更新 `app.component.ts`。

**添加语言时：** 
- 复制 en.json 文件并将其重命名为您要添加的语言。 按照目前的情况，不应该包括文化。
- 将值更改为已翻译的值

**添加或修改翻译时：**
- 选择合适的键
- 确保在每个语言文件中添加键
- 如果在通用工程中添加/修改了文本，则也需要在其他应用程序中添加/修改。

**测试**

要测试 i18n 是否按预期工作，只需将浏览器的语言更改为您要测试的任何语言。
