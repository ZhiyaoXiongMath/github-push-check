# github-push-check

[中文](README.md) | [English](README.en.md)

GitHub 推送检查与版本发布技能，当前版本 **1.0.0**。

## 项目解决什么问题

在让 Codex 将本地项目推送到 GitHub 时，容易遗漏项目说明、把敏感资料或临时文件带入仓库，或者忘记准备与代码一致的版本发布。本技能为这项工作提供统一流程：先备份，在独立的 clean version 中整理，再检查、推送和发布。

它是供 Codex 读取的技能指令与界面配置，不是独立扫描程序，也不是拦截所有 `git push` 的 Git hook。

## 主要功能

- 备份原项目并核对完整性；在独立干净副本中修改，保留原工作目录。
- 自动生成或修复中英双语 README，检查项目用途、主要功能、安装方法、使用方法、输入输出示例及双语一致性。
- 检查并处理六类问题：密码/token、个人隐私、本地绝对路径、临时文件、测试垃圾文件及其他不宜公开的内容。
- 审查工作目录、暂存内容和拟推送历史，避免只删除当前文件却保留历史泄露。
- 验证干净版本，按已授权的目标推送。
- 核实项目当前版本；检查或补齐对应 tag、Release 标题、简洁发布说明及当前功能，推送 tag 并发布 GitHub Release。
- 重复运行时复用正确的现有 tag 和 Release；版本冲突时说明问题，不强制覆盖。

## 安装方法

需要支持本地技能的 Codex、Git，以及对此私有仓库的读取权限。执行推送和 Release 时，还需要已授权的 GitHub 连接器或已登录的 GitHub CLI；具体项目需要的构建工具由项目决定。技能本身没有额外的运行时依赖或 API key 配置。

在一个用于下载的目录中运行以下 PowerShell 命令。命令会拒绝覆盖已安装版本；更新前请先备份已有技能。

```powershell
git clone --branch v1.0.0 --depth 1 https://github.com/ZhiyaoXiongMath/github-push-check.git
if ($LASTEXITCODE -ne 0) { throw 'Clone failed; check repository access.' }

$skillBase = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $HOME '.codex/skills'
}
$skillDestination = Join-Path $skillBase 'github-push-check'
if (Test-Path -LiteralPath $skillDestination) {
    throw 'Skill already exists. Back it up before updating.'
}
New-Item -ItemType Directory -Path $skillDestination -Force | Out-Null
Copy-Item -LiteralPath './github-push-check/SKILL.md', './github-push-check/agents', './github-push-check/references', './github-push-check/VERSION' -Destination $skillDestination -Recurse
```

在其他系统上，将同样的四个文件/目录复制到个人技能目录的 `github-push-check` 文件夹即可：默认 `~/.codex/skills/github-push-check/`，设置了 `CODEX_HOME` 时使用该目录下的 `skills/github-push-check/`。无需复制下载目录的 Git 历史。安装后在下一轮对话中调用技能。

## 使用方法

在 Codex 中打开要发布的项目，输入 `$github-push-check` 并说明目标仓库。也可以直接要求将项目推送到 GitHub，让 Codex 根据技能描述选择本技能。

- 完整流程：说明要推送到哪个仓库，以及项目当前版本。
- 只检查：明确写“只审查，不修改、不推送、不发布”。
- 只推送代码：明确写“这次不发布 Release”。

用户当次指定的限制优先。版本不明确时不会编造版本号；本地备份不上传，远端访问权限不擅自改变。完整行为见 [SKILL.md](SKILL.md)，Release 细则见 [发布说明](references/release.md)。

## 输入输出示例

以下是说明性示例；实际路径、版本、问题和发布结果由项目检查决定。

**输入**

```text
使用 $github-push-check 检查当前项目，先备份，在 clean version 中补齐中英双语 README 并清理六类问题。推送到我指定的私有仓库；当前版本为 1.2.3，符合条件时发布同版本 Release。
```

**预期输出**

```text
检查通过。
- 备份：已建立并核对完整性。
- clean version：已生成，原项目保持不变。
- README：中英双语齐全，六项检查通过。
- 六类检查：已处理发现的问题，无未解决的发布阻塞项。
- 验证：列出实际执行的检查及结果。
- 推送：报告实际目标分支和提交。
- 版本发布：报告 v1.2.3 tag、Release 标题和实际链接。
```

若版本冲突、验证失败或仍有敏感内容，输出会说明具体未完成项，不能将部分完成写成发布成功。

## 仓库内容与范围

- `SKILL.md`：工作流程和检查标准。
- `agents/openai.yaml`：界面名称、默认提示和自动调用设置。
- `references/release.md`：版本核实、tag 和 Release 的创建/复用规则。
- `VERSION`：当前发布版本。

自动调用取决于 Codex 的技能选择；需要明确使用时输入 `$github-push-check`。扫描与语义审查不能保证发现所有敏感信息，无法验证的内容必须如实报告。该技能不拦截你在终端或其他工具中手动执行的推送，也不会自行发布软件包或部署服务。
