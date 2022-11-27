# git-safe-rebase

安全的執行 `git rebase` 指令。

## Why?

因為不熟悉 `git rebase` 指令的新手或是老手操作失誤，可能會在執行完 `git rebase` 指令一段時間後才驚恐地意識到之前的 rebase 有問題 (比如少了幾個 commits 或是某一些 commits 發生了自動合併錯誤問題[^1])。此時候雖然也可以使用 `git reflog` 指令來查詢與恢復，但這可能會花上大量時間，所以才建立了這個 `git safe-reabse` 指令，**讓操作失誤或事後發現問題時可以輕鬆恢復到 rebase 之前。**

## Installation

```bash
# git safe-rebase
git config --global alias.safe-rebase '!git branch --force before-rebase && git branch --force before-rebase-histories/$(git branch --show-current)/$(date '+%F/%H-%M-%S')/$(git rev-parse --short HEAD) && git rebase --autostash'

# git undo-safe-rebase
git config --global alias.undo-safe-rebase '!git rebase --autostash --onto before-rebase HEAD'

# git ls-before-rebase
git config --global alias.ls-before-rebase '!git branch | grep "^[ ]*before-rebase.*"'

# git delete-before-rebase
git config --global alias.delete-before-rebase '!git branch | grep "^[ ]*before-rebase.*" | xargs git branch -D'
```

## Usage

### `git safe-rebase`

在執行 `git rebase` 指令之前先建立 `before-rebase` 與 `before-rebase-histories/{current-branch-name}/{YYYY-MM-DD}/{hh-mm-ss}/{hash}` 分支，以便執行 `git rebase` 指令後，若發現問題可以快速地恢復到 rebase 之前。

#### Example

> 可以用 `git safe-rebase` 指令來取代 `git rebase` 指令。

```bash
# git rebase origin/main
git safe-rebase origin/main

# git rebase -i origin/main
git safe-rebase -i origin/main
```

### `git undo-safe-rebase`

恢復到執行 `git safe-rebase` 指令前。

#### Example

> **Note**  
> 需搭配 `git safe-rebase` 指令使用，而不是 `git rebase` 指令。

```bash
git safe-rebase origin/main

# Oops! I screw up and do it all over again.
git undo-safe-rebase
```

### `git ls-before-rebase`

列出以 `before-rebase*` 為前綴的所有分支。

#### Example

```bash
git safe-rebase origin/main
git ls-before-rebase
```

如果需要恢復分支某個時間點 rebase 之前的狀態，可以像這樣處理:

```bash
git ls-before-rebase | grep {branch-name}
git safe-rebase --onto before-rebase-histories/{branch-name}/{YYYY-MM-DD}/{hh-mm-ss}/{hash} HEAD
```

### `git delete-before-rebase`

刪除以 `before-rebase*` 為前綴的所有分支。

#### Example

> **Warning**  
> 這個指令會刪除 `git ls-before-rebase` 指令列出的所有分支，建議刪除前先使用 `git ls-before-rebase` 指令確認。

```bash
git ls-before-rebase
git delete-before-rebase
```

## Uninstallation

移除 `git safe-rebase` 及相關指令:

```bash
git config --global --unset alias.safe-rebase
git config --global --unset alias.undo-safe-rebase
git config --global --unset alias.ls-before-rebase
git config --global --unset alias.delete-before-rebase
```

[^1]: 自動合併錯誤問題更難以察覺，例如改動到同一個檔案的不同位置：其中一位 committer 重新命名或移除了函式的參數或函式內的變數，而另一位 committer 在不同位置仍使用原本的參數或變數名稱去處理，這個情況在合併時不會發生衝突，或是被判斷為可以自動合併。

    ```diff
    - function foo (a) {
    + function foo (b) { // committed by A, rename the parameter from a to b
       // ...
    +  if (result < 0) { // committed by B
    +    return a; // Uncaught ReferenceError: a is not defined
    +  }
       // ...
       return result;
     }
    ```
