# NotePic OSS CLI

> Upload images in Markdown, MDX, and HTML to Aliyun OSS, then rewrite links in place.

[![License](https://img.shields.io/github/license/Luhui-Dev/NotePic-OSS-CLI)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org/downloads/)
[![Release](https://img.shields.io/github/v/release/Luhui-Dev/NotePic-OSS-CLI)](https://github.com/Luhui-Dev/NotePic-OSS-CLI/releases)

**Made by [@LuhuiDev](https://luhuidev.com) · Part of [LuhuiDev Toolkit](https://luhuidev.com)**

NotePic OSS CLI 是 NotePic OSS 的命令行实现。它会扫描 Markdown / MDX / HTML 中的图片引用，上传到阿里云 OSS，并把文档中的本地链接改写为公网 URL。适合批量迁移文章、接入 CI、静态站点构建或服务器脚本。

如果你主要在 Obsidian 里写笔记，请看姊妹项目 [NotePic-OSS-Obsidian](https://github.com/Luhui-Dev/NotePic-OSS-Obsidian)。

## 快速开始

```bash
git clone git@github.com:Luhui-Dev/NotePic-OSS-CLI.git
cd NotePic-OSS-CLI

pip install -e .
cp .env.example .env
# 编辑 .env，填入 OSS 配置

notepic-oss article.md --env-file .env
```

默认不会覆盖原文件，而是生成 `article.oss.md`。需要原地改写时使用 `--in-place` 或 `-i`。

```bash
notepic-oss posts/2024-trip.md --env-file .env
```

示例输出：

```text
Reading posts/2024-trip.md
Processing images...
  ✓ ./images/cover.png
      → https://cdn.example.com/markdown/8b3f9e6a4c1d2e7f9a0b.jpg
      482.1 KB → 138.4 KB  (-71%)
  ⏭  already on OSS: https://cdn.example.com/markdown/...
Writing posts/2024-trip.oss.md
Done. found=2 uploaded=1 skipped=1 failed=0
```

## 特性

- 支持 `.md`、`.mdx`、`.markdown`、`.html`、`.htm`。
- 识别 Markdown 图片、HTML / MDX `<img src="...">`、引用式图片定义、Obsidian `![[image.png]]`。
- 跳过 Markdown 围栏代码块、行内代码，以及 HTML 的 `script/style/pre/code` 和注释。
- JPEG / WebP 按质量参数重编码，PNG 无损优化；GIF / SVG / 动画图保持原样。
- 使用 SHA-256 内容哈希生成对象名，重复内容不会重复上传。
- 默认跳过已经在自家 OSS 或自定义域名上的图片。
- 可选处理远程图片，把外链图片搬运到自己的 OSS。

## 安装

需要 Python 3.9+。

```bash
pip install -e .
```

也可以在仓库目录中直接运行：

```bash
pip install -r requirements.txt
python -m notepic_oss article.md --env-file .env
```

安装后会注册两个等价命令：

```bash
notepic-oss article.md
md-oss article.md
```

## 配置

复制 `.env.example` 并填写自己的阿里云 OSS 配置：

```bash
cp .env.example .env
```

| 变量                    | 必填 | 说明                                                       |
| ----------------------- | ---- | ---------------------------------------------------------- |
| `OSS_ACCESS_KEY_ID`     | 是   | 阿里云 AccessKey ID                                        |
| `OSS_ACCESS_KEY_SECRET` | 是   | 阿里云 AccessKey Secret                                    |
| `OSS_ENDPOINT`          | 是   | 区域 Endpoint，例如 `https://oss-cn-hangzhou.aliyuncs.com` |
| `OSS_BUCKET`            | 是   | Bucket 名称                                                |
| `OSS_PREFIX`            | 否   | Bucket 内对象前缀，例如 `markdown`                         |
| `OSS_CUSTOM_DOMAIN`     | 否   | CDN / 自定义域名，用于生成最终 URL                         |

CLI 默认从系统环境变量读取配置。需要临时加载 `.env` 时加 `--env-file`：

```bash
notepic-oss article.md --env-file .env
```

## 命令用法

```bash
# 默认输出到 article.oss.md
notepic-oss article.md

# 原地覆盖
notepic-oss article.md --in-place

# 指定输出路径
notepic-oss article.md --output dist/article.md

# 跳过压缩
notepic-oss article.md --no-compress

# 调整 JPEG / WebP 质量，范围 1-100，默认 85
notepic-oss article.md --quality 90

# 把远程图片也重新上传到自己的 OSS
notepic-oss article.md --process-remote

# 预览改写结果，不写文件
notepic-oss article.md --dry-run > preview.md

# 静默输出
notepic-oss article.md --quiet
```

Obsidian wikilink 默认开启：

```bash
# 自动向上查找 .obsidian 来定位 Vault
notepic-oss note.md --env-file .env

# 附件不在笔记附近时，显式指定 Vault 根目录
notepic-oss note.md --obsidian-vault ~/Obsidian/MyVault --env-file .env

# 非 Obsidian 文档可关闭 wikilink 解析
notepic-oss article.md --no-obsidian --env-file .env
```

## 会改写什么

会改写：

- `![alt](./image.png)`
- `![alt](<image with spaces.png>)`
- `![alt](./image.png "title")`
- `<img src="./image.png" />`
- `[cover]: ./image.png`
- `![[image.png]]`
- `![[image.png|alt|400x200]]`
- 远程图片，但仅在开启 `--process-remote` 时处理

不会改写：

- `data:` URI、`#` 锚点、`mailto:` 链接
- 已经位于当前 OSS Bucket 或自定义域名下的图片
- 代码块、行内代码、HTML 跳过区域中的内容
- MDX 中 `src={variable}` 这类 JSX 表达式
- 非图片类型的 Obsidian 嵌入，例如 `![[some-note]]`

## 退出码

| 退出码 | 含义                                           |
| ------ | ---------------------------------------------- |
| `0`    | 成功，且没有图片失败                           |
| `1`    | 输入文件缺失、配置缺失，或至少一张图片处理失败 |
| `2`    | 参数错误或文件类型不支持                       |

## 安全建议

- 使用专属 RAM 子账号，不要使用阿里云主账号 AccessKey。
- 权限收窄到目标 Bucket 的对象读写能力。
- 不要提交 `.env` 或任何包含 AccessKey 的文件。
- CI / 服务器中优先使用平台 secrets，而不是把 `.env` 落盘。

## License

MIT
