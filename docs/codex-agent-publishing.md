# Codex Agent 小红书发布手册

## 目标

把小红书发布固定为一条可复用命令：

```bash
xhs post "标题" --image image.png --content "正文" --json
```

Agent 可以围绕这条命令做内容生产、草稿管理、飞书通知和历史记录，但真正发布动作只走
`xhs post`。

## 安装长期版本

当前长期改动在 fork 分支：

```bash
pip install "git+https://github.com/Zhan-boyi/xhs-cli.git@codex/creator-post-hardening"
```

如果使用 `pipx`：

```bash
pipx install "git+https://github.com/Zhan-boyi/xhs-cli.git@codex/creator-post-hardening"
```

如果使用 `uv tool`：

```bash
uv tool install "git+https://github.com/Zhan-boyi/xhs-cli.git@codex/creator-post-hardening"
```

## 发布前条件

必须满足：

1. 本地浏览器已经登录小红书。
2. 本地浏览器已经登录小红书创作者中心：

   ```text
   https://creator.xiaohongshu.com
   ```

3. 本地 cookie 已同步到：

   ```text
   ~/.xhs-cli/cookies.json
   ```

常用检查：

```bash
xhs status
```

如果 creator 登录态缺失，需要先在浏览器打开创作者中心完成登录，再运行：

```bash
xhs login
```

## 固定发布流程

```text
准备标题、正文、图片
  -> 调用 xhs post
  -> CLI 打开创作者中心发布页
  -> 切到上传图文
  -> 选择图片上传 input
  -> 上传图片
  -> 填标题和正文
  -> 必要时选择 AI 内容声明
  -> 点击发布
  -> 返回 JSON 结果
```

命令示例：

```bash
xhs post "AI 不再只是替代人，而是重排工作流" \
  --image ./cover.png \
  --content "正文内容" \
  --ai-generated \
  --json
```

返回示例：

```json
{
  "success": true,
  "note_id": ""
}
```

说明：`note_id` 是 best-effort。创作者后台不一定暴露可解析的笔记 id，所以
`success=true` 且 `note_id=""` 仍然可以表示发布动作成功。

## AI 内容声明

三种模式：

```bash
--ai-generated
```

明确声明笔记含 AI 合成内容。

```bash
--no-ai-generated
```

明确不声明 AI 合成内容。

不传参数：

```text
CLI 根据标题和正文里的常见 AI 关键词自动判断。
```

如果页面没有“笔记含AI合成内容”控件，CLI 不会因为这个声明失败而中断发布。

## Agent 行为边界

Agent 可以自动做：

- 生成标题、正文、图片。
- 组装 `xhs post` 命令。
- 调用 CLI 发布。
- 解析 `--json`。
- 写发布历史。
- 发飞书通知。

Agent 不应该做：

- 绕过验证码。
- 绕过安全验证。
- 伪造登录态。
- 把 cookie value 打印到聊天或日志里。

如果遇到验证码或安全验证，正确行为是停下来，让人完成验证后重试。

## 失败处理

### Creator login required

含义：普通小红书登录态可能有效，但创作者中心登录态不可用。

处理：

1. 浏览器打开 `https://creator.xiaohongshu.com`。
2. 完成登录。
3. 运行 `xhs login` 同步 cookie。
4. 重新运行 `xhs post`。

### 找不到上传 input

含义：创作者后台页面可能改版。

处理：

1. 查看 OpenSpec 文档：

   ```text
   openspec/changes/creator-post-hardening/design.md
   openspec/specs/creator-posting/spec.md
   ```

2. 更新 `xhs_cli/client.py` 中的页面适配 helper。
3. 补测试。

### 发布返回 success=false

处理：

1. 用 `-v` 开启调试日志：

   ```bash
   xhs -v post "标题" --image image.png --content "正文" --json
   ```

2. 检查页面是否要求验证码、二次确认或 creator 登录。

## 验证命令

单元测试：

```bash
pytest tests/test_client.py tests/test_cli.py -q
```

本次 fork 验证结果：

```text
47 passed
```

OpenSpec 风格文档：

```text
openspec/changes/creator-post-hardening/proposal.md
openspec/changes/creator-post-hardening/design.md
openspec/changes/creator-post-hardening/tasks.md
openspec/specs/creator-posting/spec.md
```

