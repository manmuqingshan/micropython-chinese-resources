# 命令行固件更新工具 circfirm

用于更新 CircuitPython 开发板固件的 CLI 工具。

## 安装

安装 Circfirm 的最佳方式是使用 `pipx`，它为依赖创建了一个隔离的虚拟环境：

```
pipx install circfirm
```

如果确定依赖不会带来问题，也可以直接用 `pip` 来安装：

```
pip install circfirm
```

## 使用方法

以下命令展示了 circfirm 的一些功能：

```
# Install a version of CircuitPython to a connected board
circfirm install 8.0.0

# Install a version of CircuitPython in French to a connected board
circfirm install 8.0.0 --language fr

# List all the cached (previously downloaded) CircuitPython versions
circfirm cache list

# List all the cached CircuitPython versions for a speciic board
circfirm cache list --board-id feather_m4_express

# Save a version of CircuitPython to the cache
# (You can also use the --language option here)
circfirm cache save feather_m4_express 8.0.0

# Clear the cached CircuitPython versions
circfirm cache clear

# You can use --board-id, --version, and --language options to further specify
# what firmwares should be cleared - this clears version 7.0.0 firmwares for
# all boards and in all languages
circfirm cache clear --version 7.0.0

# See help/information about circfirm or any specific command using --help
circfirm --help
circfirm install --help
circfirm cache save --help
```

## 相关链接

- [文档](https://circfirm.readthedocs.io/en/latest)
- [github 仓库](https://github.com/tekktrik/circfirm)
