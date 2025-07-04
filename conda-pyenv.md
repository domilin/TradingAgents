# macOS 下 conda 环境被 pyenv 劫持导致 Python 版本不对的排查与解决方案

## 1. 问题描述

在 macOS 下，使用 conda 创建并激活新环境（如 `tradingagents`，指定 Python 3.13），但激活后执行 `python --version` 发现依然是 pyenv 管理的 3.10.0，`which python` 结果为 pyenv 的 shims 路径，而不是 conda 环境的 python。

## 2. 原因分析

- 用户同时安装并初始化了 pyenv 和 conda。
- shell 配置文件（如 `.zshrc`）中，pyenv 的 shims 路径被加入 PATH，并且优先级高于 conda 环境的 bin 路径。
- 激活 conda 环境后，PATH 变量中 pyenv 的 shims 依然排在前面，导致 `python` 命令始终指向 pyenv 的 shims，而不是 conda 环境的 python。

## 3. 排查过程

1. 激活 conda 环境后，执行 `which python`，发现输出为：
   ```
   /Users/用户名/.pyenv/shims/python
   ```
2. 执行 `python --version`，发现依然是 pyenv 的版本（如 3.10.0），而不是 conda 环境指定的版本（如 3.13）。
3. 执行 `echo $PATH`，发现 PATH 顺序如下：
   ```
   ...:/Users/用户名/.pyenv/shims:/Users/用户名/anaconda3/envs/环境名/bin:...
   ```
   pyenv 的 shims 路径在 conda 环境 bin 之前。

## 4. 解决方案

### 方法一：强制将 conda 环境 bin 放到 PATH 最前面（推荐）

### ------------------【最后用的此方法】 ------------------

1. 打开 `~/.zshrc` 文件，在 conda 初始化和 pyenv 初始化之后，添加如下内容：
   ```bash
   export PATH="/Users/用户名/anaconda3/envs/环境名/bin:$PATH"
   ```
   例如：
   ```bash
   export PATH="/Users/zhoushuanglong/anaconda3/envs/tradingagents/bin:$PATH"
   ```
2. 保存 `.zshrc`，然后执行：
   ```bash
   source ~/.zshrc
   conda activate tradingagents
   which python
   python --version
   ```
   此时应输出 conda 环境下的 python 路径和正确的版本号（如 3.13.x）。

### 方法二：临时移除 pyenv shims（不推荐长期使用）

激活 conda 环境后，执行：

```bash
export PATH=$(echo $PATH | tr ':' '\n' | grep -v 'pyenv/shims' | paste -sd:)
```

## 5. 总结与注意事项

- 同时使用 pyenv 和 conda 时，PATH 顺序极其重要，pyenv 的 shims 路径不能排在 conda 环境 bin 之前。
- 推荐只在需要时用 pyenv，平时开发建议以 conda 环境为主。
- 检查 PATH 顺序、which python、python --version 是排查此类问题的关键。
- shell 配置文件建议只用 `.zshrc`，不要在 zsh 里 source `.bash_profile`，避免 PATH 被多次覆盖。

---

如遇到类似问题，可参考本方案快速定位和解决。
