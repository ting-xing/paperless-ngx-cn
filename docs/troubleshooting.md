# 故障排除

## 消费者未添加任何文件

检查以下问题：

-   确保您放置文档的目录是 Paperless 正在监视的文件夹。使用 Docker 时，此设置在 `docker-compose.yml` 文件中配置。不使用 Docker 时，请查看 `CONSUMPTION_DIR` 设置。如果使用 Docker，请不要调整此设置。

-   确保 Redis 已启动并正在运行。Paperless 异步处理其任务，文档需要 Redis 运行才能到达任务处理器。

-   确保任务处理器正在运行。Docker 会自动执行此操作。手动执行以下命令来调用任务处理器：

    ```shell-session
    celery --app paperless worker
    ```

-   查看 Paperless 的输出并检查是否有任何错误。

-   转到管理界面，检查是否有失败的任务。如果有，任务将包含错误信息。

## 消费者警告 `XX 的 OCR 失败`

如果您发现 OCR 准确率太低，和/或文档消费者警告
`XX 的 OCR 失败，但由于启用了 FORGIVING_OCR，我们将坚持使用已获得的结果`，
那么您可能需要安装与文档语言匹配的 [Tesseract 语言文件](https://packages.ubuntu.com/search?keywords=tesseract-ocr)。

例如，如果您在任何 Ubuntu 或 Debian 机器上运行 Paperless-ngx，并且您的文档是用西班牙语编写的，您可能需要运行：

    apt-get install -y tesseract-ocr-spa

## 消费者无法获取任何新文件

如果您注意到消费者仅在启动时获取消费目录中的文件，但不会找到之后添加的任何其他文件，您需要通过配置选项 [`PAPERLESS_CONSUMER_POLLING`](configuration.md#PAPERLESS_CONSUMER_POLLING) 启用文件系统轮询。

这将禁用使用 inotify 监听文件系统更改，Paperless 将改为手动检查消费目录的更改。

## Paperless 总是重定向到 /admin

您可能曾经安装过旧版本的 Paperless。Paperless 在您的浏览器中安装了到 /admin 的永久重定向，您需要清除浏览数据/缓存来解决此问题。

## 操作不允许

您可能会看到如下错误：

```shell-session
chown: changing ownership of '../export': Operation not permitted
```

容器尝试设置列出目录的文件所有权。这是必需的，以便在 Docker 内运行 Paperless 的用户对这些文件夹具有写入权限。例如，当将这些目录指向 NFS 共享时会发生这种情况。

确保可以在这些目录上执行 `chown`。

## 分类器错误：无可用训练数据

这表明自动匹配算法未找到可供学习的文档。这可能有两个原因：

-   您不使用自动匹配算法：在这种情况下可以安全地忽略此错误。
-   您正在使用自动匹配算法：分类器明确排除带有“收件箱”标签的文档。请验证您的存档中是否有不带收件箱标签的文档。该算法只会从不在收件箱中的文档中学习。

## 每个文档都出现 sklearn 中的 UserWarning

您可能会遇到如下警告：

```
/usr/local/lib/python3.7/site-packages/sklearn/base.py:315:
UserWarning: Trying to unpickle estimator CountVectorizer from version 0.23.2 when using version 0.24.0.
This might lead to breaking code or invalid results. Use at your own risk.
```

当负责自动匹配算法的 Paperless 某些依赖项更新时会发生这种情况。更新这些依赖项后，您当前的训练数据*可能*不再兼容。在大多数情况下可以忽略此警告。当 Paperless 更新训练数据时，此警告会自动消失。

如果您想消除警告或确实遇到自动匹配问题，请删除数据目录中的 `classification_model.pickle` 文件，并让 Paperless 重新创建它。

## 添加 Office 文档时出现 504 服务器错误：网关超时

使用可选的 TIKA 集成时，您可能会遇到这些错误：

```
requests.exceptions.HTTPError: 504 Server Error: Gateway Timeout for url: http://gotenberg:3000/forms/libreoffice/convert
```

Gotenberg 是一个将 Office 文档转换为 PDF 文档的服务器，默认超时时间为 30 秒。当转换时间较长时，Gotenberg 会引发此错误。

您可以通过为 Gotenberg 配置命令标志来增加超时时间（另请参见[此处](https://gotenberg.dev/docs/modules/api#properties)）。如果使用 Docker Compose，可以通过在 `docker-compose.yml` 文件中进行以下配置更改来实现：

```yaml
# gotenberg chromium 路由用于转换 .eml 文件。我们不希望允许外部内容，如跟踪像素甚至 javascript。
command:
    - 'gotenberg'
    - '--chromium-disable-javascript=true'
    - '--chromium-allow-list=file:///tmp/.*'
    - '--api-timeout=60s'
```

## 消费目录中出现权限被拒绝错误

您可能会遇到如下错误：

```shell-session
消费 document.pdf 时发生以下错误：[Errno 13] Permission denied: '/usr/src/paperless/src/../consume/document.pdf'
```

当 Paperless 没有权限删除消费目录中的文件时会发生这种情况。如果 `USERMAP_UID` 和 `USERMAP_GID` 与 `1000` 不同，请确保将它们设置为主机操作系统上使用的用户 ID 和组 ID。请参阅 [Docker 设置](setup.md#docker)。

同时确保您能够在主机上读取和写入消费目录。

## 消费文件时出现 OSError: \[Errno 19\] No such device

如果您遇到如下错误：

```shell-session
File "/usr/local/lib/python3.7/site-packages/whoosh/codec/base.py", line 570, in open_compound_file
return CompoundStorage(dbfile, use_mmap=storage.supports_mmap)
File "/usr/local/lib/python3.7/site-packages/whoosh/filedb/compound.py", line 75, in __init__
self._source = mmap.mmap(fileno, 0, access=mmap.ACCESS_READ)
OSError: [Errno 19] No such device

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
File "/usr/local/lib/python3.7/site-packages/django_q/cluster.py", line 436, in worker
res = f(*task["args"], **task["kwargs"])
File "/usr/src/paperless/src/documents/tasks.py", line 73, in consume_file
override_tag_ids=override_tag_ids)
File "/usr/src/paperless/src/documents/consumer.py", line 271, in try_consume_file
raise ConsumerError(e)
```

Paperless 使用搜索索引来提供更好更快的全文搜索。此搜索索引存储在 `data` 文件夹内。搜索索引使用内存映射文件 (mmap)。上述错误表明 Paperless 无法创建和打开这些文件。

当您尝试将数据目录存储在特定不支持内存映射文件的文件系统（主要是网络共享）上时会发生这种情况。

## Web 界面卡在“正在加载...”

这可能有多重原因。

1.  如果您自己构建了 Docker 镜像或使用裸机部署，请确保 `<paperless-root>/static/frontend/<lang-code>/` 目录中有文件。如果没有文件，请确保您已成功执行 `collectstatic`，无论是手动执行还是作为 Docker 镜像构建的一部分。

    如果前端仍然缺失，请确保前端已编译（`src/documents/static/frontend` 中存在文件）。如果未编译，您需要自己编译前端，或者下载发布压缩包而不是克隆仓库。

## 读取元数据时出错

您可能会在日志文件中找到如下消息：

```
[WARNING] [paperless.parsing.tesseract] Error while reading metadata
```

这表明 Paperless 未能从您的某个文档中读取 PDF 元数据。当您在 Paperless 中打开受影响的文档进行编辑时会发生这种情况。Paperless 将继续工作，只是不会显示无效的元数据。

## 消费者因 FileNotFoundError 而失败

您可能会在日志文件中找到如下消息：

```
[ERROR] [paperless.consumer] Error while consuming document SCN_0001.pdf: FileNotFoundError: [Errno 2] No such file or directory: '/tmp/ocrmypdf.io.yhk3zbv0/origin.pdf'
Traceback (most recent call last):
  File "/app/paperless/src/paperless_tesseract/parsers.py", line 261, in parse
    ocrmypdf.ocr(**args)
  File "/usr/local/lib/python3.8/dist-packages/ocrmypdf/api.py", line 337, in ocr
    return run_pipeline(options=options, plugin_manager=plugin_manager, api=True)
  File "/usr/local/lib/python3.8/dist-packages/ocrmypdf/_sync.py", line 385, in run_pipeline
    exec_concurrent(context, executor)
  File "/usr/local/lib/python3.8/dist-packages/ocrmypdf/_sync.py", line 302, in exec_concurrent
    pdf = post_process(pdf, context, executor)
  File "/usr/local/lib/python3.8/dist-packages/ocrmypdf/_sync.py", line 235, in post_process
    pdf_out = metadata_fixup(pdf_out, context)
  File "/usr/local/lib/python3.8/dist-packages/ocrmypdf/_pipeline.py", line 798, in metadata_fixup
    with pikepdf.open(context.origin) as original, pikepdf.open(working_file) as pdf:
  File "/usr/local/lib/python3.8/dist-packages/pikepdf/_methods.py", line 923, in open
    pdf = Pdf._open(
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/ocrmypdf.io.yhk3zbv0/origin.pdf'
```

这可能表明 Paperless 尝试消费同一个文件两次。根据文档放入消费文件夹的方式，这可能由于多种原因发生。如果 Paperless 使用 inotify（默认）检查文档，请尝试调整 [inotify 配置](configuration.md#inotify)。如果启用了轮询，请尝试调整[轮询配置](configuration.md#polling)。

## 消费者等待文件保持未修改状态时失败。

您可能会在日志文件中找到如下消息：

```
[ERROR] [paperless.management.consumer] Timeout while waiting on file /usr/src/paperless/src/../consume/SCN_0001.pdf to remain unmodified.
```

这表明 Paperless 在等待文件完全写入消费文件夹时超时。调整[轮询配置](configuration.md#polling)值应能解决此问题。

!!! note

    用户需要手动将文件移出消费文件夹再移回，以便消费最初失败的文件。

## 消费者报告“操作系统报告文件仍忙”而失败。

您可能会在日志文件中找到如下消息：

```
[WARNING] [paperless.management.consumer] Not consuming file /usr/src/paperless/src/../consume/SCN_0001.pdf: OS reports file as busy still
```

这表明 Paperless 无法打开该文件，因为操作系统报告该文件仍在使用中。为防止崩溃，Paperless 未尝试消费该文件。如果 Paperless 使用 inotify（默认）检查文档，请尝试调整 [inotify 配置](configuration.md#inotify)。如果启用了轮询，请尝试调整[轮询配置](configuration.md#polling)。

!!! note

    用户需要手动将文件移出消费文件夹再移回，以便消费最初失败的文件。

## 日志报告“创建 PaperlessTask 失败”。

您可能会在日志文件中找到如下消息：

```
[ERROR] [paperless.management.consumer] Creating PaperlessTask failed: db locked
```

您可能正在使用基于 sqlite 的安装，增加了工作进程数量，并且遇到了 sqlite 的并发限制。同时上传或消费多个文件会导致许多工作进程尝试同时访问数据库。

如果您经常需要同时处理许多文档，请考虑更改为 PostgreSQL 数据库。否则，请尝试调整 [`PAPERLESS_DB_TIMEOUT`](configuration.md#PAPERLESS_DB_TIMEOUT) 设置，以允许数据库有更多时间解锁。此外，您可以将 SQLite 数据库更改为使用[“预写日志 (WAL)”](https://sqlite.org/wal.html)。这些更改可能会对性能产生轻微影响，但有助于防止数据库锁定问题。

## granian 启动失败，提示“不是有效的端口号”

您可能正在使用 Kubernetes 运行，它会自动创建一个名为 `${serviceName}_PORT` 的环境变量。Paperless 使用相同的环境变量来可选地更改 granian 监听的端口。

要解决此问题，请将 [`PAPERLESS_PORT`](configuration.md#PAPERLESS_PORT) 重新设置为您所需的端口，或默认的 8000。

## 数据库警告唯一约束“documents_tag_name_uniq”

您可能会看到如下数据库日志行：

```
ERROR:  duplicate key value violates unique constraint "documents_tag_name_uniq"
DETAIL:  Key (name)=(NameF) already exists.
STATEMENT:  INSERT INTO "documents_tag" ("owner_id", "name", "match", "matching_algorithm", "is_insensitive", "color", "is_inbox_tag") VALUES (NULL, 'NameF', '', 1, true, '#a6cee3', false) RETURNING "documents_tag"."id"
```

在使用轮询进行大量消费时可能会发生这种情况。Paperless 将正确处理，文件仍将被消费。

## 消费失败，提示“Ghostscript PDF/A 渲染失败”

新版本的 OCRmyPDF 在处理过程中遇到错误时会失败。
这是有意为之的，因为输出的归档文件可能与原始文件存在意外或不希望的差异。
如日志所示，如果遇到此错误，您可以设置 `PAPERLESS_OCR_USER_ARGS: '{"continue_on_soft_render_error": true}'` 来尝试“强制”处理存在此问题的文档。

## 删除文档时日志显示“可能存在不兼容的数据库列” {#convert-uuid-field}

删除文档时您可能会看到如下错误：

```
Data too long for column 'transaction_id' at row 1
```

此错误可能发生在从使用 Django 4 的 Paperless-ngx 版本（Paperless-ngx v2.13.0 之前版本）升级并使用 MariaDB/MySQL 数据库的安装中。由于 Django 5 中的向后不兼容更改，需要重新创建列 "documents_document.transaction_id"，这可以通过一次性运行以下管理命令来完成：

```shell-session
$ python3 manage.py convert_mariadb_uuid
```

## 平台特定部署故障排除

有一个用户维护的 Wiki 页面可用于帮助解决在特定平台（例如 SELinux）上部署 Paperless-ngx 时可能出现的问题。请参阅 [Wiki](https://github.com/paperless-ngx/paperless-ngx/wiki/Platform%E2%80%90Specific-Troubleshooting)。