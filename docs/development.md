# 开发

本节描述了开始开发 Paperless-ngx 所需采取的步骤。

从 GitHub 检出源代码。仓库的组织方式如下：

-   `main` 分支始终代表最新发布版本，只有在新版本发布时才会看到更改。
-   `dev` 分支包含将在下一个版本中出现的代码。
-   `feature-X` 分支包含将在某个版本中出现（但不一定是下一个版本）的较大更改。

当对 Paperless-ngx 进行功能性更改时，**务必**在 `dev` 分支上进行更改。

除此之外，文件夹结构如下：

-   `docs/` - 文档。
-   `src-ui/` - 前端代码。
-   `src/` - 后端代码。
-   `scripts/` - 用于辅助开发各个部分的各类脚本。
-   `docker/` - 构建 Docker 镜像所需的文件。

## 为 Paperless-ngx 做贡献

也许您使用 Paperless-ngx 已有一段时间，想要添加一两个功能，或者您遇到了一个错误，并且对如何解决有一些想法。开源软件的优点在于，您可以看到问题所在，并帮助修复它，让每个人受益！

在贡献之前，请查看我们的[行为准则](https://github.com/paperless-ngx/paperless-ngx/blob/main/CODE_OF_CONDUCT.md)以及[贡献指南](https://github.com/paperless-ngx/paperless-ngx/blob/main/CONTRIBUTING.md)中的其他重要信息。

## 使用 pre-commit 钩子进行代码格式化

为了确保项目源代码风格和格式的一致性，项目利用 Git 的 [`pre-commit`](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) 钩子在允许提交之前执行一些格式化和代码检查。这样，每个人都使用相同的风格，并且可以及早发现一些常见问题。

安装后，钩子会在您提交时运行。如果格式不正确或代码检查器发现了问题，提交将被拒绝。您需要查看输出并修复问题。一些钩子，例如 Python 代码检查和格式化工具 `ruff`，会格式化失败的文件，因此您只需再次 `git add` 这些文件并重试提交即可。

## 通用设置

从 GitHub 分叉并克隆代码后，您需要进行首次设置。

!!! note

      除非另有说明，否则所有命令都直接从项目的根文件夹执行。

1.  按照[裸机部署](setup.md#bare_metal)中的说明安装先决条件 + [uv](https://github.com/astral-sh/uv)。

2.  将 `paperless.conf.example` 复制为 `paperless.conf`，并通过 `PAPERLESS_DEBUG=true` 在文件中启用调试模式。

3.  创建 `consume` 和 `media` 目录：

    ```bash
    mkdir -p consume media
    ```

4.  安装 Python 依赖项：

    ```bash
    $ uv sync --group dev
    ```

5.  安装 pre-commit 钩子：

    ```bash
    $ uv run pre-commit install
    ```

6.  为您的开发实例应用迁移并创建超级用户（也可以通过 Web UI 完成）：

    ```bash
    # src/

    $ uv run manage.py migrate
    $ uv run manage.py createsuperuser
    ```

7.  您现在可以...

    -   安装 Redis，或者
    -   使用包含的 `scripts/start_services.sh` 脚本通过 Docker 启动一个 Redis 实例（以及其他一些服务，如 Tika、Gotenberg 和数据库服务器），或者
    -   启动一个裸 Redis 容器

        ```
        docker run -d -p 6379:6379 --restart unless-stopped redis:latest
        ```

8.  继续后端或前端开发——或者两者都进行 :-)。

## 后端开发

后端是一个 [Django](https://www.djangoproject.com/) 应用程序。[PyCharm](https://www.jetbrains.com/de-de/pycharm/) 和 [Visual Studio Code](https://code.visualstudio.com) 都适合开发，但您可以使用任何您想要的工具。

将 IDE 配置为使用 `src/` 文件夹作为基本源代码文件夹。在您的 IDE 中配置以下启动配置：

-   `python3 manage.py runserver`
-   `python3 manage.py document_consumer`
-   `celery --app paperless worker -l DEBUG`（或任何其他日志级别）

要启动所有这些服务：

```bash
# src/

$ python3 manage.py runserver & \
  python3 manage.py document_consumer & \
  celery --app paperless worker -l DEBUG
```

您可能需要前端来测试后端代码。
这假设您的系统上已安装 AngularJS。
有关更多详细信息，请转到[前端开发](#front-end-development)部分。
要一次性构建前端，请使用以下命令：

```bash
# src-ui/

$ pnpm install
$ ng build --configuration production
```

### 测试

-   在 `src/` 目录中运行 `pytest` 以执行所有测试。这也会生成一个 HTML 覆盖率报告。运行测试时，也会加载 `paperless.conf`。但是，测试依赖于默认配置。这并不理想。但目前，在测试时请确保除了 DEBUG 之外没有覆盖任何设置。

!!! note

      行长度规则 E501 通常有助于在屏幕上并排显示多个源文件。但是，在某些情况下，确实无法使某些行符合要求，尤其是复杂的 IF 语句。附加 `# noqa: E501` 以禁用对特定行的此检查。

### 包管理

Paperless 使用 `uv` 来管理开发和生产的包和虚拟环境。
要使用 `uv` 完成一些常见任务，请遵循以下快捷方式：

要升级所有锁定的包到允许的最新版本：`uv lock --upgrade`

要升级单个锁定的包：`uv lock --upgrade-package <package>`

要添加新包：`uv add <package>`

要添加新的开发包：`uv add --dev <package>`

## 前端开发

前端使用 AngularJS 构建。要开始使用，您需要 Node.js（版本 24+）和 `pnpm`。

!!! note

    以下命令均在 `src-ui` 目录中执行。您需要一个正在运行的后端（包括活动会话）来连接到后端 API。要启动它，请参考上面[后端开发](#back-end-development)部分中的命令。

1.  安装 Angular CLI。您可能需要 sudo 权限来执行此命令：

    ```bash
    pnpm install -g @angular/cli
    ```

2.  确保它在您的路径上。

3.  安装所有必要的模块：

    ```bash
    pnpm install
    ```

4.  您可以通过运行以下命令启动开发服务器：

    ```bash
    ng serve
    ```

    这会在您保存时自动更新。但是，原地编译可能会因语法错误而失败，在这种情况下您需要重新启动它。

    默认情况下，开发服务器在 `http://localhost:4200/` 上可用，并配置为访问 `http://localhost:8000/api/` 的 API，这是后端的默认地址。如果您在后端启用了 `DEBUG`，则会应用一些针对允许的主机和 CORS 的安全覆盖，以便前端的行为与生产环境完全一致。

### 测试和代码风格

前端代码（.ts、.html、.scss）通过 Git 的 `pre-commit` 钩子使用 `prettier` 进行代码格式化，这些钩子在提交时自动运行。有关安装说明，请参见[上文](#code-formatting-with-pre-commit-hooks)。您也可以通过 CLI 运行此操作，例如使用以下命令：

```bash
$ git ls-files -- '*.ts' | xargs pre-commit run prettier --files
```

前端测试使用 Jest 和 Playwright。可以非交互式地运行单元测试和 e2e 测试：

```bash
$ ng test
$ npx playwright test
```

Playwright 还包含一个 UI，可以通过以下方式运行：

```bash
$ npx playwright test --ui
```

### 构建前端

为了构建前端并将其作为 Django 的一部分提供服务，请执行：

```bash
$ ng build --configuration production
```

这将构建前端并将其放在 Django 服务器将作为静态内容提供的位置。这样，您可以验证身份验证是否正常工作。

## 本地化

Paperless-ngx 支持多种不同的语言。由于 Paperless-ngx 由 Django 应用程序和 AngularJS 前端组成，这两个部分必须分别进行翻译。

### 前端本地化

-   AngularJS 前端根据 [Angular 文档](https://angular.io/guide/i18n)进行本地化。
-   项目的源语言是 "en_US"。
-   源字符串最终位于文件 `src-ui/messages.xlf` 中。
-   翻译后的字符串需要放在 `src-ui/src/locale/` 文件夹中。
-   为了从源文件中提取添加或更改的字符串，请调用 `ng extract-i18n`。

添加新语言需要在 `src-ui/src/locale/` 文件夹中添加翻译文件并调整几个文件。

1.  调整 `src-ui/angular.json`：

    ```json
    "i18n": {
        "sourceLocale": "en-US",
        "locales": {
            "de": "src/locale/messages.de.xlf",
            "nl-NL": "src/locale/messages.nl_NL.xlf",
            "fr": "src/locale/messages.fr.xlf",
            "en-GB": "src/locale/messages.en_GB.xlf",
            "pt-BR": "src/locale/messages.pt_BR.xlf",
            "language-code": "language-file"
        }
    }
    ```

2.  将语言添加到 `src-ui/src/app/services/settings.service.ts` 中的 `LANGUAGE_OPTIONS` 数组：

    ```
    `dateInputFormat` 是一个特殊的字符串，用于定义日期输入字段的行为，并且必须包含 "dd"、"mm" 和 "yyyy"。
    ```

3.  在 `src-ui/src/app/app.module.ts` 中导入并注册此语言环境的 Angular 数据：

    ```typescript
    import localeDe from '@angular/common/locales/de'
    registerLocaleData(localeDe)
    ```

### 后端本地化

后端中出现的大部分字符串仅在管理员界面中使用。然而，其中一些仍然显示在前端（例如错误消息）。

-   Django 应用程序根据 [Django 文档](https://docs.djangoproject.com/en/3.1/topics/i18n/translation/)进行本地化。
-   项目的源语言是 "en_US"。
-   本地化文件最终位于 `src/locale/` 文件夹中。
-   为了从应用程序中提取字符串，请调用 `python3 manage.py makemessages -l en_US`。在对可翻译字符串进行更改后，这一点很重要。
-   消息文件需要编译才能在应用程序中显示。调用 `python3 manage.py compilemessages` 来完成此操作。生成的文件不会提交到 git 中，因为它们是衍生工件。构建流水线负责执行此命令。

添加新语言需要在 `src/locale/` 文件夹中添加翻译文件，并调整文件 `src/paperless/settings.py` 以包含新语言：

```python
LANGUAGES = [
    ("en-us", _("English (US)")),
    ("en-gb", _("English (GB)")),
    ("de", _("German")),
    ("nl-nl", _("Dutch")),
    ("fr", _("French")),
    ("pt-br", _("Portuguese (Brazil)")),
    # 在此处添加语言。
]
```

## 构建文档

文档使用 material-mkdocs 构建，请参阅其[文档](https://squidfunk.github.io/mkdocs-material/reference/)。
如果您想在本地构建文档，可以这样做：

1.  构建文档

    ```bash
    $ uv run mkdocs build --config-file mkdocs.yml
    ```

    _或者..._

2.  提供文档服务。这将在 http://127.0.0.1:8000 启动一份文档副本，每次您更改内容时都会自动刷新。

    ```bash
    $ uv run mkdocs serve
    ```

## 构建 Docker 镜像

Docker 镜像主要由 GitHub Actions 工作流构建，但在开发时本地构建和标记镜像可能更快。

确保已安装 `docker-buildx` 包。构建镜像的方式与任何镜像相同：

```
docker build --file Dockerfile --tag paperless:local .
```

## 扩展 Paperless-ngx

Paperless-ngx 没有任何花哨的插件系统，并且可能永远不会有。但是，应用程序的某些部分经过设计，允许在不修改基础代码的情况下轻松集成附加功能。

### 创建自定义解析器

Paperless-ngx 使用解析器来添加文档。解析器负责：

-   从原始文件中检索内容
-   创建缩略图
-   _可选：_ 从原始文件中检索创建日期
-   _可选：_ 从原始文件创建归档文档

可以向 Paperless-ngx 添加自定义解析器以支持更多文件类型。为此，您需要编写解析器本身，并向 Paperless-ngx 宣告其存在。

解析器本身必须继承 `documents.parsers.DocumentParser`，并且必须实现 `parse` 和 `get_thumbnail` 方法。如果您不想依赖 Paperless-ngx 的默认日期猜测机制，可以提供自己的 `get_date` 实现。

```python
class MyCustomParser(DocumentParser):

    def parse(self, document_path, mime_type):
        # 此方法不返回任何内容。相反，您应该将从文档中获取的任何内容分配给以下字段：

        # 文档的内容。
        self.text = "content"

        # 可选：您从原始文件创建的 PDF 文档的路径。
        self.archive_path = os.path.join(self.tempdir, "archived.pdf")

        # 可选：文档的“创建”日期。
        self.date = get_created_from_metadata(document_path)

    def get_thumbnail(self, document_path, mime_type):
        # 这应该返回您为此文档创建的缩略图的路径。
        return os.path.join(self.tempdir, "thumb.webp")
```

如果在解析过程中遇到任何问题，请引发 `documents.parsers.ParseError`。

`self.tempdir` 目录是一个临时目录，保证为空并在消费完成后删除。您可以使用该目录存储任何中间文件，也可以使用它来存储缩略图/归档文档。

之后，您需要向 Paperless-ngx 宣告您的解析器。您需要将一个处理程序连接到 `document_consumer_declaration` 信号。请查看文件 `src/paperless_tesseract/apps.py` 以了解如何完成此操作。该处理程序是一个返回有关解析器信息的方法：

```python
def myparser_consumer_declaration(sender, **kwargs):
    return {
        "parser": MyCustomParser,
        "weight": 0,
        "mime_types": {
            "application/pdf": ".pdf",
            "image/jpeg": ".jpg",
        }
    }
```

-   `parser` 是对继承 `DocumentParser` 的类的引用。
-   `weight` 用于当两个或更多解析器能够解析一个文件时：权重较高的解析器胜出。这可用于覆盖 Paperless-ngx 提供的解析器。
-   `mime_types` 是一个字典。键是您的解析器支持的 MIME 类型，值是 Paperless-ngx 在存储文件和提供下载时应使用的默认文件扩展名。我们可以从文件扩展名猜测，但某些 MIME 类型关联了许多扩展名，并且负责猜测扩展名的 Python 方法并不总是返回相同的值。

## 使用 Visual Studio Code devcontainer

另一种简单的开始开发的方法是使用 Visual Studio Code devcontainer。这种方法将创建一个预配置的开发环境，其中包含所有必需的工具和依赖项。
[了解更多关于 devcontainer 的信息](https://code.visualstudio.com/docs/devcontainers/containers)。
.devcontainer/vscode/tasks.json 和 .devcontainer/vscode/launch.json 文件包含有关特定任务和启动配置的更多信息（请参阅非标准的 "description" 字段）。

开始使用：

1.  在您的机器上克隆仓库，并在 VS Code 中打开 Paperless-ngx 文件夹。

2.  VS Code 将提示您“在容器中重新打开”。执行此操作并等待环境启动。

3.  如果您的宿主机操作系统是 Windows：

    -   Visual Studio Code 中的源代码管理视图可能会显示：“检测到的 Git 仓库可能不安全，因为该文件夹的所有者不是当前用户。” 使用“管理不安全仓库”来修复此问题。
    -   Git 可能检测到所有文件都已修改，因为 Windows 使用 CRLF 行尾。在容器的终端中运行 `git checkout .` 以修复此问题。

4.  通过运行任务 **Project Setup: Run all Init Tasks** 来初始化项目。这将初始化数据库表并创建一个超级用户。然后，您可以为生产环境编译前端或以调试模式运行前端。

5.  项目已准备好进行调试，可以启动全栈调试或单独的调试进程。要在不调试的情况下启动项目，请运行任务 **Project Start: Run all Services**。