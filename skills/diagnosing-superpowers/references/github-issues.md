# GitHub 议题

如果 `gh` 已安装并通过身份验证，就用它处理认证、请求限额和 JSON。否则用 curl 调用公开 API。curl 也不可用时，把网址交给与你协作的人在浏览器中打开。

## 搜索

```bash
gh search issues --repo obra/superpowers --limit 10 "<terms>" \
  --json number,state,title --jq '.[] | "\(.number)\t\(.state)\t\(.title)"'
```

没有 `gh` 时，可执行以下命令（未经认证，每分钟限 10 次请求）。

```bash
curl -s -H "Accept: application/vnd.github+json" \
  "https://api.github.com/search/issues?q=repo:obra/superpowers+is:issue+<url-encoded terms>&per_page=10" \
  | jq -r '.items[] | "\(.number)\t\(.state)\t\(.title)"'
```

如果 curl 也不可用，交给对方这个网址 `https://github.com/obra/superpowers/issues?q=<terms>`。

## 提交

将填好的 `templates/issue.md` 写入工作区并展示拟提交的原文。获得批准后执行以下命令。

```bash
gh issue create --repo obra/superpowers --title "<title>" --body-file <path> \
  --label bug --label automated-issue-report
```

报告者若没有推送权限，GitHub 会静默丢弃标签，因此只有协作者提交时标签才会生效。模板末尾仍会注明该议题由技能提交。`gh` 无法上传附件。议题创建后，将诊断包路径交给对方，让对方通过浏览器上传。

没有 `gh` 时，提供套用 `bug_report.md` 模板的预填链接。该模板会为任何报告者提交的议题加上 `bug` 标签。

```
https://github.com/obra/superpowers/issues/new?template=bug_report.md&title=<url-encoded title>&body=<url-encoded body>
```

GitHub 会拒绝约 8,000 字符以上的网址。超出时，提供只预填标题的链接，让对方从文件中粘贴正文。
