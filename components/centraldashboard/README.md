# Kubeflow 登录页面

该组件用作 Kubeflow 部署的登录页面和中央仪表板。
它为平台的所有其他方面提供了一个起点。

## 构建和部署

此文件夹的构建工件是一个 Docker 容器映像，
它托管为 API 端点和前端应用程序代码提供服务的 Express 网络服务器。

要构建新的容器镜像，在文件夹内执行 `make` 。要推送到
kubeflow-images-public 存储库，执行 `make push`。

### 在现有的 Kubeflow 开发上测试构建

在运行 `make push` 之后，记下创建的标签。

```
Pushed gcr.io/kubeflow-images-public/centraldashboard with :v20190328-v0.4.0-rc.1-280-g8563142d-dirty-319cfe tags
```

然后，使用 **kubectl** 来更新现有中央仪表盘的镜像，
指定上面步骤输出镜像名称和标签。

```
kubectl --record deployment.apps/centraldashboard \
    set image deployment.v1.apps/centraldashboard \
    centraldashboard=gcr.io/kubeflow-images-public/centraldashboard:v20190328-v0.4.0-rc.1-280-g8563142d-dirty-319cfe
```

## 开发

### 开始
确保你已经安装了最新的 `node` 版本并附带了 `npm`。

1. 克隆存储库并切到 `components/centraldashboard` 目录
2. 运行 `make build-local`。这将安装所有项目依赖以
   准备好你的系统开发。
3. 要开始一个本地环境，运行 `npm run dev`。
    - 这将在 [public](./public) 文件夹里的前端代码中运行 [webpack](https://webpack.js.org/) 并启动
      [webpack-dev-server](https://webpack.js.org/configuration/dev-server/) 在
      http://localhost:8081.
    - 它同时也在 http://localhost:8082 启动了 Express API 服务。
      来自前端的 `/api` 请求将代理到 Express
      服务。 所有其他请求都由反映生产配置的前端服务器处理。
4. 要接入 Kubenetes REST API，执行 `kubectl proxy --port=8083`。
   - 执行 [kubectl proxy](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#proxy) 来创建一个反向代理 http://localhost:8083。来自前端的 `/jupyter`、`/metadata`、`/notebook` 及 `/pipeline` 请求会代理到 [Kubenetes API server](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)。查看 [webpack 配置文件](https://github.com/kubeflow/kubeflow/blob/master/components/centraldashboard/webpack.config.js) 获取更多信息。

### Server 组件

服务端代码存储在 [app](./app) 目录。服务端使用
[Express](https://expressjs.com/) 由 [Typescript](https://www.typescriptlang.org/docs/home.html) 编写。

### 前端组件

客户端代码存储在 [public](./public) 目录。客户端组件
使用 [Polymer 3](https://polymer-library.polymer-project.org/3.0/docs/about_30)
web 组件类库。所有 Polymer 组件需要在
[public/components](./public/components) 路径编写。你可以使用[内联样式](https://polymer-library.polymer-project.org/3.0/docs/first-element/step-2)来创建你的 DOM，或者将 CSS 和 HTML 分开
指定到不同的文件中。我们现在用 [Pug](https://pugjs.org/api/getting-started.html)
模板支持额外的 markup。 查看 [main-page.js](public/components/main-page.js)
示例。

### 测试

在引入新的更改时，您应该确保您的代码已经过测试。按照
惯例，测试应与被测代码位于同一目录中，并
以 `_test.(js|ts)` 后缀命名。比如名叫
`new-fancy-component.js` 的模块，它的测试用例应是为 `new-fancy-component_test.js` 的文件。

服务端和客户端单元测试都需要使用 [Jasmine](https://jasmine.github.io/api/3.3/global) 编写。
默认情况下，客户端测试使用无头 Chrome 运行 Karma。

#### 代码覆盖

要运行单元测试和代码发覆盖，执行 `npm run coverage`。这将在中断输出
覆盖统计，并为 `coverage` 下的
`app` 和 `public` 创建子目录来展示代码覆盖。
请尝试为具有业务逻辑的代码部分争取高水平的覆盖率。

---

## 样式指引
Kubeflow 仪表板是一个可视化和网络平台，将 Kubeflow 生态系统中的其他子应用程序
链接在一起。 因此，我们在应用程序中拥有了各种 iframe 的网络应用程序。
为了保持这种体验的统一，我们为
所有 Kubeflow 集成商提供了风格指南以供遵循/使用。

### 方法
CSS 调色板在 [_public/kubeflow-palette.css_](public/kubeflow-palette.css)。
当发布到集群，可通过 `cluster-host/kubeflow-palette.css` 引入。

Name | Value | Usage
--- | --- | ---
`--accent-color` | #007dfc | 应用于突出显示动作按钮、边框和 FABs
`-kubeflow-color` | #003c75 | Kubeflow CentralDashboard 主背景色
`--primary-background-color` | #2196f3 | 为所有指引元素背景色 (Toolbars, FOBs)
`--sidebar-color` | #f8fafb | 子应用边栏背景色
`--border-color` | #f4f4f6 | 页面所有边框颜色

_**Note:** 此样式指南可能会发生更改，最终将作为CDN使用，因此最终不需要手动更新。_

### 一般原则
- 尽可能追随 Material Design 2 规范
- Elevations 应满足 [_**MD2 Elevation 规范**_](https://material.io/design/environment/elevation.html)
- 子应用程序不应提及名称空间，而应使用中央仪表板提供的值。

### 用法
下载 [_**styleguide**_](public/kubeflow-palette.css)，并在 html 中引入：

```html
<head>
  <!-- Meta tags / etc -->
  <link rel='stylesheet' href='kubeflow-palette.css'>
```

然后在样式表中，可以使用调色板变量，如：

```css
body > .Main-Toolbar {background-color: var(--primary-background-color)}
.List > .item:not(:first-of-type) {border-top: 1px solid var(--border-color)}
/* etc */
```

## 客户端类库
仪表盘使用 iframe 包含其他 Kubeflow 组件，一个
一个独立的 Javascript 库暴漏子页面和仪表板，使其之间的通信成为可能。

通过在集群引入 `/dashboard_lib.bundle.js` 类库。
当通过这种方式引用，将在全局范围创建
一个 `centraldashboard` 模块。 要在仪表盘和 iframe 引入页面建立通信
信道，使用如下禁用
`<select>` 元素的示例，并在仪表盘的空间选择器更新时赋予新值：

```js
window.addEventListener('DOMContentLoaded', function (event) {
    if (window.centraldashboard.CentralDashboardEventHandler) {
        // Init method will invoke the callback with the event handler instance
        // and a boolean indicating whether the page is iframed or not
        window.centraldashboard.CentralDashboardEventHandler
            .init(function (cdeh, isIframed) {
                var namespaceSelector = document.getElementById('ns-select');
                namespaceSelector.disabled = isIframed;
                // Binds a callback that gets invoked anytime the Dashboard's
                // namespace is changed
                cdeh.onNamespaceSelected = function (namespace) {
                    namespaceSelector.value = namespace;
                };
            });
    }
});
```
