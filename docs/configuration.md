# 配置

Paperless 提供了广泛的定制选项。根据您运行 Paperless 的方式，这些设置需要在不同的地方定义。

某些配置选项可以通过用户界面设置。目前这包括常见的 [OCR](#ocr) 相关设置和一些前端设置。如果设置了这些选项，它们将优先于通过环境变量进行的设置。如果未设置，则将使用环境设置或适用的默认值。

-   如果您在 Docker 上运行 Paperless，则不使用 `paperless.conf`。
    相反，请通过将必要的选项复制到 `docker-compose.env` 来配置 Paperless。

-   如果您在其他任何环境中运行 Paperless，Paperless 将在以下位置搜索配置文件，并使用找到的第一个文件：
    -   环境变量 `PAPERLESS_CONFIGURATION_PATH`
    -   `/path/to/paperless/paperless.conf`
    -   `/etc/paperless.conf`
    -   `/usr/local/etc/paperless.conf`

## 必需服务

### Redis 代理

#### [`PAPERLESS_REDIS=<url>`](#PAPERLESS_REDIS) {#PAPERLESS_REDIS}

: 这是处理计划任务（如电子邮件获取、索引优化以及训练自动文档匹配器）所必需的。

    -   如果您的 Redis 服务器需要登录凭据，则 PAPERLESS_REDIS = `redis://<用户名>:<密码>@<主机>:<端口>`
    -   使用 requirepass 选项时，PAPERLESS_REDIS = `redis://:<密码>@<主机>:<端口>`
    -   要包含 Redis 数据库索引，PAPERLESS_REDIS = `redis://<用户名>:<密码>@<主机>:<端口>/<数据库索引>`

    [有关保护 Redis 实例的更多信息](https://redis.io/docs/latest/operate/oss_and_stack/management/security)。

    默认为 `redis://localhost:6379`。

#### [`PAPERLESS_REDIS_PREFIX=<前缀>`](#PAPERLESS_REDIS_PREFIX) {#PAPERLESS_REDIS_PREFIX}

: 在 Redis 中用于键和通道的前缀。对于在多个 Paperless 实例之间共享一个 Redis 服务器很有用。

    默认为无前缀。

### 数据库

默认情况下，Paperless 使用 **SQLite**，数据库存储在 `data/db.sqlite3`。
要切换到 **PostgreSQL** 或 **MariaDB**，请设置 [`PAPERLESS_DBHOST`](#PAPERLESS_DBHOST) 并可选择配置其他与数据库相关的环境变量。

#### [`PAPERLESS_DBHOST=<主机名>`](#PAPERLESS_DBHOST) {#PAPERLESS_DBHOST}

: 如果未设置，Paperless 默认使用 **SQLite**。

    设置 `PAPERLESS_DBHOST` 以切换到 PostgreSQL 或 MariaDB。

#### [`PAPERLESS_DBENGINE=<引擎名称>`](#PAPERLESS_DBENGINE) {#PAPERLESS_DBENGINE}

: 可选。指定连接到远程数据库时要使用的数据库引擎。
可用选项是 `postgresql` 和 `mariadb`。

    如果设置了 `PAPERLESS_DBHOST`，则默认为 `postgresql`。

    !!! 警告

        使用 MariaDB 有一些注意事项。请参阅 [MySQL 注意事项](advanced_usage.md#mysql-caveats)。

#### [`PAPERLESS_DBPORT=<端口>`](#PAPERLESS_DBPORT) {#PAPERLESS_DBPORT}

: 连接到 PostgreSQL 或 MariaDB 时使用的端口。

    PostgreSQL 默认为 `5432`，MariaDB 默认为 `3306`。

#### [`PAPERLESS_DBNAME=<名称>`](#PAPERLESS_DBNAME) {#PAPERLESS_DBNAME}

: 使用 PostgreSQL 或 MariaDB 时要连接的数据库名称。

    默认为 "paperless"。

#### [`PAPERLESS_DBUSER=<名称>`](#PAPERLESS_DBUSER) {#PAPERLESS_DBUSER}

: 用于 PostgreSQL 或 MariaDB 数据库身份验证的用户名。

    默认为 "paperless"。

#### [`PAPERLESS_DBPASS=<密码>`](#PAPERLESS_DBPASS) {#PAPERLESS_DBPASS}

: PostgreSQL 或 MariaDB 数据库用户的密码。

    默认为 "paperless"。

#### [`PAPERLESS_DBSSLMODE=<模式>`](#PAPERLESS_DBSSLMODE) {#PAPERLESS_DBSSLMODE}

: 连接到 PostgreSQL 或 MariaDB 时使用的 SSL 模式。

    请参阅 [PostgreSQL 关于 sslmode 的官方文档](https://www.postgresql.org/docs/current/libpq-ssl.html)。

    请参阅 [MySQL 和 MariaDB 关于 sslmode 的官方文档](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-mode)。

    *注意*：PostgreSQL 和 MariaDB 的 SSL 模式值不同。

    PostgreSQL 默认为 `prefer`，MariaDB 默认为 `PREFERRED`。

#### [`PAPERLESS_DBSSLROOTCERT=<ca路径>`](#PAPERLESS_DBSSLROOTCERT) {#PAPERLESS_DBSSLROOTCERT}

: 用于验证数据库服务器的 SSL 根证书的路径。

    请参阅 [PostgreSQL 关于 sslmode 的官方文档](https://www.postgresql.org/docs/current/libpq-ssl.html)。
    更改 `root.crt` 的位置。

    请参阅 [MySQL 和 MariaDB 关于 sslmode 的官方文档](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-ca)。

    默认为未设置，使用主目录中的标准位置。

#### [`PAPERLESS_DBSSLCERT=<客户端证书路径>`](#PAPERLESS_DBSSLCERT) {#PAPERLESS_DBSSLCERT}

: 安全连接时使用的客户端 SSL 证书的路径。

    请参阅 [PostgreSQL 关于 sslmode 的官方文档](https://www.postgresql.org/docs/current/libpq-ssl.html)。

    请参阅 [MySQL 和 MariaDB 关于 sslmode 的官方文档](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-cert)。

    更改 `postgresql.crt` 的位置。

    默认为未设置，使用主目录中的标准位置。

#### [`PAPERLESS_DBSSLKEY=<客户端证书密钥>`](#PAPERLESS_DBSSLKEY) {#PAPERLESS_DBSSLKEY}

: 安全连接时使用的客户端 SSL 私钥的路径。

    请参阅 [PostgreSQL 关于 sslmode 的官方文档](https://www.postgresql.org/docs/current/libpq-ssl.html)。

    请参阅 [MySQL 和 MariaDB 关于 sslmode 的官方文档](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-key)。

    更改 `postgresql.key` 的位置。

    默认为未设置，使用主目录中的标准位置。

#### [`PAPERLESS_DB_TIMEOUT=<整数>`](#PAPERLESS_DB_TIMEOUT) {#PAPERLESS_DB_TIMEOUT}

: 设置数据库连接在超时前应等待多长时间。

    对于 SQLite，这设置了数据库被锁定时等待的时间。
    对于 PostgreSQL 或 MariaDB，这设置了连接超时。

    默认为未设置，使用 Django 的内置默认值。

#### [`PAPERLESS_DB_POOLSIZE=<整数>`](#PAPERLESS_DB_POOLSIZE) {#PAPERLESS_DB_POOLSIZE}

: 定义池中保留的最大数据库连接数。

    仅适用于 PostgreSQL。对于其他数据库引擎，此设置将被忽略。

    该值必须大于或等于 1 才能使用。
    默认为未设置，这将禁用连接池。

    !!! 注意

        每个工作进程 8-10 个连接的池通常就足够了。
        如果您遇到诸如 `couldn't get a connection` 或数据库连接超时之类的错误消息，您可能需要增加池大小。

    !!! 警告
        确保您的 PostgreSQL `max_connections` 设置足够大以处理连接池：
        `(NB_PAPERLESS_WORKERS + NB_CELERY_WORKERS) × POOL_SIZE + SAFETY_MARGIN`。例如，使用
        4 个 Paperless 工作进程和 2 个 Celery 工作进程，池大小为 8：``(4 + 2) × 8 + 10 = 58`，
        因此 `max_connections = 60`（或更多）是合适的。

        这假设只有 Paperless-ngx 连接到您的 PostgreSQL 实例。如果您有其他应用程序，
        则应相应地增加 `max_connections`。

#### [`PAPERLESS_DB_READ_CACHE_ENABLED=<布尔值>`](#PAPERLESS_DB_READ_CACHE_ENABLED) {#PAPERLESS_DB_READ_CACHE_ENABLED}

: 将数据库读取查询结果缓存到 Redis 中。这可以通过缓存数据库查询来显著提高应用程序响应时间，代价是内存使用量略有增加。

    默认为 `false`。

    !!! 危险

        **不要在应用程序运行时从外部修改数据库。**
        这包括诸如恢复备份、升级数据库或执行手动插入等操作。所有外部修改必须**仅在应用程序停止时**进行。
        进行任何此类更改后，您**必须使用 `invalidate_cachalot` 管理命令使数据库读取缓存失效**。

#### [`PAPERLESS_READ_CACHE_TTL=<整数>`](#PAPERLESS_READ_CACHE_TTL) {#PAPERLESS_READ_CACHE_TTL}

: 指定读取的数据应缓存多长时间（以秒为单位）。

    允许的值在 `1`（一秒）到 `31536000`（一年）之间。默认为 `3600`（一小时）。

    !!! 警告

        高 TTL 会随着时间的推移增加内存使用量。即使使用 `invalidate_cachalot` 命令使缓存失效，内存也可能在 TTL 结束前一直被占用。

在内存不足 (OOM) 的情况下，Redis 可能会停止接受新数据——包括缓存条目、计划任务和要消费的文档。
如果您的系统 RAM 有限，请考虑为读取缓存配置一个专用的 Redis 实例，并设置内存限制和驱逐策略为 `allkeys-lru`。
有关更多详细信息，请参阅 [Redis 驱逐策略文档](https://redis.io/docs/latest/develop/reference/eviction/)，并查看 `PAPERLESS_READ_CACHE_REDIS_URL` 设置以指定单独的 Redis 代理。

#### [`PAPERLESS_READ_CACHE_REDIS_URL=<url>`](#PAPERLESS_READ_CACHE_REDIS_URL) {#PAPERLESS_READ_CACHE_REDIS_URL}

: 定义用于读取缓存的 Redis 实例。

    默认为 `None`。

    !!! 注意
    如果未设置此值，用于计划任务的同一 Redis 实例也将用于缓存。

## 可选服务

### Tika {#tika}

Paperless 可以利用 [Tika](https://tika.apache.org/) 和
[Gotenberg](https://gotenberg.dev/) 来解析和转换
"Office" 文档（例如 ".doc"、".xlsx" 和 ".odt"）。
Tika 和 Gotenberg 也是允许解析电子邮件 (.eml) 所必需的。

如果您希望使用此功能，必须提供 Tika 服务器和 Gotenberg 服务器，
配置它们的端点，并启用该功能。

#### [`PAPERLESS_TIKA_ENABLED=<布尔值>`](#PAPERLESS_TIKA_ENABLED) {#PAPERLESS_TIKA_ENABLED}

: 启用（或禁用）Tika 解析器。

    默认为 false。

#### [`PAPERLESS_TIKA_ENDPOINT=<url>`](#PAPERLESS_TIKA_ENDPOINT) {#PAPERLESS_TIKA_ENDPOINT}

: 设置 Paperless 可以访问您的 Tika 服务器的端点 URL。

    默认为 "<http://localhost:9998>"。

#### [`PAPERLESS_TIKA_GOTENBERG_ENDPOINT=<url>`](#PAPERLESS_TIKA_GOTENBERG_ENDPOINT) {#PAPERLESS_TIKA_GOTENBERG_ENDPOINT}

: 设置 Paperless 可以访问您的 Gotenberg 服务器的端点 URL。

    默认为 "<http://localhost:3000>"。

如果您在 Docker 上运行 Paperless，可以将这些服务添加到
Docker Compose 文件中（请参阅提供的
[`docker-compose.sqlite-tika.yml`](https://github.com/paperless-ngx/paperless-ngx/blob/main/docker/compose/docker-compose.sqlite-tika.yml)
文件作为参考）。

将所有三个配置参数添加到您的配置中。如果使用
Docker，这可能是 Web 服务器的 `environment` 键或
`docker-compose.env` 文件。裸机安装可能有一个包含配置参数的 `.conf` 文件。
请确保使用正确的格式，并在编辑 YAML 文件时注意缩进。

### 电子邮件解析

#### [`PAPERLESS_EMAIL_PARSE_DEFAULT_LAYOUT=<整数>`](#PAPERLESS_EMAIL_PARSE_DEFAULT_LAYOUT) {#PAPERLESS_EMAIL_PARSE_DEFAULT_LAYOUT}

: 用作文档消费的电子邮件的默认布局。必须是以下整数选项之一。请注意，邮件
规则可以指定此设置，因此此回退用于默认选择以及通过其他方式消费的 .eml 文件。

    - `1` = 文本，然后是 HTML
    - `2` = HTML，然后是文本
    - `3` = 仅 HTML
    - `4` = 仅文本

## 路径和文件夹

#### [`PAPERLESS_CONSUMPTION_DIR=<路径>`](#PAPERLESS_CONSUMPTION_DIR) {#PAPERLESS_CONSUMPTION_DIR}

: 这是您的文档应放置以被消费的位置。确保在启动 Paperless 之前，
该目录存在并且运行 Paperless 服务的用户可以读取/写入其内容。

    使用 Docker 时不要更改此设置，因为它只更改容器内的路径。
    请改为在 docker-compose.yml 文件中更改本地消费目录。

    默认为 "../consume/"，相对于 "src" 目录。

#### [`PAPERLESS_DATA_DIR=<路径>`](#PAPERLESS_DATA_DIR) {#PAPERLESS_DATA_DIR}

: 这是 Paperless 存储其所有数据（搜索索引、SQLite
数据库、分类模型等）的位置。

    默认为 "../data/"，相对于 "src" 目录。

#### [`PAPERLESS_EMPTY_TRASH_DIR=<路径>`](#PAPERLESS_EMPTY_TRASH_DIR) {#PAPERLESS_EMPTY_TRASH_DIR}

: 当文档被删除时（例如，清空回收站后），原始文件将被移动到这里
而不是从文件系统中删除。仅保留原始版本。

    运行 Paperless 的用户必须对此目录具有写入权限。在 Docker 内部运行时，
请确保此路径位于永久卷内（例如 "../media/trash"），以便在升级时不会丢失。

    请注意，在使用此设置之前，目录必须存在。

    默认为空（即真正删除文件）。

    此设置以前名为 PAPERLESS_TRASH_DIR。

#### [`PAPERLESS_MEDIA_ROOT=<路径>`](#PAPERLESS_MEDIA_ROOT) {#PAPERLESS_MEDIA_ROOT}

: 这是存储您的文档和缩略图的位置。

    您可以将此设置和 PAPERLESS_DATA_DIR 设置为同一文件夹，以便让
    Paperless 将其所有数据存储在同一卷内。

    默认为 "../media/"，相对于 "src" 目录。

#### [`PAPERLESS_STATICDIR=<路径>`](#PAPERLESS_STATICDIR) {#PAPERLESS_STATICDIR}

: 在此处覆盖默认的 STATIC_ROOT。这是使用 "collectstatic" 管理命令创建的所有静态
文件的存储位置。

    除非您在做一些特殊的事情，否则无需覆盖此设置。
    如果更改了此设置，您可能需要再次运行 `collectstatic`。

    默认为 "../static/"，相对于 "src" 目录。

#### [`PAPERLESS_FILENAME_FORMAT=<格式>`](#PAPERLESS_FILENAME_FORMAT) {#PAPERLESS_FILENAME_FORMAT}

: 更改 Paperless 用于在媒体目录中存储文档的文件名。有关详细信息，请参阅 [文件名处理](advanced_usage.md#file-name-handling)。

    默认为无，这将禁用此功能。

#### [`PAPERLESS_FILENAME_FORMAT_REMOVE_NONE=<布尔值>`](#PAPERLESS_FILENAME_FORMAT_REMOVE_NONE) {#PAPERLESS_FILENAME_FORMAT_REMOVE_NONE}

: 告诉 Paperless 将 `PAPERLESS_FILENAME_FORMAT` 中解析为
'none' 的占位符替换为从结果文件名中省略。这也适用于
目录名。有关详细信息，请参阅 [文件名处理](advanced_usage.md#empty-placeholders)。

    默认为 `false`，这将禁用此功能。

#### [`PAPERLESS_LOGGING_DIR=<路径>`](#PAPERLESS_LOGGING_DIR) {#PAPERLESS_LOGGING_DIR}

: 这是 Paperless 将存储日志文件的位置。

    默认为 `PAPERLESS_DATA_DIR/log/`。

#### [`PAPERLESS_NLTK_DIR=<路径>`](#PAPERLESS_NLTK_DIR) {#PAPERLESS_NLTK_DIR}

: 这是 Paperless 将搜索 NLTK 处理所需数据的位置（如果您正在使用它）。如果您使用 Docker 镜像，
则不应更改此设置，因为数据已包含在镜像中。

以前，位置默认为 `PAPERLESS_DATA_DIR/nltk`。
除非您在裸机安装或其他设置中使用此功能，
否则不再需要此文件夹，可以手动删除。

默认为 `/usr/share/nltk_data`

#### [`PAPERLESS_MODEL_FILE=<路径>`](#PAPERLESS_MODEL_FILE) {#PAPERLESS_MODEL_FILE}

: 这是 Paperless 将存储分类模型的位置。

    默认为 `PAPERLESS_DATA_DIR/classification_model.pickle`。

## 日志记录

