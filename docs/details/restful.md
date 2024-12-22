# RESTful API

以下是后端提供的Restful API，作为前后端沟通的桥梁

## 登录注册相关

API 前缀为 `/api`

### 登录

登录利用用户名和密码。登录成功后会设置 cookies 字段 `username` 和 `user_token`。在不退出登录的情况下，利用 cookies 进行鉴权。规定 `user_token` 不为空时，用户是登入的。

```shell
curl -X POST http://127.0.0.1:5000/api/login \
-H "Content-Type: application/json" \
-d '{"username": "admin", "password": "114514"}'
```

### 注册

```shell
curl -X POST http://127.0.0.1:5000/api/register \
-H "Content-Type: application/json" \
-d '{"username": "admin", "password": "114514","email":"admic@matrix.com"}'
// response with http-header: Set-Cookie
```

### 登出

不需要在消息体里面发送东西，后端会读取 cookies 中的字段查找相应的表项，更新状态。规定 `user_token` 为空时，用户是登出的。

```json
curl -X POST http://127.0.0.1:5000/api/logout \
-b cookies.txt
```

## 容器相关

API 前缀为 `/containers`

### 查看用户的所有容器

```sql
curl -X GET http://127.0.0.1:5000/containers -b cookies.txt
```

返回当前用户的所有容器的cid，示例：

```json
{
	"cid": [1, 2, 3]
}
```

### 查看容器状态

查看 `cid=1` 的容器状态

```json
curl -X GET http://127.0.0.1:5000/containers/status/1 \
	-b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
	"result": "running" / "stopped"
}
```

或者

```json
{
	"status": -1,
	"error": "no container or not authorized to visit"
}
```

### 获取指定容器的信息

获取 `cid=2` 的镜像信息

```json
curl -X GET http://127.0.0.1:5000/containers/property/2 \
	-b cookies.txt
```

返回示例

```json
{
  "status": 0,
  "name": "test3",
  "cpu": 4,
  "memory": 4096,
  "portssh": 7295,
  "portjupyter": 7685,
  "porttsb": 6255,
  "passwd": "dzgihSH45GKdry39",
  "ip": "172.18.198.207",
  "cpu_name": "Intel(R) Xeon(R) CPU E7-4830",
  "gpu_type": "无GPU"
}
```

### 创建容器

```json
curl -X POST http://127.0.0.1:5000/containers \
	-H "Content-Type: application/json" \
	-d '{"iid": 1, "name": "container_name", "cpu": 2, "mem": 2048, "gid": 1}' \
	-b cookies.txt
```

返回：

```json
{
	"status": 0, // 0表示成功
	"passwd": "n8OMvBQeVfYmgiun",
	"portssh": 6264, 
	"portjupyter": 7451, 
	"porttsb": 7491
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括：

- 'no image or not authorized to visit image’
- 'cpu or memory not enough’
- 'gpu {gid} not available’
- 'docker run failed’

### 启动容器

启动 `cid=1` 的容器

```json
curl -X PUT http://127.0.0.1:5000/containers/1 \
     -H "Content-Type: application/json" \
     -d '{"cmd": "start", "gid": 1}' \
     -b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括

- 'no container or not authorized to visit’
- 'container already running’
- "no container or not authorized to visit"
- 'cpu or memory not enough’
- 'gpu {gid} not available’

### 停止容器

停止 `cid=1` 的容器

```json
curl -X PUT http://127.0.0.1:5000/containers/1 \
	-H "Content-Type: application/json" \
	-d '{"cmd": "stop"}' \
	-b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括

- "no container or not authorized to visit"
- 'container already stopped’

### 删除容器

删除 `cid=1` 的容器

```json
curl -X DELETE http://127.0.0.1:5000/containers/1 \
     -b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括

- 'no container or not authorized to visit’

## 设备相关

API 前缀为 `/devices`

### 查看所有设备

```sql
curl -X GET http://127.0.0.1:5000/devices \
	-b cookies.txt
```

返回所有设备的did，示例：

```json
{
	"cid": [1, 2, 3]
}
```

### 获取设备属性

查询 `did=1` 的设备的属性

```json
curl -X GET http://127.0.0.1:5000/devices/property/1 \
	-b cookies.txt
```

返回设备属性，示例：

```json
{
  "total_cpu": 26,
  "used_cpu": 10,
  "cpu_name": "Intel(R) Core(TM) i7-14700K",
  "total_memory": 24576,
  "used_memory": 2048,
  "gpus": [
    {
      "gid": 1,
      "gpu_type": "RTX4060 Ti 16GB"
    }
  ],
  "gpu_used": [
    [1]
  ],
  "ip": "172.18.198.204"
}
```

images.User如果为public，代表镜像是公有的，不属于某个用户

## 镜像相关

API 前缀为 `/images`

### 获取用户自己创建的镜像

```json
curl -X GET http://127.0.0.1:5000/images \
     -b cookies.txt
```

返回示例

```json
{
	"iid": [2, 3]
}
```

这里返回的只是用户自己的镜像，不包括公有镜像

### 获取所有公开的镜像

```json
curl -X GET http://127.0.0.1:5000/images/public \
	-b cookies.txt
```

返回示例

```json
{
	"iid": [2, 3]
}
```

这里返回的只是用户自己的镜像，不包括公有镜像

### 获取指定镜像的信息

获取 `iid=1` 的镜像信息

```json
curl -X GET http://127.0.0.1:5000/images/property/1 \
	-b cookies.txt
```

返回示例

```json
{
  "did": 2,
  "name": "test03",
  "public": true,
  "user": "lsy",
  "size": 3520950239
}
```

user处为用户名，当镜像不属于某个用户时，填public

### 从容器创建镜像

从 `cid=1` 的容器创建镜像

```json
curl -X POST http://127.0.0.1:5000/images \
	-H "Content-Type: application/json" \
	-d '{"cid": 1, "name": "my_image"}' \
	-b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括

- 'no container or not authorized to visit’

### 删除镜像

删除 `iid=2` 的镜像

```json
curl -X DELETE http://127.0.0.1:5000/images/2 \
	-b cookies.txt
```

返回

```json
{
	"status": 0, // 0表示成功
}
```

或者

```json
{
	"status": -1,
	"error": "报错信息"
}
```

报错信息包括

- 'no image or not authorized to remove’
- 'some containers is based on the image, unable to remove image’