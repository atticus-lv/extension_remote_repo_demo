请帮我创建一个“托管在 GitHub Pages 上的 Blender 扩展静态远程仓库”项目。

目标：
1. 这是一个 Blender Extensions 的 static repository。
2. 当前仓库不保存任何扩展 zip 文件。
3. 当前仓库只维护一个源配置文件，配置若干扩展 zip 的远程下载链接。
4. 每个 zip 都来自其他 GitHub 仓库的 Release asset 直链。
5. 在构建阶段，自动下载这些 zip，读取其中的 `blender_manifest.toml`，校验内容并生成 Blender 需要的 `index.json`。
6. 生成后的站点部署到 GitHub Pages，并可在 Blender 中通过：
   `https://<user>.github.io/<repo>/index.json`
   作为 Remote Repository 使用。

请按“最少依赖、易维护、纯静态部署”的原则实现。

实现要求：
1. 使用 Python 实现。
2. 不要引入数据库、后端服务、对象存储或额外服务器。
3. 不要把 zip 文件提交到仓库。
4. 不要手写完整的 `extensions.json` 元数据。
5. 元数据应尽可能直接从每个 zip 内的 `blender_manifest.toml` 提取。
6. 允许仓库里的源配置文件只保存：
   - `archive_url`
   - 少量可选 override，例如 `website`、`tags`、`enabled`
7. 通过 GitHub Actions 自动生成 `index.json` 和可选 `index.html`，再部署到 GitHub Pages。

请生成一个最小但完整可运行的仓库，包含以下内容：

目录与文件：
1. 一个清晰的项目目录结构。
2. `sources.json`：
   - 这是唯一需要手动维护的源配置文件。
   - 至少支持以下格式：

   ```json
   [
     {
       "archive_url": "https://github.com/path—to-file.zip"
     }
   ]
	```

 - 也支持少量可选字段，例如：
   - `enabled`
   - `website`
   - `tags`
   - `notes`

3. 一个 Python 脚本，例如 `scripts/generate_index.py`，负责：
   - 读取 `sources.json`
   - 跳过 `enabled: false` 的条目
   - 校验 `archive_url`
   - 下载 zip
   - 在 zip 内查找 `blender_manifest.toml`
   - 解析 TOML
   - 校验 manifest 和 zip 是否符合要求
   - 计算 `archive_size`
   - 计算 `archive_hash`，格式必须为 `sha256:<hex>`
   - 生成符合 Blender Extension Listing API v1 的 `index.json`
   - 可选生成一个简单的 `index.html`

4. 一个 GitHub Actions 工作流，要求：
   - push 到 `main` 时运行
   - 支持手动触发
   - 安装 Python 依赖
   - 运行生成脚本
   - 将生成结果部署到 GitHub Pages

5. 一个 `.nojekyll` 文件，避免 GitHub Pages 对静态产物做不必要处理。

6. 一个 `README.md`，说明：
   - 仓库用途
   - 如何新增扩展
   - 如何填写 GitHub Release zip 直链
   - 如何本地生成 `index.json`
   - 如何启用 GitHub Pages
   - 如何在 Blender 中添加远程仓库
   - 常见错误及排查方式

功能细节要求：
1. `archive_url` 必须是 GitHub Release asset 的 zip 直链，例如：
   `https://github.com/<owner>/<repo>/releases/download/<tag>/<file>.zip`

2. 脚本必须明确拒绝以下 URL：
   - `https://github.com/<owner>/<repo>/releases/tag/<tag>`
   - `https://github.com/<owner>/<repo>/releases/latest`
   - 非 `.zip` 结尾的下载链接

3. 下载与校验时，至少检查：
   - HTTP 请求成功
   - 响应内容是 zip
   - zip 内存在 `blender_manifest.toml`
   - zip 可正常读取
   - manifest 至少包含 Blender 扩展需要的关键字段
   - 多个扩展的 `id` 不能重复

4. 对 `blender_manifest.toml` 的处理规则：
   - 尽量从 manifest 直接提取字段
   - 支持从 manifest 提取以下字段：
     - `id`
     - `name`
     - `tagline`
     - `version`
     - `type`
     - `maintainer`
     - `license`
     - `website`
     - `tags`
     - `schema_version`
     - `blender_version_min`
     - `blender_version_max`
   - 若 `sources.json` 中提供同名 override，可按你设计的合理策略覆盖，但请在 README 里写清楚规则

5. 要兼容 zip 中这两种常见结构：
   - `blender_manifest.toml` 在 zip 根目录
   - `blender_manifest.toml` 在唯一顶层目录中

6. 生成的 `index.json` 必须符合 Blender Extension Listing API v1 结构，至少包含：
   - top-level:
     - `version`
     - `blocklist`
     - `data`
   - each item:
     - `id`
     - `name`
     - `tagline`
     - `version`
     - `type`
     - `archive_size`
     - `archive_hash`
     - `archive_url`
     - `blender_version_min`
     - `maintainer`
     - `license`
     - `schema_version`
   - 如果字段存在，也应输出：
     - `website`
     - `tags`
     - `blender_version_max`

7. `index.html` 只需要简洁实用：
   - 展示扩展列表
   - 显示名称、版本、tagline、maintainer
   - 提供下载链接
   - 明确指出仓库入口 `index.json`

8. 代码风格要求：
   - 默认 ASCII
   - 代码和注释简洁
   - 不要过度抽象
   - 错误信息要清楚，方便维护者定位哪一个 URL 或哪一个 zip 出错

本地开发要求：
1. 提供一个简单的本地运行方式，例如：
   - `python scripts/generate_index.py`
2. 如果需要依赖，请提供 `requirements.txt`
3. Python 版本请用 GitHub Actions 常见可用版本，例如 3.11 或 3.12

请直接输出：
1. 目录结构
2. 每个文件的完整内容
3. GitHub Actions 工作流完整内容
4. GitHub Pages 启用步骤
5. Blender 中添加该仓库的步骤
6. 一个最小可运行示例
7. README 完整内容

如果有实现取舍，请优先选择：
1. 少依赖
2. 易读
3. 易维护
4. 对错误更敏感而不是静默容错

另外，请严格参考以下官方文档，输出内容应尽量与官方格式保持一致：
- Blender Manual: Creating an Extensions Repository
  https://docs.blender.org/manual/en/latest/advanced/extensions/creating_repository/
- Blender Manual: Creating a Static Extensions Repository
  https://docs.blender.org/manual/en/latest/advanced/extensions/creating_repository/static_repository.html
- Blender Developer Docs: Extension Listing API
  https://developer.blender.org/docs/features/extensions/api_listing/
- Blender Developer Docs: Extension Listing API v1
  https://developer.blender.org/docs/features/extensions/api_listing/v1/
- Blender Developer Docs: Extension Manifest Schema 1.0.0
  https://developer.blender.org/docs/features/extensions/schema/1.0.0/
- Blender Manual: Extensions Command Line Arguments
  https://docs.blender.org/manual/en/latest/advanced/command_line/extension_arguments.html
- GitHub Docs: Configuring a publishing source for your GitHub Pages site
  https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- GitHub Docs: Using custom workflows with GitHub Pages
  https://docs.github.com/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages
- GitHub Docs: GitHub Pages limits
  https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits

如果官方文档与实现细节冲突，请优先遵循当前官方文档，并在 README 中说明。