---
id: gbyvc9xdid3zp6ktymkjmlg
title: Fix Open-in-Slack on KDE
desc: ""
updated: 1689210617651
created: 1689210162660
---

Slack use a magic URL like `slack://T08PSQ7BQ/magic-login/<magic-id>` to open a workspace on Slack Desktop.

On KDE, here is a bug: <https://bugs.kde.org/show_bug.cgi?id=429408#c3>

It looks like this:

```bash
kde-open slack://T08PSQ7BQ/magic-login/<magic-id>
```

but kde-open turn it to:

```bash
slack -s slack://t08psq7bq/magic-login/<magic-id>
                 ^ --- lowercase workspace id
```

A bypass way to resolve it is create `/usr/local/bin/xdg-open` with following content:

```shell
#!/usr/bin/env bash

if [[ "${1:-}" = slack://* ]]; then
    exec /usr/lib/slack/slack --enable-crashpad "$1"
fi

exec /usr/bin/xdg-open "$@"
```

Reference: <https://stackoverflow.com/a/71579300/9018019>
