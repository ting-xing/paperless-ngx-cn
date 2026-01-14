# 常见问题解答

## _Paperless-ngx 的总体计划是什么？_

**答：** 虽然 Paperless-ngx 在很大程度上已经被认为是“功能完备”的，但它是一个社区驱动的项目，开发将以此为指导。新功能可以通过 [GitHub 讨论区](https://github.com/paperless-ngx/paperless-ngx/discussions) 提交，并由社区“点赞投票”，但这并不能保证该功能一定会被实现。本项目始终欢迎以 PR、想法等形式进行合作。

## _我使用 Docker。我的文档在哪里？_

**答：** 默认情况下，您的文档存储在 Docker 卷 `paperless_media` 中。Docker 会自动为您管理这个卷。它是一个持久化存储，只要您不显式删除它，数据就会一直保留。实际位置取决于您的主机操作系统。在 Linux 上，这个位置很可能是：

```
/var/lib/docker/volumes/paperless_media/_data
```

!!! warning

    请勿随意操作此文件夹。不要更改权限，也不要手动移动文件。此文件夹完全由 Docker 和 Paperless 管理。

!!! note

    从消费目录摄入的文件会在此媒体目录内重新创建，并从消费目录本身移除。

## 假设我一年后想换用其他工具。我能轻松迁移到其他系统吗？

**答：** 您的文档以纯文件形式存储在媒体文件夹中。您随时可以将这些文件拖出该文件夹以在其他地方使用。以下是关于此操作的几点说明：

-   Paperless-ngx 从不修改您的原始文档。它会保留所有文档的校验和，并使用计划任务检查器来确保它们保持不变。
-   默认情况下，Paperless 使用每个文档的内部 ID 作为其文件名。这对于导出可能不太方便。但是，您可以通过 [配置文件名格式](advanced_usage.md#file-name-handling) 来调整 Paperless 中文件的存储方式。
-   [导出器](administration.md#exporter) 是另一种以合理文件名将文件从 Paperless 中导出的简便方法。

## _Paperless-ngx 支持哪些文件类型？_

**答：** 目前支持以下文件：

-   PDF 文档、PNG 图像、JPEG 图像、TIFF 图像、GIF 图像和 WebP 图像会经过 OCR 处理并转换为 PDF 文档。
-   纯文本文档也受支持，并会原样添加到 Paperless 中。
-   启用可选的 Tika 集成后（参见 [Tika 配置](https://docs.paperless-ngx.com/configuration#tika)），Paperless 还支持各种 Office 文档（.docx、.doc、.odt、.ppt、.pptx、.odp、.xls、.xlsx、.ods）。

Paperless-ngx 通过检查文件内容来确定其类型。文件扩展名无关紧要。

## _Paperless-ngx 能在树莓派上运行吗？_

**答：** 简短的回答是：可以。我已在树莓派 3 B 上测试过。详细的回答是：Paperless 的某些部分运行会非常慢，例如 OCR。在树莓派上，尽量在将文档输入 Paperless 之前先进行 OCR，以便 Paperless 可以复用文本。Web 界面会流畅得多，因为它运行在您的浏览器中，并且 Paperless 提供数据所需的工作量要少得多。

!!! note

    您可以调整一些设置，使 Paperless 使用更少的处理能力。详情请参阅 [安装指南](setup.md#less-powerful-devices)。

## _如何在树莓派上安装 Paperless-ngx？_

**答：** 有针对 arm64 硬件的 Docker 镜像可用，因此只需按照 [Docker Compose 说明](https://docs.paperless-ngx.com/setup/#installation) 操作即可。与裸机安装相比，除了需要更多磁盘空间外，即使在树莓派上，Docker 带来的开销也几乎为零。

如果您决定采用裸机安装路线，请注意一些 Python 依赖项没有针对 ARM/ARM64 的预编译包。安装这些包需要额外的开发库，并且编译会花费很长时间。

!!! note

    对于 ARMv7（32 位）系统，Paperless 可能仍然可以运行，但可能需要修改 Dockerfile（如果使用 Docker）或安装额外的工具来进行裸机安装。建议升级到 arm64 系统。

## _如何在 Unraid 上运行？_

**答：** Paperless-ngx 在 Unraid 中作为 [社区应用](https://unraid.net/community/apps?q=paperless-ngx) 提供。[Uli Fahrer](https://github.com/Tooa) 为此创建了一个容器模板。

## _如何在我的烤面包机上运行？_

**答：** 老实说，我不知道！对于所有其他可能能够运行 Paperless 的设备，您需要自己摸索。如果您无法运行 Docker 镜像，文档中提供了裸机安装的说明。

## _关于 Redis 许可变更和使用其开源分支的情况如何？_

目前（2024 年 10 月），像 Valkey 或 Redirect 这样的 Redis 分支尚未得到我们上游库的官方支持，因此使用它们来替代 Redis 不受官方支持。

然而，它们声称与 Redis 协议兼容，并且很可能可以工作，但我们目前还不会正式更新以使用这些分支作为代理。