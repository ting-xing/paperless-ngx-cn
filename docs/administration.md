# 管理

## 创建备份 {#backup}

根据您安装 Paperless 的方式，有多种创建 Paperless 实例备份的选项。

在创建备份之前，最好确保 Paperless 当时没有正在处理文档。

适用于任何 Paperless 安装的选项：

-   使用[文档导出器](#exporter)。文档导出器会将您的所有文档、缩略图、元数据和数据库内容导出到特定文件夹。您可以将文档和设置重新导入到新的 Paperless 实例中，或者使用此导出将文档存储到另一个 DMS 中。

    文档导出器还能够更新已存在的导出。因此，使用 `rsync` 进行增量备份是完全可行的。

    导出器不包含 API 令牌，导入后需要重新生成。

!!! caution

    您无法将使用一个 Paperless 版本生成的导出导入到另一个不同版本的 Paperless 中。导出包含数据库的精确镜像，而数据库迁移可能会更改数据库结构。

适用于 Docker 安装的选项：

-   备份 Docker 卷。这些卷通常位于主机的 `/var/lib/docker/volumes` 目录下，您需要 root 权限才能访问它们。

    Paperless 使用 4 个卷：

    -   `paperless_media`：这是存储文档的位置。
    -   `paperless_data`：这是存储辅助数据的位置。如果您使用 SQLite，此文件夹也包含 SQLite 数据库。
    -   `paperless_pgdata`：仅在使用 PostgreSQL 时存在，包含数据库。
    -   `paperless_dbdata`：仅在使用 MariaDB 时存在，包含数据库。

适用于裸机和非 Docker 安装的选项：

-   备份整个 Paperless 文件夹。这确保了如果您的 Paperless 实例在某个时刻崩溃或磁盘故障，您可以简单地将文件夹复制回原位，它就能正常工作。

    当使用 PostgreSQL 或 MariaDB 时，您还需要备份数据库。

### 恢复 {#migrating-restoring}

如果您使用[文档导出器](#exporter)备份了 Paperless-ngx，则可以使用[文档导入器](#importer)轻松恢复。

当然，其他备份策略需要恢复您在上述步骤中创建的任何卷、文件夹和数据库副本。

## 更新 Paperless {#updating}

### Docker 方式 {#docker-updating}

如果有新的 Paperless-ngx 版本可用，升级方式取决于您最初安装 Paperless-ngx 的方式。发布版本可在[发布页面](https://github.com/paperless-ngx/paperless-ngx/releases)找到。

首先，确保没有正在运行的活跃进程（如消费），然后[创建备份](#backup)。

之后，确保 Paperless 已停止：

```shell-session
$ cd /path/to/paperless
$ docker compose down
```

1.  如果您从 Docker Hub 拉取镜像，您只需要：

    ```shell-session
    docker compose pull
    docker compose up
    ```

    Docker Compose 文件引用 `latest` 版本，该版本始终是最新的稳定版本。

1.  如果您自己构建镜像，请执行以下操作：

    ```shell-session
    git pull
    docker compose build
    docker compose up
    ```

运行 `docker compose up` 也会应用任何新的数据库迁移。
如果您看到一切正常，请按一次 CTRL+C 以优雅地停止 Paperless。然后您可以使用 `-d` 参数启动 Paperless-ngx，使其在后台运行。

!!! note

    在版本 0.9.14 中，更新过程发生了变化。在 0.9.13 及更早版本中，Docker Compose 文件指定了确切的版本，`pull` 不会自动更新到新版本。为了启用如上所述的更新，要么从[此处](https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose)获取新的 `docker-compose.yml` 文件，要么编辑 `docker-compose.yml` 文件，找到以下行：

    ```
    image: ghcr.io/paperless-ngx/paperless-ngx:0.9.x
    ```

    并将版本替换为 `latest`：

    ```
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    ```

!!! note

    从版本 1.7.1 开始，Docker 镜像现在可以固定到某个发布系列。这通常与自动更新工具（如 Watchtower）结合使用，以允许仅安全地无人值守升级到新的错误修复版本。仍然建议在升级前始终查看发布说明。要将您的安装固定到某个发布系列，请编辑 `docker-compose.yml`，找到以下行：

    ```
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    ```

    并将版本替换为您想要跟踪的系列，例如：

    ```
    image: ghcr.io/paperless-ngx/paperless-ngx:1.7
    ```

### 裸机方式 {#bare-metal-updating}

获取新版本并解压内容后，执行以下操作：

1.  更新依赖项。新的 Paperless 版本可能需要额外的依赖项。所需的依赖项列在[裸机安装](setup.md#bare_metal)部分。

2.  更新 Python 依赖项。请记住，如果您使用虚拟环境，请在此之前激活它。

    ```shell-session
    pip install -r requirements.txt
    ```

    !!! note

        有时，某些依赖项会从 requirements.txt 中移除。比较版本并移除不再需要的依赖项将保持您的系统或虚拟环境清洁，并防止可能的冲突。

3.  迁移数据库。

    ```shell-session
    cd src
    python3 manage.py migrate # (1)
    ```

    1.  可能需要包含 `sudo -Hu <paperless_user>`

    这可能实际上不会执行任何操作。并非每个新的 Paperless 版本都附带新的数据库迁移。

### 数据库升级

Paperless-ngx 与 Django 支持的 PostgreSQL 和 MariaDB 版本兼容，通常可以安全地将它们更新到新版本。但是，您应该始终进行备份，并按照数据库文档中的说明进行主要版本之间的升级。

!!! note

    从 Paperless-ngx v2.18 开始，PostgreSQL 的最低支持版本是 14。

对于 PostgreSQL，请参考[升级 PostgreSQL 集群](https://www.postgresql.org/docs/current/upgrading.html)。

对于 MariaDB，请参考[升级 MariaDB](https://mariadb.com/kb/en/upgrading/)。

您也可以在创建了更新版本的 PostgreSQL 或 MariaDB 数据库后，使用带有 `--data-only` 标志的导出器和导入器。

!!! warning

    执行此操作时，不应更改任何设置，尤其是路径，否则存在数据丢失的风险。

## 管理工具 {#management-commands}

Paperless 附带一些管理命令，用于在您的 Paperless 实例上执行各种维护任务。您可以通过以下方式调用这些命令：

使用 Docker Compose，在 Paperless 运行时：

```shell-session
$ cd /path/to/paperless
$ docker compose exec webserver <command> <arguments>
```

使用 Docker，在 Paperless 运行时：

```shell-session
$ docker exec -it <container-name> <command> <arguments>
```

裸机：

```shell-session
$ cd /path/to/paperless/src
$ python3 manage.py <command> <arguments> # (1)
```

1.  可能需要包含 `sudo -Hu <paperless_user>`

所有命令都有内置帮助，可以通过使用 `--help` 参数执行它们来访问。

### 文档导出器 {#exporter}

文档导出器将您的所有数据（包括设置和数据库内容）从 Paperless 导出到一个文件夹中，用于备份或迁移到另一个 DMS。

如果您在 cronjob 中使用文档导出器来备份数据，可以在 exec 后面使用 `-T` 标志来抑制 "The input device is not a TTY" 错误。例如：`docker compose exec -T webserver document_exporter ../export`

```
document_exporter target [-c] [-d] [-f] [-na] [-nt] [-p] [-sm] [-z]

可选参数：
-c,  --compare-checksums
-cj, --compare-json
-d,  --delete
-f,  --use-filename-format
-na, --no-archive
-nt, --no-thumbnail
-p,  --use-folder-prefix
-sm, --split-manifest
-z,  --zip
-zn, --zip-name
--data-only
--no-progress-bar
--passphrase
```

`target` 是数据写入的文件夹。这包括文档、缩略图和一个 `manifest.json` 文件。清单包含数据库中的所有元数据（通信者、标签等）。

当您使用提供的 Docker Compose 脚本时，请指定 `../export` 作为目标。容器内的此路径会自动挂载到您主机上的 `export` 文件夹。

如果目标目录已存在并包含文件，Paperless 将假定导出目录的内容是之前的导出，并尝试更新先前的导出。Paperless 只会导出已更改和新增的文件。Paperless 通过检查文件属性"修改日期/时间"和"大小"来确定文件是否已更改。如果这对您不起作用，请指定 `-c` 或 `--compare-checksums`，Paperless 将尝试比较文件校验和。这速度较慢。清单和元数据 JSON 文件总是会更新，除非指定了 `cj` 或 `--compare-json`。

Paperless 不会删除导出目录中的任何现有文件。如果您希望 Paperless 也删除不属于当前导出的文件（例如已删除文档的文件），请指定 `-d` 或 `--delete`。将 Paperless 指向已包含其他文件的目录时要小心。

此命令生成的文件名遵循格式 `[创建日期] [通信者] [标题].[扩展名]`。如果您希望 Paperless 使用 [`PAPERLESS_FILENAME_FORMAT`](configuration.md#PAPERLESS_FILENAME_FORMAT) 作为导出的文件名，请指定 `-f` 或 `--use-filename-format`。

如果提供了 `-na` 或 `--no-archive`，则不会导出归档文件，只导出原始文件。

如果提供了 `-nt` 或 `--no-thumbnail`，则不会导出缩略图文件。

!!! note

    当使用 `-na`/`--no-archive` 或 `-nt`/`--no-thumbnail` 选项时，导出器将不会输出这些文件进行备份。导入后，[完整性检查器](#sanity-checker) 会警告缺少缩略图和归档文件，直到使用 `document_thumbnails` 或 [`document_archiver`](#archiver) 重新生成它们。从备份中省略这些文件可能是有意义的，因为它们的内容和校验和可能会更改（新的归档算法），并可能导致去重备份中额外的空间使用。

如果提供了 `-p` 或 `--use-folder-prefix`，文件将根据其性质导出到专用文件夹：`archive`、`originals`、`thumbnails` 或 `json`。

如果提供了 `-sm` 或 `--split-manifest`，文档信息将放置在单独的 JSON 文件中，而不是单个 JSON 文件中。主要的 manifest.json 仍将包含应用程序范围的信息（例如标签、通信者、文档类型等）。

如果提供了 `-z` 或 `--zip`，导出将是目标目录中的一个 zip 文件，根据当前本地日期或 `-zn` 或 `--zip-name` 中设置的值命名。

如果提供了 `--data-only`，则只导出数据库。此选项旨在方便数据库升级，而无需从媒体目录中清理文档和缩略图。

如果提供了 `--no-progress-bar`，进度条将被隐藏，使导出器静默运行。此选项对于脚本场景很有用，例如将导出器与 `crontab` 一起使用时。

如果提供了 `--passphrase`，它将用于加密导出中的某些字段。导入时必须提供此值。如果此值丢失，则无法导入导出。

!!! warning

    如果使用文件名格式导出，可能会由于操作系统的最大路径长度而导致错误。尝试调整导出目标，或者考虑不使用文件名格式。

### 文档导入器 {#importer}

文档导入器接收由[文档导出器](#exporter)生成的导出，并将其导入到 Paperless 中。

导入器的工作方式与导出器类似。您将其指向一个目录或生成的 .zip 文件，脚本会完成其余工作：

```shell
document_importer source
```

| 选项                | 必需 | 默认值 | 描述                                                                 |
| ------------------- | -------- | ------- | ------------------------------------------------------------------------- |
| source              | 是       | N/A     | 包含导出的目录                                                           |
| `--no-progress-bar` | 否       | False   | 如果提供，进度条将被隐藏                                                  |
| `--data-only`       | 否       | False   | 如果提供，仅导入数据，不导入文档文件或缩略图                              |
| `--passphrase`      | 否       | N/A     | 如果您的导出使用了密码短语加密，则必须提供                                |

当您使用提供的 Docker Compose 脚本时，请将导出放在 Paperless 源代码目录的 `export` 文件夹中。将 `source` 指定为 `../export`。

!!! note

    从旧版本的 Paperless 导入可能有效，但为了获得最佳效果，建议版本匹配。

!!! warning

    导入器应针对完全空（数据库和目录）的 Paperless-ngx 安装运行。如果使用仅数据导入，则只有数据库必须为空。

### 文档重标记器 {#retagger}

假设您导入了数百个文档，现在想要引入一个标签或设置一个新的通信者，并将其匹配应用于所有当前已导入的文档。这个问题很常见，因此有相应的工具。

```
document_retagger [-h] [-c] [-T] [-t] [-i] [--id-range] [--use-first] [-f]

可选参数：
-c, --correspondent
-T, --tags
-t, --document_type
-s, --storage_path
-i, --inbox-only
--id-range
--use-first
-f, --overwrite
```

在更改或添加匹配规则后运行此命令。它将遍历数据库中的所有文档，并尝试根据新规则匹配文档。

指定 `-c`、`-T`、`-t` 和 `-s` 的任何组合，让重标记器执行指定元数据类型的匹配。如果您不指定任何这些选项，文档重标记器将不会执行任何操作。

指定 `-i` 让文档重标记器仅处理带有收件箱标签的文档。当您不想弄乱已处理的文档时，这很有用。

指定 `--id-range 1 100` 让文档重标记器仅处理特定的文档 ID 范围。如果您有很多文档并且只想在文档子集上测试匹配规则，这可能很有用。

当多个文档类型或通信者匹配单个文档时，重标记器不会将这些分配给文档。指定 `--use-first` 来覆盖此行为，仅使用它找到的第一个通信者或类型。此选项不适用于标签，因为任何数量的标签都可以应用于文档。

最后，`-f` 指定您希望覆盖已分配的通信者、类型和/或标签。默认行为是不将通信者和类型分配给已分配了此数据的文档。`-f` 对标签的工作方式不同：默认情况下，只会向文档添加额外的标签，不会删除任何标签。使用 `-f` 时，不再匹配文档的标签也会被删除。

### 管理自动匹配算法

_自动_ 匹配算法需要一个训练好的神经网络才能工作。每当您的数据发生变化时，都需要更新此网络。Docker 镜像通过任务调度程序自动处理此问题。您可以通过调用以下管理命令手动更新分类器：

```
document_create_classifier
```

此命令不带参数。

### 文档缩略图 {#thumbnails}

使用此命令重新创建文档缩略图。可选地包含 `--document {id}` 选项以仅为特定文档生成缩略图。

您还可以指定 `--processes` 来控制用于生成新缩略图的进程数。默认是使用可用处理器的四分之一。

```
document_thumbnails
```

### 管理文档搜索索引 {#index}

文档搜索索引负责为网站提供搜索结果。每当文档被添加到 Paperless、更改或从 Paperless 中删除时，文档索引都会自动更新。但是，如果搜索产生不存在的文档或找不到任何内容，您可能需要手动重新创建索引。

```
document_index {reindex,optimize}
```

指定 `reindex` 以从头开始创建索引。这可能需要一些时间。

指定 `optimize` 以优化索引。这会更新索引的某些方面，通常使查询更快，并确保自动补全正常工作。此命令由任务调度程序定期调用。

### 清除数据库读取缓存

如果启用了数据库读取缓存，**在应用程序上下文之外对数据库进行任何更改后，您必须运行此命令**。这包括诸如恢复数据库备份或执行 SQL 语句（如 UPDATE、INSERT、DELETE、ALTER、CREATE 或 DROP）等操作。

在此类修改后未能使缓存失效，可能导致从缓存提供过时数据，并**可能导致数据损坏**或应用程序行为不一致。

使用以下管理命令清除缓存：

```
python3 manage.py invalidate_cachalot
```

!!! info
数据库读取缓存基于 Django-Cachalot。您可以参考其[文档](https://django-cachalot.readthedocs.io/en/latest/quickstart.html#manage-py-command)。

### 管理文件名 {#renamer}

如果您使用 Paperless 的功能为文档[分配自定义文件名](advanced_usage.md#file-name-handling)，您可以使用此命令在更改命名方案后移动所有文件。

!!! warning

    由于此命令会移动您的文档，建议事先进行备份。重命名逻辑是健壮的，永远不会覆盖或删除文件，但再怎么小心也不为过。

```
document_renamer
```

该命令不带参数，一次性处理所有文档。

了解如何使用[管理工具](#management-commands)。

### 完整性检查器 {#sanity-checker}

Paperless 有一个内置的完整性检查器，用于检查您的文档集合是否存在问题。

完整性检查器检测到的问题如下：

-   缺少原始文件。
-   缺少归档文件。
-   由于权限不当导致无法访问原始文件。
-   由于权限不当导致无法访问归档文件。
-   通过将其校验和与数据库中存储的内容进行比较，检查原始文档是否损坏。
-   通过将其校验和与数据库中存储的内容进行比较，检查归档文档是否损坏。
-   缺少缩略图。
-   由于权限不当导致无法访问缩略图。
-   没有任何内容的文档（警告）。
-   媒体目录中的孤立文件（警告）。这些是 Paperless 中任何文档都未引用的文件。

```
document_sanity_checker
```

该命令不带参数。根据您的文档存档大小，这可能需要一些时间。

### 获取电子邮件