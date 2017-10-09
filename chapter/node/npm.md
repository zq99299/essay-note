# npm

## 添加用户
```bash
# registry 是指定源，有时候把源更好成了淘宝的镜像
$ npm adduser --registry http://registry.npmjs.org
# 命令执行后，会依次出现以下填写项
Username: work_lib_tlz
Password:
Email: (this IS public) work_lib_tlz@126.com
Logged in as work_lib_tlz on http://registry.npmjs.org/.
```

## node-gyp.js error MSB6006: “VCBuild.exe”
在安装`stompjs`的时候下载websocket模块报错
```bash
e:\IdeaProjects\net.tidebuy.sched\sched-vue\node_modules\websocket>if not defined npm_config_node_gyp (node "e:\node\node_global\node_modules\npm\bi
node-gyp-bin\\..\..\node_modules\node-gyp\bin\node-gyp.js" rebuild )  else (node "" rebuild )
Building the projects in this solution one at a time. To enable parallel build, please add the "/m" switch.
  错误: 项目文件“e:\IdeaProjects\net.tidebuy.sched\sched-vue\node_modules\websocket\build\bufferutil.vcproj”找不到或者不是有效的项目文件。
  该项目的所有配置项都需要系统提供对某些平台的支持，但在此计算机上没有安装这些平台。因此无法加载该项目。
MSBUILD : error MSB6006: “VCBuild.exe”已退出，代码为 -1。 [e:\IdeaProjects\net.tidebuy.sched\sched-vue\node_modules\websocket\build\binding.sln]
  错误: 项目文件“e:\IdeaProjects\net.tidebuy.sched\sched-vue\node_modules\websocket\build\validation.vcproj”找不到或者不是有效的项目文件。
  该项目的所有配置项都需要系统提供对某些平台的支持，但在此计算机上没有安装这些平台。因此无法加载该项目。
MSBUILD : error MSB6006: “VCBuild.exe”已退出，代码为 -1。 [e:\IdeaProjects\net.tidebuy.sched\sched-vue\node_modules\websocket\build\binding.sln]
```

原因是 node-gyp 需要调用相关的库，[官网中有依赖说明](https://github.com/nodejs/node-gyp)

在windows中安装[windows-build-tools](https://github.com/felixrieseberg/windows-build-tools)
```bash
npm install --global --production windows-build-tools
```

## `.npmrc`
```bash
# 在npm install过程中打印当前的请求下载日志
loglevel=http
```