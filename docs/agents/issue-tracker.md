# Issue tracker: GitHub

Issues live as GitHub issues on `annalinneajohansson82/audhd-toolkit`. All
issue work happens there, using whatever GitHub access your harness provides —
the `gh` CLI, a GitHub MCP server, or the API directly. The outcome matters,
not the tool.

Pull requests are not a triage surface — ideas and requests become issues,
never PRs.

## Reference commands (`gh` CLI)

For agents with shell access to `gh` (Hermes's default setup):

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments`
- **Comment**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..." / --remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

Agents without `gh` use the equivalent GitHub MCP or API operations.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Read the GitHub issue, including its comments.
