# 版本控制(GIT)

> 对于课程网站的仓库
> - 将版本历史可视化并进行探索
> - 是谁最后修改了`README.md`文件？（提示：使用 `git log` 命令并添加合适的参数）
> - 最后一次修改`_config.yml` 文件中 `collections:` 行时的提交信息是什么？（提示：使用 `git blame` 和 `git show`）

- 版本历史可视化
    ```bash
    git log --all --oneline --graph
    ```

- 查看最后修改`README.md`文件的提交者
    ```bash
    git log -1 --pretty=format:"%an" README.md
    ```
- 查看最后修改`_config.yml` 文件中 `collections:` 行时的提交信息
    ```bash
    git blame _config.yml | grep "collections:" | tail -n1 | awk '{print $1}' | xargs git show --pretty=format:"%s" | head -n1
    ```

> 使用 `Git` 时的一个常见错误是提交本不应该由 `Git` 管理的大文件，或是将含有敏感信息的文件提交给 `Git` 。尝试向仓库中添加一个文件并添加提交信息，然后将其从历史中删除

```bash
git filter-branch --tree-filter 'rm -f password.txt' HEAD
```

- `filter-branch`可以改写历史提交数据
- `--tree-filter`检出每个提交，运行指定命令，然后重新提交

> 从 GitHub 上克隆某个仓库，修改一些文件。当您使用 `git stash` 会发生什么？当您执行 `git log --all --oneline` 时会显示什么？通过 `git stash pop` 命令来撤销 `git stash` 操作，什么时候会用到这一技巧？

`git stash`会将暂存区的修改保存到栈中，然后将工作区和暂存区恢复到最后一次提交的状态。
`git log --all --oneline`会多出两项提交记录。
`git stash pop`可以用于将修改记录应用到当前分支。

> 与其他的命令行工具一样，`Git` 也提供了一个名为 `~/.gitconfig` 配置文件 (或 `dotfile`)。请在 `~/.gitconfig` 中创建一个别名，使您在运行 `git graph` 时，您可以得到 `git log --all --graph --decorate --oneline` 的输出结果；

```bash
[alias]
    graph = log --all --graph --decorate --oneline
```

> 可以通过执行 `git config --global core.excludesfile ~/.gitignore_global` 在 `~/.gitignore_global` 中创建全局忽略规则。配置您的全局 `gitignore` 文件来自动忽略系统或编辑器的临时文件，例如 `.DS_Store`；


