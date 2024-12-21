# 后端控制器实现

## 脚本列表

返回的形式为json

### 容器控制

cid是容器编号,iid是镜像编号，uid是用户编号

| container_status.py | 输入cid，返回容器状态 |
| --- | --- |
| container_start.py | 输入cid、启动容器，如果硬件不够，会返回启动失败 |
| container_stop.py | 输入cid，停止容器 |
| container_rm.py | 输入cid，删除容器 |
| container_init.py | 输入iid、容器名、cpu内存配置、gpu编号，从镜像构建容器并启动，返回密码、端口等配置 |
| get_properties.get_containers | 输入uid，返回用户的所有容器的cid |

### 镜像控制

cid是容器编号,iid是镜像编号，uid是用户编号

| image_create.py | 输入uid和cid，保存容器为镜像 |
| --- | --- |
| image_rm.py | 输入uid和iid，销毁用户镜像 |
| get_properties.get_device_images | 输入did，输出这台设备上的所有镜像和所有者 |
| get_properties.get_user_images | 输入uid，返回用户创建的所有镜像的iid |

### 设备控制

cid是容器编号,iid是镜像编号,gid是gpu编号,did是设备编号，uid是用户编号

| get_properties.device_property | 输入did，获取当前设备cpu、cpu分配量、型号、内存、内存分配量、gpu、gpu分配量、所有可用镜像 |
| --- | --- |
| get_devices.py | 获取当前所有设备的did |
|  |  |

## RESTful API

## RESTful API

### 登录&注册

登录利用用户名和密码。登录成功后会设置 cookies 字段 `username` 和 `user_token`。在不退出登录的情况下，利用 cookies 进行鉴权。规定 `user_token` 不为空时，用户是登入的。

```shell
curl -X POST http://127.0.0.1:5000/api/login \
-H "Content-Type: application/json" \
-d '{"username": "admin", "password": "114514"}'

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

### 查看所有containers

get_properties.get_containers

```sql
curl -X GET http://127.0.0.1:5000/containers -b cookies.txt
```

返回当前用户的所有容器的cid，示例：

```json
{
	"cid": [1, 2, 3]
}
```

### 创建container

container_init.py

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

### 启动container

container_start.py

启动cid=1的容器

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

### 查看container状态

container_status.py

查看cid=1的容器状态

```json
curl -X GET http://127.0.0.1:5000/containers/1 \
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

### 停止container

container_stop.py

停止cid=1的容器

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

### 删除container

container_rm.py

删除cid=1的容器

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

### 查看所有设备

get_devices.py

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

查询did=1的设备的属性

```json
curl -X GET http://127.0.0.1:5000/devices/1 \
     -b cookies.txt
```

返回设备属性，示例：

```json
{
	"total_cpu" :  26,
	"used_cpu" :  2,
	"cpu_name" :  "Intel(R) Core(TM) i7-14700K",
	"total_memory" :  24576,
	"used_memory" :  2048,
	"gpus" :  [
							{"gid": 1, "name": "RTX4060 Ti 16GB"}
						],
	"gpu_used" :  [1],
	"images" :  [
								{"iid": 1, "User": "public"},
								{"iid": 2, "User": "CJL"}
							],
}
```

images.User如果为public，代表镜像是公有的，不属于某个用户

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

### 获取指定镜像的信息

获取iid=1的镜像信息

```json
curl -X GET http://127.0.0.1:5000/images/1 \
     -b cookies.txt
```

返回示例

```json
{
	"iid": 1, 
	"did": 1, 
	"name": "pytorch2.4.0-cuda11.8-cudnn9", 
	"user": "public"
}
```

user处为用户名，当镜像不属于某个用户时，填public

### 从容器创建镜像

从cid=1的容器创建镜像

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

删除iid=2的镜像

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