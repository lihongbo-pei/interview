# Git

如果你想回到 `git add` 之前的状态，可以尝试以下方法：

方法一：使用 `git reset`

如果你已经执行了 `git commit`，但还没有推送到远程仓库，可以使用 `git reset` 命令来撤销提交。

1. 撤销最后一次提交

```bash
git reset HEAD~ --soft
```
- `HEAD~` 表示撤销最后一次提交。
- `--soft` 选项会保留更改的内容，撤销提交操作，但不会撤销 `git add` 的操作。

如果你希望撤销 `git add` 的操作，可以使用：
```bash
git reset HEAD~ --hard
```
- `--hard` 选项会撤销最后一次提交，并且撤销 `git add` 的操作，但不会删除工作目录中的更改。

2. 撤销 `git add`

如果你只想撤销 `git add` 的操作，可以使用：
```bash
git reset
```
这会撤销 `git add` 的操作，但不会撤销更改。

方法二：使用 `git checkout` 或 `git restore`

如果你已经执行了 `git commit`，但还没有推送到远程仓库，可以使用 `git checkout` 或 `git restore` 来恢复到之前的状态。

1. 撤销最后一次提交

```bash
git checkout HEAD~ -- .
```
或者
```bash
git restore --staged --worktree .
```
这会撤销最后一次提交，并且撤销 `git add` 的操作，但不会删除工作目录中的更改。

方法三：使用 `git revert`

如果你**已经将更改推送到远程仓库**，可以使用 `git revert` 来创建一个新的提交，撤销之前的提交。

```bash
git revert HEAD
```
这会创建一个新的提交，撤销最后一次提交的内容。

总结

根据你的需求，选择合适的方法：
- 如果你只是想撤销最后一次提交，但保留更改，使用 `git reset HEAD~ --soft`。
- 如果你希望撤销最后一次提交，并且撤销 `git add` 的操作，但保留工作目录中的更改，使用 `git reset HEAD~` 或 `git checkout HEAD~ -- .`。
- 如果你希望撤销最后一次提交，并且撤销 `git add` 的操作，同时删除工作目录中的更改，使用 `git reset HEAD~ --hard`。
- 如果你已经将更改推送到远程仓库，使用 `git revert HEAD`。
