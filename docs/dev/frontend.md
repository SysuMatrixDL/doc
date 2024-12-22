# 前端实现

前端采用 React 框架实现，作为算力市场平台，前端设计为尽可能动态地呈现数据起到监控资源的效果。

## vite 配置

读取 `BACKEND_SUFFIX` 环境变量作为后端 API 域名前缀。开发中设置该环境变量，vite 会将这个后端进行代理，避免跨域问题。在生产环境实际部署的时候，这个代理由 caddy 统一代理前后端完成。

`server.hmr.overlay` 设置为 `false` 禁用，是因为启用后暂未弄清楚的原因导致崩溃。

还有一个 `GRAFANA_URL` 环境变量本来是控制节点上的 grafana 服务用的，后来这一服务被删去所以弃用了。

## 路由

采用 [React-Router](https://reactrouter.com/) 实现路由。`app/routes.ts` 中路由配置如下：

```ts
export default [
  index("home/home.tsx"),
  route("login", "user/login.tsx"),
  route("register", "user/register.tsx"),
  layout("navi.tsx", [
    route("console", "console/console.tsx"),
    route("market", "market/market.tsx"),
    route("market/:did", "market/create.tsx"),
    route("community", "community/community.tsx")
  ]),
] satisfies RouteConfig;
```

## 组件库

主要使用了 [mantine](https://mantine.dev/) 组件库。

icon 主要使用了 mantine 所支持的 [tabler](https://tabler.io/icons) 组件库。

## CSS

主要使用 tailwind 风格的 CSS。

暂未实现深色模式的适配。

## 鉴权

两种登录方式:
- `/api/login` 密码登录, `login` 登录界面使用
- `/api/token_login` cookie 登录, `/home` 页面点击登录使用

在登录之后依靠用户的 cookie 进行鉴权登录，若 cookie 失效将跳转到登录界面。

在 `Navi` 导航栏的实现中每次渲染都先进行一次鉴权, `Navi` 组件作为 `/console`, `/market`, `/community` 这几个路由的 layout，每次访问这几个路由都会鉴权，实现当用户带着无效 cookie 访问这些网页自动跳转到登录。