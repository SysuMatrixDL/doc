# 开发

所有开发仓库均开源于 [SysuMatrixDL](https://github.com/SysuMatrixDL) 组织下，有若干仓库如下：
- [doc](https://github.com/SysuMatrixDL/doc) 本文档的仓库
- [deploy](https://github.com/SysuMatrixDL/deploy) 控制节点的部署
- [worker](https://github.com/SysuMatrixDL/worker) 工作节点的部署
- [backend](https://github.com/SysuMatrixDL/backend) 后端控制器实现及 API 封装
- [frontend](https://github.com/SysuMatrixDL/frontend) 前端实现

[frontend](https://github.com/SysuMatrixDL/frontend) 和 [backend](https://github.com/SysuMatrixDL/backend) 均使用 github actions 自动化打包上传镜像到 ghcr.io/sysumatrixdl 下。它们的镜像发布为:
- ghcr.io/sysumatrixdl/frontend
- ghcr.io/sysumatrixdl/backend

开发和部署中还需要两个镜像为:
- ghcr.io/sysumatrixdl/opengauss
- ghcr.io/sysumatrixdl/grafana
