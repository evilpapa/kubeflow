# Jupyter web 应用

该 Web 应用程序允许用户在他们的 Kubeflow 集群中操作 Jupyter Notebook。 为了实现这一目标，它提供了一种用户友好的方式来处理 Notebook CR 的生命周期。

## Image Groups

随着 Kubeflow 1.3 的发布，除了熟悉的 JupyterLab 之外，还添加了两种类型的笔记本服务器：

- Group 1
- Group 2

一些额外的配置适用于属于这些组的笔记本服务器：

`notebooks.kubeflow.org/http-rewrite-uri: /` 注解添加到两个组的 Notebook 资源中。
This configures Istio to rewrite the URI to `/` on
the container. This is useful for applications which host their on `/`
and do not allow you to change the URI subpath easily.

The annotation `notebooks.kubeflow.org/http-headers-request-set:`
`'{"X-RStudio-Root-Path":"/notebook/<namespace>/<name>/"}'` is added to
Notebook resources belonging to Group 2. This configures Istio to add
this header to requests, which is necessary for images from Group 2 to work.

The Jupyter Web App displays the logos for each Notebook Server group
in a button toggle in the Spawner UI. To easily identify the group of
a running Notebook Server, the Index page contains a column that displays
the icon for each image group. The SVG logos and icons for each group are added
with a [configmap](./manifests/base/configs/logos-configmap.yaml) to make it easy for users to customize the logos and icons for their environment.

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
cd ../../../jupyter/frontend
npm i
npm link kubeflow
npm run build:watch
```

### 后端
```bash
# create a virtual env and install deps
# https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/
cd components/crud-web-apps/jupyter/backend
python3.8 -m pip install --user virtualenv
python3.8 -m venv web-apps-dev
source web-apps-dev/bin/activate

# install the deps on the activated virtual env
make -C backend install-deps

# run the backend
make -C backend run-dev
```

### Internationalization
Support for non-English languages is only supported in a best effort way.

Internationalization(i18n) was implemented using [Angular's i18n](https://angular.io/guide/i18n)
guide and practices, in the frontend. You can use the following methods to
ensure the text of the app will be localized:
1. `i18n` attribute in html elements, if the node's text should be translated
2. `i18n-{attribute}` in an html element, if the element's attribute should be
   translated
3. [$localize](https://angular.io/api/localize/init/$localize) to mark text in
   TypeScript variables that should be translated

英文文本文件放在 `i18n/messages.xlf` 路径，并且
其他语言也是放在相对的本地文件夹中，比如 `i18n/fr/messages.fr.xfl`.
每个语言文件夹，除了Each language's folder, aside from English, should have a distinct and up to
date OWNERs file that reflects the maintainers of that language.

**测试**

通过命令，你可以在本地运行不同的 app 翻译：
```bash
ng serve --configuration=fr
```

同样需确认后端也在运行中，Angular 的开发服务会代理请求倒后端地址 `localhost:5000`。

## E2E 测试

要在本地运行测试，你需要：
```bash
# navigate to the frontend and make sure the node modules are installed
cd frontend
npm i
# ensure the backend is running and serving the files under localhost:5000
# you can run the backend with: make run-dev
# run the tests
npm run e2e
```
