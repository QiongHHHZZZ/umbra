# Umbra 国服维护说明（CN-MAINTAINER v1）

## 文档目的

- 作为 `QiongHHHZZZ/umbra` 的国服维护规则文档。
- 面向维护者，不面向普通用户；用户说明放在 `README.md`。

## 维护范围与分发仓库

- 源码仓库：`QiongHHHZZZ/umbra`
- 上游仓库：`una-xiv/umbra`
- 插件清单仓库：`QiongHHHZZZ/DalamudPlugins`
  - 清单地址：`https://raw.githubusercontent.com/QiongHHHZZZ/DalamudPlugins/main/pluginmaster.json`

## 上游同步规则（固定流程）

- 先同步上游，再做 CN 适配。
- 仅使用 `merge` 同步上游，不使用 `cherry-pick` 复制上游历史。
- 不使用网页端“丢弃 N 个提交”式同步。

标准流程：

1. `git fetch upstream --prune`
2. `git checkout main`
3. `git merge upstream/main`
4. 解决冲突并编译验证
5. 再提交本地化/CN 修复

## 版本规则（Umbra）

- 版本真源：`Umbra/Umbra.csproj` 的 `<Version>`。
- 统一沿用 ICE 规则：4 段纯数字 `A.B.C.D`，其中第 4 段使用 `UUFF`。
  - `UU`：上游版本“最后一段”数字（Umbra 目前通常是第三段，如 `3.1.11` 的 `11`）
  - `FF`：本地修订号（`00-99`）
- 纯上游同步版：`FF=00`；本地修订：`FF` 递增。
- 已发布版本必须严格递增，禁止回退。

示例：

- 上游 `3.1.11` -> 纯同步 `3.1.11.1100`
- 第一次国服修订 -> `3.1.11.1101`
- 第二次国服修订 -> `3.1.11.1102`
- 上游升到 `3.1.12` -> `3.1.12.1200`

## 当前国服差异（同步冲突时优先保留）

- `Directory.Build.props`
  - 自动探测 `XIVLauncherCN` 常见路径并设置 `DALAMUD_HOME`，避免本地缺省路径仅指向国际服目录。
- `Umbra/src/Umbra.Fonts.cs`
  - `Dalamud Default` 使用 `NotoSansCJKsc-Medium.otf`，避免中文缺字方框。
- `Umbra.Game/src/CustomDeliveries/CustomDeliveriesRepository.cs`
  - `SatisfactionBonusGuarantee` 改为 `TryGetRow` + 回退逻辑，避免 `rowId` 越界崩溃。

## 发版流程

1. **确定版本号**
   - 按本文件版本规则生成新版本号。
2. **更新版本真源**
   - 修改 `Umbra/Umbra.csproj` 中 `<Version>`。
3. **本地构建**
   - `dotnet build Umbra.sln -c Release`
4. **确认产物**
   - 产物目录：`out/Release/Umbra/`
   - 至少确认：`latest.zip`、`Umbra.json`
5. **发布源码仓库 Release**
   - 创建 tag：`v<版本号>`
   - 上传发布资产（建议统一命名 `Umbra.zip`，或直接使用 `latest.zip`）
6. **更新 DalamudPlugins 清单**
   - 编辑 `QiongHHHZZZ/DalamudPlugins` 的 `pluginmaster.json` 中 Umbra 条目：
     - `AssemblyVersion`
     - `DownloadLinkInstall`
     - `DownloadLinkUpdate`
     - `RepoUrl`
7. **分发验证**
   - 检查 `pluginmaster.json` 可访问且 JSON 合法。
   - Dalamud 客户端手动刷新仓库，确认可安装/更新。

## 发布检查清单

- [ ] `Umbra/Umbra.csproj` 版本号已更新且单调递增
- [ ] `dotnet build Umbra.sln -c Release` 通过
- [ ] `out/Release/Umbra/latest.zip` 已生成
- [ ] GitHub Release 已上传对应 zip 资产
- [ ] `DalamudPlugins/pluginmaster.json` 已同步版本与下载链接
- [ ] Dalamud 客户端安装/更新验证通过

## Umbra 清单条目模板（DalamudPlugins）

在 `QiongHHHZZZ/DalamudPlugins/pluginmaster.json` 中新增或更新 Umbra 条目时，可直接参考：

```json
{
  "Author": "Una, QiongHHHZZZ(国服适配)",
  "Name": "Umbra XIV",
  "InternalName": "Umbra",
  "AssemblyVersion": "<与发布包一致，例如 3.1.12.0>",
  "Description": "Umbra UI",
  "ApplicableVersion": "any",
  "RepoUrl": "https://github.com/QiongHHHZZZ/umbra",
  "Tags": [
    "ui",
    "marker",
    "markers",
    "toolbar"
  ],
  "DalamudApiLevel": 14,
  "LoadPriority": 0,
  "IconUrl": "https://raw.githubusercontent.com/QiongHHHZZZ/DalamudPlugins/main/icons/umbra.png",
  "Punchline": "Adds fully customizable toolbars and 3D world markers to the game.",
  "AcceptsFeedback": true,
  "DownloadLinkInstall": "https://github.com/QiongHHHZZZ/umbra/releases/download/v<版本号>/Umbra.zip",
  "DownloadLinkUpdate": "https://github.com/QiongHHHZZZ/umbra/releases/download/v<版本号>/Umbra.zip"
}
```

注意：

- `AssemblyVersion` 以发布包中的 `Umbra.json` 为准（需与包内版本一致）。
- `v<版本号>` 与 Release tag 保持一致，例如 `v3.1.12.0`。
- 若发布资产名不是 `Umbra.zip`（例如 `latest.zip`），下载链接文件名也要同步修改。

## 常见问题

- **编译报 Dalamud installation not found**
  - 先确认 `XIVLauncherCN` 的 Dalamud 资源存在；本仓库已在 `Directory.Build.props` 做常见路径探测。
- **编译报缺少 `drawing/Una.Drawing/Una.Drawing.csproj`**
  - 先执行：`git submodule update --init --recursive`。
- **老主顾（CustomDeliveries）初始化报 `rowId` 越界**
  - 检查 `CustomDeliveriesRepository` 的 `TryGetRow` 防护是否被上游合并冲掉。
- **中文显示方框**
  - 检查 `Umbra/src/Umbra.Fonts.cs` 是否仍为 `NotoSansCJKsc-Medium.otf`。
