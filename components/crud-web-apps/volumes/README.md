# Volumes web 应用

此 Web 应用程序负责允许用户在其 Kubeflow 集群中操作 PVC。 为此，它提供了一种用户友好的方式来处理 PVC 对象的生命周期。

## 开发

要求：
* node 12.0.0
* python 3.8

### 前端

```bash
# build the common library
cd components/crud-web-apps/common/frontend/kubeflow-common-lib
npm i
npm run build
cd dist/kubeflow
npm link

# build the app frontend
cd ../../../volumes/frontend
npm i
npm link kubeflow
npm run build:watch
```

### 后端
```bash
# 创建虚拟环境并安装依赖
# https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/
cd component/crud-web-apps/volumes/backend
python3.8 -m pip install --user virtualenv
python3.8 -m venv web-apps-dev
source web-apps-dev/bin/activate

# 在激活的虚拟环境安装依赖
make -C backend install-deps

# 运行后端
make -C backend run-dev
```

### 国际化
只有尽最大努力才能支持非英语语言。

在前端国际化(i18n) 通过 [angular's i18n](https://angular.io/guide/i18n) 来实现指引和实践。
您可以使用以下方法确保
应用程序的文本本地化：
1. `i18n` 属性，应该翻译节点的文本
2. `i18n-{attribute}` 在html元素中，应该翻译元素的属性
3. [$localize](https://angular.io/api/localize/init/$localize) 标记应翻译
   的 typescript 变量文本

英语文本文件放在 `i18n/messages.xlf` 而其他
语言分别当道对应的目录，如 `i18n/fr/messages.fr.xfl`。
除英语外，每种语言的文件夹都应该有一个
不同的、最新的所有者文件，该文件反映了该语言的维护者。

**测试**

您可以通过运行在本地运行应用程序的不同翻译
```bash
ng serve --configuration=fr
```

您还必须确保后端在运行中，argular 的开发服务器
通过代理将请求发送到后端 `localhost:5000`。
