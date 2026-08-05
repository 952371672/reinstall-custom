# 自用 reinstall 项目（DD + FRP 保活版）

本目录基于 `bin456789/reinstall`，增加了 DD 后的 FRP 紧急持久化：

1. 临时 Alpine/initramfs 先启动 FRP，用于查看重装进度。
2. `dd_raw_with_extract` 成功写完目标磁盘后，立即挂载目标系统分区。
3. 立即把 `frpc`、FRP 配置和 `frpc.service` 写入目标 systemd 系统。
4. 立即执行 `systemctl enable frpc.service`，然后才继续 resize 和其它系统修改。
5. `frpc.service` 使用 `Restart=always`，开机和网络延迟时都会持续重试。

## 发布到自己的 GitHub

把整个目录上传到自己的仓库，例如：

```text
https://github.com/你的用户名/reinstall-custom
```

然后修改 `reinstall.bat` 顶部：

```bat
set confhome=https://raw.githubusercontent.com/你的用户名/reinstall-custom/main
set confhome_cn=https://raw.githubusercontent.com/你的用户名/reinstall-custom/main
```

如果国内访问 GitHub 不稳定，可以把 `confhome_cn` 改为自己的国内 Git 镜像 raw 地址。

## 使用

管理员 CMD 中运行：

```bat
reinstall.bat dd --img="你的镜像地址" --frpc-config C:\frpc.toml --username niujia --password "你的密码"
```

如果是本地镜像路径，按上游项目支持的格式填写 `--img`。

## 保活边界

该修改能覆盖：

- DD 写盘完成后，后续 resize/系统修改阶段异常；
- 重装过程中的后续脚本错误；
- 目标系统已经写入并启用 FRP 后发生重启。

无法保证 DD 正在写入磁盘时突然断电，因为此时目标系统可能还没有完整写入，无法启动任何永久服务。

## 重要说明

- 使用 `--frpc-config` 时，目标系统会按配置安装永久 FRP。
- 如果目标系统不是 systemd，当前自定义的“提前持久化”步骤会保留临时 FRP，但不会冒险写入未知系统；上游后续流程仍会按其原逻辑处理。
- 每次上游更新 `reinstall.sh`/`trans.sh` 后，需要重新把本目录的自定义改动合并进去。
