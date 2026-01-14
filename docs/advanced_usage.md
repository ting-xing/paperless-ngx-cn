# 高级主题

Paperless 提供了一些功能，可以自动执行某些任务，让您的生活更轻松。

## 匹配标签、通信者、文档类型和存储路径 {#matching}

Paperless 将比较数据库中每个标签、通信者、文档类型和存储路径定义的匹配算法，以查看它们是否适用于文档中的文本。换句话说，如果您定义了一个名为 `Home Utility` 的标签，其 `match` 属性为 `bc hydro`，`matching_algorithm` 为 `Exact`，那么只要文档正文中某处出现文本 `bc hydro`，Paperless 就会自动用您的 `Home Utility` 标签标记新摄入的文档。

匹配逻辑非常强大。它支持使用不同的算法搜索文档文本，因此可能需要一些实验才能正确设置。

为了让标签、通信者、文档类型或存储路径自动分配给新摄入的文档，请使用 Web 界面分配匹配项和匹配算法。这些设置定义了何时将标签、通信者、文档类型和存储路径分配给文档。

可用的算法如下：

-   **无：** 不执行匹配。
-   **任意：** 在 PDF 中查找匹配项中提供的任何单词的任何出现。如果您将匹配项定义为 `Bank1 Bank2`，它将匹配包含其中任一术语的文档。
-   **全部：** 要求提供的每个单词都出现在 PDF 中，尽管不一定按照提供的顺序。
-   **精确：** 仅当匹配项在 PDF 中完全按提供的方式出现（即保持顺序）时才匹配。
-   **正则表达式：** 将匹配项解析为正则表达式，并尝试在文档中查找匹配项。
-   **模糊匹配：** 使用基于在文档内定位标签文本的部分匹配，使用 [partial ratio](https://rapidfuzz.github.io/RapidFuzz/Usage/fuzz.html#partial-ratio)
-   **自动：** 尝试自动匹配新文档。这不需要您设置匹配项。请参阅 [下面的说明](#automatic-matching)。

使用 _任意_ 或 _全部_ 匹配算法时，您可以通过将术语用双引号括起来来搜索由多个单词组成的术语。例如，使用 _任意_ 算法定义匹配文本 `"Bank of America" BofA`，将匹配包含 "Bank of America" 或 "BofA" 的文档，但不会匹配包含 "Bank of South America" 的文档。

然后只需保存您的标签、通信者、文档类型或存储路径，并通过消费者运行另一个文档。完成后，您应该会看到新创建的文档，并自动标记了适当的数据。

### 自动匹配 {#automatic-matching}

Paperless-ngx 附带了一个名为 _自动_ 的新匹配算法。此匹配算法尝试根据您已在现有文档上分配这些内容的方式，将标签、通信者、文档类型和存储路径分配给您的文档。它在底层使用了神经网络。

例如，如果您在 Bank of America 的 123 账户的所有银行对账单都标记了标签 "bofa123"，并且此标签的匹配算法设置为 _自动_，那么这个神经网络将检查您的文档，并自动学习何时分配此标签。

Paperless 试图隐藏这种方法涉及的许多复杂性。但是，在使用此功能时，您需要记住以下几点注意事项：

-   对文档的更改不会立即反映在匹配算法中。神经网络需要在更改后对您的文档进行 _训练_。Paperless 会定期（默认：每小时一次）检查更改，并自动为您执行此操作。
-   自动匹配算法仅考虑未放入收件箱（即，没有分配任何收件箱标签）的文档。这确保了神经网络仅从您之前正确标记的文档中学习。
-   匹配算法只有在标签、通信者、文档类型或存储路径与文档本身之间存在相关性时才能工作。您的银行对账单通常包含您的银行账号和银行名称，因此这通常效果很好。但是，诸如 "TODO" 之类的标签无法自动分配。
-   匹配算法需要合理数量的文档来确定何时分配标签、通信者、存储路径和类型。如果一千个文档中有一个的通信者是 "五年前我买过东西的非常不起眼的网店"，那么如果您再次从他们那里购买东西，它可能不会自动分配此通信者。文档越多越好。
-   Paperless 还需要合理数量的负面示例来决定何时不分配某个标签、通信者、文档类型或存储路径。当您开始用文档填充 Paperless 时，通常会出现这种情况。示例：如果您的所有文档都来自 "网店" 或 "银行"，并且两者都设置为自动匹配，那么 Paperless 会将其中一个通信者分配给 _任何_ 新文档。

## 挂钩到消费过程 {#consume-hooks}

有时，您可能希望在文档被消费时执行任意操作。Paperless 没有试图预测您可能想要做什么，而是允许您使用几个简单的挂钩，在文档被消费之前或之后执行您自己选择的脚本。

只需编写一个脚本，将其放在 Paperless 可以读取和执行的位置，然后将该脚本的路径放入 `paperless.conf` 或 `docker-compose.env` 中，变量名称为 [`PAPERLESS_PRE_CONSUME_SCRIPT`](configuration.md#PAPERLESS_PRE_CONSUME_SCRIPT) 或 [`PAPERLESS_POST_CONSUME_SCRIPT`](configuration.md#PAPERLESS_POST_CONSUME_SCRIPT)。

!!! info

    这些脚本在 **阻塞** 进程中执行，这意味着如果脚本运行时间很长，可能会显著减慢您的文档消费流程。如果您希望异步运行，必须在脚本中 fork 进程并退出。

### 消费前脚本 {#pre-consume-script}

在消费者在消费文件夹中看到新文档之后，但在执行任何文档处理之前执行。此脚本可以访问设置的以下相关环境变量：

| 环境变量            | 描述                                           |
| ------------------- | ---------------------------------------------- |
| `DOCUMENT_SOURCE_PATH`  | 已消费文档的原始路径                           |
| `DOCUMENT_WORKING_PATH` | 消费将处理的原始文档副本的路径                 |
| `TASK_ID`               | 用于处理新文档的任务的 UUID（如果有）         |

!!! note

    修改文档的消费前脚本应仅更改 `DOCUMENT_WORKING_PATH` 文件，否则可能会触发第二个消费任务，导致两个任务处理同一文档路径时失败。

!!! warning

    如果您的脚本以非确定性的方式修改 `DOCUMENT_WORKING_PATH`，这可能导致存储重复文档。

一个简单但常见的例子是创建如下简单脚本：

`/usr/local/bin/ocr-pdf`

```bash
#!/usr/bin/env bash
pdf2pdfocr.py -i ${DOCUMENT_WORKING_PATH}
```

`/etc/paperless.conf`

```bash
...
PAPERLESS_PRE_CONSUME_SCRIPT="/usr/local/bin/ocr-pdf"
...
```

这将把即将被消费的文档路径传递给 `/usr/local/bin/ocr-pdf`，后者将依次在您的文档上调用 [pdf2pdfocr.py](https://github.com/LeoFCardoso/pdf2pdfocr)，然后用文件的 OCR 版本覆盖该文件并退出。此时，消费过程将开始处理新修改的文件。

脚本的 stdout 和 stderr 将逐行记录到 Web 服务器日志中，同时记录的还有脚本的退出代码。

### 消费后脚本 {#post-consume-script}

在消费者成功处理文档并将其移入 Paperless 后执行。它接收以下环境变量：

| 环境变量                 | 描述                                       |
| ------------------------ | ------------------------------------------ |
| `DOCUMENT_ID`                | 文档的数据库主键                           |
| `DOCUMENT_FILE_NAME`         | 格式化的文件名，不包括路径                 |
| `DOCUMENT_TYPE`              | 文档类型（如果有）                         |
| `DOCUMENT_CREATED`           | 文档创建的日期和时间                       |
| `DOCUMENT_MODIFIED`          | 文档最后修改的日期和时间                   |
| `DOCUMENT_ADDED`             | 文档添加的日期和时间                       |
| `DOCUMENT_SOURCE_PATH`       | 原始文档文件的路径                         |
| `DOCUMENT_ARCHIVE_PATH`      | 生成的归档文件的路径（如果有）             |
| `DOCUMENT_THUMBNAIL_PATH`    | 生成的缩略图的路径                         |
| `DOCUMENT_DOWNLOAD_URL`      | 文档下载的 URL                             |
| `DOCUMENT_THUMBNAIL_URL`     | 文档缩略图的 URL                           |
| `DOCUMENT_OWNER`             | 文档所有者的用户名（如果有）               |
| `DOCUMENT_CORRESPONDENT`     | 分配的通信者（如果有）                     |
| `DOCUMENT_TAGS`              | 应用的标签的逗号分隔列表（如果有）         |
| `DOCUMENT_ORIGINAL_FILENAME` | 原始文档的文件名                           |
| `TASK_ID`                    | 用于导入文档的任务 UUID（如果有）          |

脚本可以使用任何语言。一个简单的 shell 脚本示例：

```bash title="post-consumption-example"
--8<-- "./scripts/post-consumption-example.sh"
```

!!! note

    消费后脚本不能取消消费过程。

!!! warning

    消费后脚本不应直接修改文档文件。

脚本的 stdout 和 stderr 将逐行记录到 Web 服务器日志中，同时记录的还有脚本的退出代码。

### Docker {#docker-consume-hooks}

要在使用 Docker 时挂钩到消费过程，您需要通过 `docker-compose.yml` 中的主机挂载将脚本传递到容器中。

假设您有一个脚本 `/home/paperless-ngx/scripts/post-consumption-example.sh`，您希望运行它。

您可以通过主机挂载将该脚本传递到消费者容器中：

```yaml
...
webserver:
  ...
  volumes:
    ...
    - /home/paperless-ngx/scripts:/path/in/container/scripts/ # (1)!
  environment: # (3)!
    ...
    PAPERLESS_POST_CONSUME_SCRIPT: /path/in/container/scripts/post-consumption-example.sh # (2)!
...
```

1. 外部脚本目录被挂载到容器内的一个位置。
2. 脚本的内部位置用于设置要运行的脚本。
3. 这也可以在 `docker-compose.env` 中设置。

故障排除：

-   监控 Docker Compose 日志
    `cd ~/paperless-ngx; docker compose logs -f`
-   检查脚本的权限，例如在出现权限错误时
    `sudo chmod 755 post-consumption-example.sh`
-   将脚本的输出管道传输到日志文件，例如
    `echo "${DOCUMENT_ID}" | tee --append /usr/src/paperless/scripts/post-consumption-example.log`

## 文件名处理 {#file-name-handling}

默认情况下，Paperless 将您的文档存储在媒体目录中，并使用分配给每个文档的标识符重命名它们。您最终会在媒体目录中得到像 `0000123.pdf` 这样的文件。这不一定是坏事，因为您通常不需要手动访问这些文件。但是，如果您希望以不同的方式命名文件，可以通过调整 [`PAPERLESS_FILENAME_FORMAT`](configuration.md#PAPERLESS_FILENAME_FORMAT) 配置选项或使用 [存储路径（见下文）](#storage-paths) 来实现。Paperless 会自动添加正确的文件扩展名，例如 `.pdf`、`.jpg`。

此变量允许您使用占位符配置文件名（允许使用文件夹）。例如，将其配置为：

```bash
PAPERLESS_FILENAME_FORMAT={{ created_year }}/{{ correspondent }}/{{ title }}
```

将创建如下目录结构：

```
2019/
  My bank/
    Statement January.pdf
    Statement February.pdf
2020/
  My bank/
    Statement January.pdf
    Letter.pdf
    Letter_01.pdf
  Shoe store/
    My new shoes.pdf
```

!!! warning

    请勿手动移动媒体文件夹中的文件。Paperless 会记住文档最后存储的文件名。如果您重命名文件，Paperless 将报告文件丢失并无法找到它们。

!!! tip

    Paperless 在保存文档时会检查其文件名。更改（或删除）[存储路径](#storage-paths) 将自动反映在文件系统中。但是，更改 `PAPERLESS_FILENAME_FORMAT` 时，您需要手动运行 [`文档重命名器`](administration.md#renamer) 来移动任何现有文档。

### 占位符 {#filename-format-variables}

Paperless 提供以下变量用于文件名中：

-   `{{ asn }}`: 文档的归档序列号，或 "none"。
-   `{{ correspondent }}`: 通信者的名称，或 "none"。
-   `{{ document_type }}`: 文档类型的名称，或 "none"。
-   `{{ tag_list }}`: 分配给文档的所有标签的逗号分隔列表。
-   `{{ title }}`: 文档的标题。
-   `{{ created }}`: 文档创建的全日期（ISO 8601 格式，例如 `2024-03-14`）。
-   `{{ created_year }}`: 仅创建年份，格式为带世纪的年份。
-   `{{ created_year_short }}`: 仅创建年份，格式为不带世纪的年份，零填充。
-   `{{ created_month }}`: 仅创建月份（数字 01-12）。
-   `{{ created_month_name }}`: 创建月份的名称，根据区域设置。
-   `{{ created_month_name_short }}`: 创建月份的缩写名称，根据区域设置。
-   `{{ created_day }}`: 仅创建日期（数字 01-31）。
-   `{{ added }}`: 文档添加到 Paperless 的全日期（ISO 格式）。
-   `{{ added_year }}`: 仅添加年份。
-   `{{ added_year_short }}`: 仅添加年份，格式为不带世纪的年份，零填充。
-   `{{ added_month }}`: 仅添加月份（数字 01-12）。
-   `{{ added_month_name }}`: 添加月份的名称，根据区域设置。
-   `{{ added_month_name_short }}`: 添加月份的缩写名称，根据区域设置。
-   `{{ added_day }}`: 仅添加日期（数字 01-31）。
-   `{{ owner_username }}`: 文档所有者的用户名（如果有），或 "none"。
-   `{{ original_name }}`: 文档原始文件名，减去扩展名（如果有），或 "none"。
-   `{{ doc_pk }}`: 文档的 Paperless 标识符（主键）。

!!! warning

    使用文件名占位符时，特别是使用 `{tag_list}` 时，可能会遇到操作系统最大路径长度的限制。在这种情况下，文件将保留先前的路径，并记录该问题。

!!! tip

    这些变量都是简单的字符串，但格式可以是完整的模板。有关更高级的格式设置，请参阅 [文件名模板](#filename-templates)。

Paperless 将尽可能保留数据库中的信息。但是，您可以在文档标题和通信者名称中使用的一些字符（例如 `: \ /` 等）不允许在文件名中使用，将被替换为短横线。

如果 Paperless 检测到两个文档共享相同的文件名，Paperless 会自动在文件名后附加 `_01`、`_02` 等。如果文件名中的所有占位符都计算为相同的值，就会发生这种情况。

如果 `PAPERLESS_FILENAME_FORMAT` 中包含的占位符有任何错误，Paperless 将回退到使用默认命名方案。

!!! caution

    截至目前，您可以通过设置以下内容，潜在地告诉 Paperless 将文件存储在媒体目录之外的任何位置：

    ```
    PAPERLESS_FILENAME_FORMAT=../../my/custom/location/{title}
    ```

    但是，请记住，在 Docker 内部，如果文件存储在预定义卷之外，重启后它们将丢失。

#### 空占位符

您可以通过更改 [`PAPERLESS_FILENAME_FORMAT_REMOVE_NONE`](configuration.md#PAPERLESS_FILENAME_FORMAT_REMOVE_NONE) 设置来影响空占位符的处理方式。

启用此设置后，所有空占位符将解析为 "" 而不是上面所述的 "none"。空占位符之前的空格也会被移除，空目录会被省略。

### 存储路径

当单个存储布局不足以满足您的用例时，存储路径允许设置更复杂的结构，以精确确定每个文档在文件系统中的存储位置。

-   每个存储路径都是一个 [`PAPERLESS_FILENAME_FORMAT`](configuration.md#PAPERLESS_FILENAME_FORMAT)，并遵循上述规则。
-   每个文档使用上述匹配算法分配一个存储路径，但可以随时覆盖。

例如，您可以定义以下两个存储路径：

1.  正常通信放入按 `年份/通信者` 排序的文件夹结构中。
2.  与保险公司的通信存储在扁平结构中，文件名较长，但包含通信的完整日期。

```
By Year = {{ created_year }}/{{ correspondent }}/{{ title }}
Insurances = Insurances/{{ correspondent }}/{{ created_year }}-{{ created_month }}-{{ created_day }} {{ title }}
```

然后，如果您将这些存储路径映射到文档，可能会得到以下结果。为简单起见，`By Year` 定义了与前面示例中相同的结构。

```text
2019/                                   # By Year
   My bank/
     Statement January.pdf
     Statement February.pdf

Insurances/                             # Insurances
   Healthcare 123/
     2022-01-01 Statement January.pdf
     2022-02-02 Letter.pdf
     2022-02-03 Letter.pdf
   Dental 456/
     2021-12-01 New Conditions.pdf
```

!!! tip

    定义存储路径是可选的。如果文档未定义存储路径，则应用全局的 [`PAPERLESS_FILENAME_FORMAT`](configuration.md#PAPERLESS_FILENAME_FORMAT)。

### 文件名模板 {#filename-templates}

文件名格式化使用 [Jinja 模板](https://jinja.palletsprojects.com/en/3.1.x/templates/) 来构建文件名。这允许在格式中包含复杂的逻辑，包括 [逻辑结构](https://jinja.palletsprojects.com/en/3.1.x/templates/#list-of-control-structures) 和 [过滤器](https://jinja.palletsprojects.com/en/3.1.x/templates/#id11) 来操作提供的 [变量](#filename-format-variables)。模板以字符串形式提供，可能是多行的，并渲染为单行。

此外，整个 Document 实例可用于更高级的方式，以及一些仅在更复杂逻辑下访问才有意义的变量。

#### 自定义 Jinja2 过滤器

##### 自定义字段访问

`get_cf_value` 过滤器从自定义字段数据中检索值，并可选择默认回退。

###### 语法

```jinja2
{{ custom_fields | get_cf_value('field_name') }}
{{ custom_fields | get_cf_value('field_name', 'default_value') }}
```

###### 参数

-   `custom_fields`: 这 _必须_ 是提供的自定义字段数据。
-   `name` (str): 要检索的自