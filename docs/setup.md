## 安装

您可以通过多种途径来设置和运行 Paperless：

-   [使用脚本设置 Docker 安装](#docker_script)
-   [使用 Docker Compose 模板](#docker)
-   [自行构建 Docker 镜像](#docker_build)
-   [直接在您的系统上手动安装 Paperless-ngx（"裸机"安装）](#bare_metal)
-   用户维护的商业托管提供商列表可以在 [wiki](https://github.com/paperless-ngx/paperless-ngx/wiki/Related-Projects) 中找到。

Docker 方式快速且简单，是推荐的方法。
它会自动配置上述所有组件，使其正常工作，并为所有配置选项使用合理的默认值。
这里有一份 Docker 初学者的速查表：[CLI 基础](https://www.sehn.tech/refs/devops-with-docker/)。

裸机安装方式设置起来比较复杂，但如果您想贡献一些代码，这种方式会更方便。您需要自己配置和运行上述组件。

### 使用安装脚本 {#docker_script}

Paperless 提供了一个交互式安装脚本来设置 Docker Compose 安装。该脚本会询问一些配置选项，然后创建必要的配置文件、拉取 Docker 镜像、启动 Paperless-ngx 并创建您的超级用户账户。该脚本本质上自动执行了 [Docker 设置](#docker) 中描述的步骤。

1.  确保 Docker 和 Docker Compose 已[安装](https://docs.docker.com/engine/install/){:target="\_blank"}。

2.  下载并运行安装脚本：

    ```shell-session
    bash -c "$(curl --location --silent --show-error https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/main/install-paperless-ngx.sh)"
    ```

    !!! note

        macOS 用户需要安装支持以 `sed` 运行的 [gnu-sed](https://formulae.brew.sh/formula/gnu-sed) 以及 [wget](https://formulae.brew.sh/formula/wget)。

### 使用 Docker Compose {#docker}

1.  确保 Docker 和 Docker Compose 已[安装](https://docs.docker.com/engine/install/){:target="\_blank"}。

2.  转到项目页面的 [/docker/compose 目录](https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose){:target="\_blank"}，根据您想要使用的数据库后端，下载其中一个 `docker-compose.*.yml` 文件。将文件放在本地目录中，并将其重命名为 `docker-compose.yml`。在同一目录下也下载 `docker-compose.env` 文件和 `.env` 文件。

    如果您想启用对 Office 和其他文档的可选支持，请下载文件名中包含 `-tika` 的文件。

    !!! tip

        对于新安装，建议使用 PostgreSQL 作为数据库后端。

3.  根据需要修改 `docker-compose.yml`。例如，您可能希望更改消费目录、媒体目录等的路径以使用"绑定挂载"。
    找到指定挂载目录的行，例如：

    ```yaml
    - ./consume:/usr/src/paperless/consume
    ```

    将冒号*之前*的部分替换为您选择的本地目录：

    ```yaml
    - /home/jonaswinkler/paperless-inbox:/usr/src/paperless/consume
    ```

    您可能还想将 Web 服务器使用的默认端口（8000）更改为其他端口，例如端口 8010：

    ```yaml
    ports:
        - 8010:8000
    ```

    **无根模式**

    !!! warning

        如果通过 `PAPERLESS_OCR_LANGUAGES` 指定了额外的语言，目前无法以无根模式运行容器。

    如果您想以无根容器模式运行 Paperless，您需要在 `docker-compose.yml` 中执行以下操作：

    -   将运行容器的 `user` 设置为映射到容器内的 `paperless` 用户。此值（下面的 `user_id`）应与下一步中设置的 `USERMAP_UID` 和 `USERMAP_GID` 相同。请参阅[此处](configuration.md#docker)的 `USERMAP_UID` 和 `USERMAP_GID`。

    您的 Paperless 条目应包含类似以下内容：

    > ```
    > webserver:
    >   image: ghcr.io/paperless-ngx/paperless-ngx:latest
    >   user: <user_id>
    > ```

4.  使用您想要的任何配置选项修改 `docker-compose.env`。
    有关所有选项，请参阅[配置文档](configuration.md)。

    您可能还需要将 `USERMAP_UID` 和 `USERMAP_GID` 设置为主机系统上您的用户的 uid 和 gid。使用 `id -u` 和 `id -g` 来获取这些值。这确保了容器和主机用户都对消费目录具有写访问权限。如果您主机系统上的 UID 和 GID 是 1000（大多数系统上第一个普通用户的默认值），则无需任何修改即可开箱即用。运行 `id "用户名"` 来检查。

    !!! note

        您可以通过在配置值后附加 `_FILE` 来利用 Docker 密钥进行配置设置。例如，[`PAPERLESS_DBUSER`](configuration.md#PAPERLESS_DBUSER) 可以使用 `PAPERLESS_DBUSER_FILE=/var/run/secrets/password.txt` 来设置。

    !!! warning

        某些文件系统（如 NFS 网络共享）不支持使用 `inotify` 的文件系统通知。当将消费目录存储在此类文件系统上时，Paperless 将无法通过默认配置拾取新文件。您需要使用 [`PAPERLESS_CONSUMER_POLLING`](configuration.md#PAPERLESS_CONSUMER_POLLING)，这将禁用 inotify。请参阅[此处](configuration.md#polling)。

5.  运行 `docker compose pull`。默认情况下，这将从 GitHub 容器注册表拉取镜像，但您可以通过将 `image` 行更改为 `image: paperlessngx/paperless-ngx:latest` 来更改为从 Docker Hub 拉取镜像。

6.  运行 `docker compose up -d`。这将创建并启动必要的容器。

7.  恭喜！您的 Paperless-ngx 实例现在应该可以通过 `http://127.0.0.1:8000`（或类似地址，取决于您的配置）访问。当您首次访问 Web 界面时，系统将提示您创建超级用户账户。

### 自行构建 Docker 镜像 {#docker_build}

1.  克隆 Paperless 的整个仓库：

    ```shell-session
    git clone https://github.com/paperless-ngx/paperless-ngx
    ```

    主分支始终反映最新的稳定版本。

2.  根据您想要使用的数据库后端，将 `docker/compose/docker-compose.*.yml` 中的一个复制到根文件夹中的 `docker-compose.yml`。同样将 `docker-compose.env` 复制到项目根目录。

3.  在 `docker-compose.yml` 文件中，找到指示 Docker Compose 从 Docker Hub 拉取 Paperless 镜像的行：

    ```yaml
    webserver:
        image: ghcr.io/paperless-ngx/paperless-ngx:latest
    ```

    并将其替换为指示 Docker Compose 从当前工作目录构建镜像的行：

    ```yaml
    webserver:
        build:
            context: .
    ```

4.  按照上面的 [Docker 设置](#docker) 进行操作，但当要求运行 `docker compose pull` 来拉取镜像时，改为运行以下命令来构建镜像：

    ```shell-session
    docker compose build
    ```

### 裸机安装方式 {#bare_metal}

Paperless 仅在 Linux 上运行。以下过程已在 Debian/Buster 的最小化安装上测试过，这是撰写本文时的当前稳定版本。Windows 不受支持，也永远不会支持。

Paperless 需要 Python 3。目前，3.10 - 3.12 是经过测试的版本。
更新的版本可能可以工作，但某些依赖项可能不完全支持新版本。
随着旧版本 Python 达到生命周期终点或新版本发布、依赖项支持得到确认等，可能会停止对旧版本 Python 的支持。

1.  安装依赖项。Paperless 需要以下软件包。

    -   `python3`
    -   `python3-pip`
    -   `python3-dev`
    -   `default-libmysqlclient-dev` 用于 MariaDB
    -   `pkg-config` 用于 mysqlclient（Python 依赖项）
    -   `fonts-liberation` 用于为纯文本文件生成缩略图
    -   `imagemagick` >= 6 用于 PDF 转换
    -   `gnupg` 用于处理加密文档
    -   `libpq-dev` 用于 PostgreSQL
    -   `libmagic-dev` 用于 MIME 类型检测
    -   `mariadb-client` 用于 MariaDB 编译时
    -   `libzbar0` 用于条形码检测
    -   `poppler-utils` 用于条形码检测

    使用以下列表进行您首选的包管理：

    ```
    python3 python3-pip python3-dev imagemagick fonts-liberation gnupg libpq-dev default-libmysqlclient-dev pkg-config libmagic-dev libzbar0 poppler-utils
    ```

    这些依赖项是 OCRmyPDF 所需的，OCRmyPDF 用于文本识别。

    -   `unpaper`
    -   `ghostscript`
    -   `icc-profiles-free`
    -   `qpdf`
    -   `liblept5`
    -   `libxml2`
    -   `pngquant`（建议用于某些 PDF 图像优化）
    -   `zlib1g`
    -   `tesseract-ocr` >= 4.0.0 用于 OCR
    -   `tesseract-ocr` 语言包（`tesseract-ocr-eng`、`tesseract-ocr-deu` 等）

    使用以下列表进行您首选的包管理：

    ```
    unpaper ghostscript icc-profiles-free qpdf liblept5 libxml2 pngquant zlib1g tesseract-ocr
    ```

    在 Raspberry Pi 上，还需要这些库：

    -   `libatlas-base-dev`
    -   `libxslt1-dev`
    -   `mime-support`

    您还需要这些来安装一些 Python 依赖项：

    -   `build-essential`
    -   `python3-setuptools`
    -   `python3-wheel`

    使用以下列表进行您首选的包管理：

    ```
    build-essential python3-setuptools python3-wheel
    ```

2.  安装 `redis` >= 6.0 并将其配置为自动启动。

3.  可选。安装 `postgresql` 并为 Paperless 配置数据库、用户和密码。如果您不希望使用 PostgreSQL，MariaDB 和 SQLite 也可用。

    !!! note

        在使用 SQLite 的裸机安装中，请确保启用了 [JSON1 扩展](https://code.djangoproject.com/wiki/JSON1Extension)。通常情况如此，但并非总是如此。

4.  创建一个系统用户，并为其指定一个新的主目录，您希望在此用户下运行 Paperless。

    ```shell-session
    adduser paperless --system --home /opt/paperless --group
    ```

5.  从 <https://github.com/paperless-ngx/paperless-ngx/releases> 获取发布存档，例如使用：

    ```shell-session
    curl -O -L https://github.com/paperless-ngx/paperless-ngx/releases/download/v1.10.2/paperless-ngx-v1.10.2.tar.xz
    ```

    使用以下命令解压存档：

    ```shell-session
    tar -xf paperless-ngx-v1.10.2.tar.xz
    ```

    并将内容复制到您之前创建的用户的主文件夹（`/opt/paperless`）。

    可选：如果您克隆了 git 仓库，您将需要自己编译前端，请参阅[此处](development.md#front-end-development)并使用 `build` 步骤，而不是 `serve`。

6.  配置 Paperless。有关详细信息，请参阅[配置](configuration.md)。
    编辑包含的 `paperless.conf` 并根据您的需要调整设置。使 Paperless 运行所需的设置包括：

    -   [`PAPERLESS_REDIS`](configuration.md#PAPERLESS_REDIS) 应指向您的 Redis 服务器，例如 <redis://localhost:6379>。
    -   [`PAPERLESS_DBENGINE`](configuration.md#PAPERLESS_DBENGINE) 可选，应为 `postgres`、`mariadb` 或 `sqlite` 之一。
    -   [`PAPERLESS_DBHOST`](configuration.md#PAPERLESS_DBHOST) 应是运行 PostgreSQL 服务器的主机名。不要配置此项以使用 SQLite。同时根据需要配置端口、数据库名称、用户和密码。
    -   [`PAPERLESS_CONSUMPTION_DIR`](configuration.md#PAPERLESS_CONSUMPTION_DIR) 应指向 Paperless 应监视文档的文件夹。您可能希望将其放在其他地方。同样，[`PAPERLESS_DATA_DIR`](configuration.md#PAPERLESS_DATA_DIR) 和 [`PAPERLESS_MEDIA_ROOT`](configuration.md#PAPERLESS_MEDIA_ROOT) 定义了 Paperless 存储其数据的位置。如果您愿意，可以将两者指向同一目录。
    -   [`PAPERLESS_SECRET_KEY`](configuration.md#PAPERLESS_SECRET_KEY) 应是一个随机字符序列。它用于身份验证。不这样做会允许第三方伪造身份验证凭据。
    -   [`PAPERLESS_URL`](configuration.md#PAPERLESS_URL) 如果您在反向代理后面。这应指向您的域名。请参阅[配置](configuration.md)以获取更多信息。

    可以对 Paperless 进行更多调整，尤其是 OCR 部分。以下选项推荐给所有人：

    -   将 [`PAPERLESS_OCR_LANGUAGE`](configuration.md#PAPERLESS_OCR_LANGUAGE) 设置为您大多数文档所使用的语言。
    -   将 [`PAPERLESS_TIME_ZONE`](configuration.md#PAPERLESS_TIME_ZONE) 设置为您当地的时区。

    !!! warning

        确保您的 Redis 实例[是安全的](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)。

7.  如果以下目录缺失，请创建它们：

    -   `/opt/paperless/media`
    -   `/opt/paperless/data`
    -   `/opt/paperless/consume`

    如果您配置了不同的文件夹，请相应调整。
    确保 Paperless 用户对每个文件夹都有写权限，使用：

    ```shell-session
    ls -l -d /opt/paperless/media
    ```

    如果需要，使用以下命令更改所有者：

    ```shell-session
    sudo chown paperless:paperless /opt/paperless/media
    sudo chown paperless:paperless /opt/paperless/data
    sudo chown paperless:paperless /opt/paperless/consume
    ```

8.  从 `requirements.txt` 文件安装 Python 依赖项。

    ```shell-session
    sudo -Hu paperless pip3 install -r requirements.txt
    ```

    这将在新的 Paperless 用户的主目录中安装所有 Python 依赖项。

    !!! tip

        是否使用虚拟环境来管理 Python 依赖项由您决定。这是上述方法的替代方案，可能需要调整示例脚本以使用虚拟环境路径。

    !!! tip

        如果您使用现代的 Python 工具，例如 `uv`，安装将不包括 Postgres 或 MariaDB 的依赖项。您可以使用 `--extra <EXTRA>` 选择这些额外项，或使用 `--all-extras` 选择所有额外项。

9.  转到 `/opt/paperless/src`，并执行以下命令：

    ```bash
    # 这将创建数据库模式。
    sudo -Hu paperless python3 manage.py migrate
    ```

    当您首次访问 Web 界面时，系统将提示您创建超级用户账户。

10. 可选：通过执行以下命令测试 Paperless 是否正常工作：

    ```bash
    # 手动启动 Web 服务器
    sudo -Hu paperless python3 manage.py runserver
    ```

    如果从安装 Paperless 的同一设备访问，请将浏览器指向 http://localhost:8000。
    如果从另一台机器访问，请设置 systemd 服务。您可能需要设置 `PAPERLESS_DEBUG=true` 才能使开发服务器在浏览器中正常工作。

    !!! warning

        这是一个开发服务器，不应在生产环境中使用。
        它未经安全审计，性能也低于生产就绪的 Web 服务器。

    !!! tip

        这不会启动消费者。Paperless 在一个单独的进程中执行此操作。

11. 设置 systemd 服务以自动运行 Paperless。您可以使用 `scripts` 文件夹中包含的服务定义文件作为起点。

    Paperless 需要 `webserver` 脚本来运行 Web 服务器，需要 `consumer` 脚本来监视输入文件夹，需要 `taskqueue` 来处理文档消费等后台工作，需要 `scheduler` 脚本来在特定时间运行诸如电子邮件检查等任务。

    !!! note

        `socket` 脚本使 `granian` 能够在端口 80 上运行而无需 root 权限。为此，您需要在 `webserver` 脚本中取消注释 `Require=paperless-webserver.socket`，并配置 `granian` 监听端口 80（设置 `GRANIAN_PORT`）。

    这些服务依赖于 Redis 和可选的数据库服务器，但不需要按特定顺序启动。示例文件依赖于 Redis 已启动。如果您使用数据库服务器，则应添加额外的依赖项。

    !!! note

        有关使用反向代理的说明，请[参阅 wiki](https://github.com/paperless-ngx/paperless-ngx/wiki/Using-a-Reverse-Proxy-with-Paperless-ngx#)。

    !!! warning

        如果 celery 无法启动（使用 `sudo systemctl status paperless-task-queue.service` 检查 paperless-task-queue.service 和 paperless-scheduler.service），您需要更改文件中的路径。示例：
        `ExecStart=/opt/paperless/.local/bin/celery --app