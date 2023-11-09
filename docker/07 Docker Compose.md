## Compose 简介
Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

如果你还不了解 YML 文件配置，可以先阅读 YAML 入门教程。

Compose 使用的三个步骤：
- 使用 `Dockerfile` 定义应用程序的环境。
- 使用 `docker-compose.yml` 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 `docker-compose up` 命令来启动并运行整个应用程序。

docker-compose.yml 的配置案例如下（配置参数参考下文）：
```yaml
# yaml 配置实例
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

## Compose 安装
Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases。

运行以下命令以下载 Docker Compose 的当前稳定版本：

    $ sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

要安装其他版本的 Compose，请替换 v2.2.2。

Docker Compose 存放在 GitHub，不太稳定。

你可以也通过执行下面的命令，高速安装 Docker Compose。

    curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

将可执行权限应用于二进制文件：

    sudo chmod +x /usr/local/bin/docker-compose

创建软链：

    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

测试是否安装成功：

    docker-compose version

> 注意： 对于 alpine，需要以下依赖包： py-pip，python-dev，libffi-dev，openssl-dev，gcc，libc-dev，和 make。

### macOS
Mac 的 Docker 桌面版和 Docker Toolbox 已经包括 Compose 和其他 Docker 应用程序，因此 Mac 用户不需要单独安装 Compose。Docker 安装说明可以参阅 MacOS Docker 安装。

### windows PC
Windows 的 Docker 桌面版和 Docker Toolbox 已经包括 Compose 和其他 Docker 应用程序，因此 Windows 用户不需要单独安装 Compose。Docker 安装说明可以参阅 Windows Docker 安装。

## 使用
### 1、准备
创建一个测试目录：

    mkdir composetest
    cd composetest

在测试目录中创建一个名为 app.py 的文件，并复制粘贴以下内容：

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

在此示例中，redis 是应用程序网络上的 redis 容器的主机名，该主机使用的端口为 6379。

在 composetest 目录中创建另一个名为 requirements.txt 的文件，内容如下：

    flask
    redis

### 2、创建 Dockerfile 文件
在 composetest 目录中，创建一个名为 Dockerfile 的文件，内容如下：

```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

Dockerfile 内容解释：
- `FROM python:3.7-alpine`: 从 Python 3.7 映像开始构建镜像。
- `WORKDIR /code`: 将工作目录设置为 /code。
- 设置 flask 命令使用的环境变量。
  
        ENV FLASK_APP app.py
        ENV FLASK_RUN_HOST 0.0.0.0

- `RUN apk add --no-cache gcc musl-dev linux-headers`: 安装 gcc，以便诸如 MarkupSafe 和 SQLAlchemy 之类的 Python 包可以编译加速。
- 复制 requirements.txt 并安装 Python 依赖项。

        COPY requirements.txt requirements.txt
        RUN pip install -r requirements.txt

- `COPY . .`: 将 . 项目中的当前目录复制到 . 镜像中的工作目录。
- `CMD ["flask", "run"]`: 容器提供默认的执行命令为：flask run。

### 3、创建 docker-compose.yml
在测试目录中创建一个名为 docker-compose.yml 的文件，然后粘贴以下内容：

```yaml
# yaml 配置
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

该 Compose 文件定义了两个服务：web 和 redis。

- `web`：该 web 服务使用从 Dockerfile 当前目录中构建的镜像。然后，它将容器和主机绑定到暴露的端口 5000。此示例服务使用 Flask Web 服务器的默认端口 5000 。
- `redis`：该 redis 服务使用 Docker Hub 的公共 Redis 映像。

### 4、使用 Compose 命令构建和运行您的应用
在测试目录中，执行以下命令来启动应用程序：

    docker-compose up

如果你想在后台执行该服务可以加上 -d 参数：

    docker-compose up -d

![image](img/07_01.png)

## yml 配置指令参考
### version
指定本 yml 依从的 compose 哪个版本制定的。

### build
指定为构建镜像上下文路径：

例如 webapp 服务，指定为从上下文路径 ./dir/Dockerfile 所构建的镜像：
```yml
version: "3.7"
services:
  webapp:
    build: ./dir
```

或者，作为具有在上下文指定的路径的对象，以及可选的 Dockerfile 和 args：
```yml
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
      labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"
      target: prod
```

- `context`：上下文路径。
- `dockerfile`：指定构建镜像的 Dockerfile 文件名。
- `args`：添加构建参数，这是只能在构建过程中访问的环境变量。
- `labels`：设置构建镜像的标签。
- `target`：多层构建，可以指定构建哪一层。

### cap_add，cap_drop
添加或删除容器拥有的宿主机的内核功能。
```yml
cap_add:
  - ALL # 开启全部权限

cap_drop:
  - SYS_PTRACE # 关闭 ptrace权限
```


### cgroup_parent
为容器指定父 cgroup 组，意味着将继承该组的资源限制。
```yml
cgroup_parent: m-executor-abcd
```

### command
覆盖容器启动的默认命令。
```yml
command: ["bundle", "exec", "thin", "-p", "3000"]
```

### container_name
指定自定义容器名称，而不是生成的默认名称。
```yml
container_name: my-web-container
```

### depends_on
设置依赖关系。
- `docker-compose up` ：以依赖性顺序启动服务。在以下示例中，先启动 db 和 redis ，才会启动 web。
- `docker-compose up SERVICE` ：自动包含 SERVICE 的依赖项。在以下示例中，docker-compose up web 还将创建并启动 db 和 redis。
- `docker-compose stop` ：按依赖关系顺序停止服务。在以下示例中，web 在 db 和 redis 之前停止。

```yml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> 注意：web 服务不会等待 redis db 完全启动 之后才启动。

















