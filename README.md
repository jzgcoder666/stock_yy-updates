# 版本清单（更新源）

`version.json` 是应用「设置 → 关于与更新」读取的唯一文件，**只含版本号、发布时间、说明和 Release 链接，不含任何凭证或台账数据**。它托管在独立的公开仓库 `jzgcoder666/stock_yy-updates`（本仓库保持 Private）。

## 发布流程（全自动）

1. 修改 `app/version.py` 的 `__version__`（如 `1.2.0`）。
2. 提交并推送 `main`。
3. 打标签并推送：

   ```bash
   git tag v1.2.0
   git push origin v1.2.0
   ```

`.github/workflows/release.yml` 会依次：

- 校验标签与 `app/version.py` 一致；
- 在 Windows / macOS 上构建并跑冒烟测试，产物上传为 Release 资产；
- 创建 GitHub Release；
- 用部署密钥（仓库 Secret `UPDATES_DEPLOY_KEY`）更新公开仓库里的 `version.json`。

## 首次配置

1. 公开仓库 `jzgcoder666/stock_yy-updates` 中放入本目录的 `version.json`（作为基线，版本号 = 当前发布的版本）。
2. 生成部署密钥并只授予该公开仓库写权限：

   ```bash
   ssh-keygen -t ed25519 -N "" -C "stock_yy-updates-deploy" -f /tmp/stock_yy_updates_deploy
   gh repo deploy-key add /tmp/stock_yy_updates_deploy.pub \
     -R jzgcoder666/stock_yy-updates --title "release workflow" --allow-write
   gh secret set UPDATES_DEPLOY_KEY -R jzgcoder666/stock_yy < /tmp/stock_yy_updates_deploy
   rm /tmp/stock_yy_updates_deploy /tmp/stock_yy_updates_deploy.pub
   ```

3. 未配置 `UPDATES_DEPLOY_KEY` 时，Release 仍会正常创建，只在日志里提示跳过版本清单更新。

## 客户端行为

- 启动时后台检查一次，结果缓存 6 小时；失败不缓存。
- 发现新版本时弹出一次提示（同一版本只提醒一次），设置页显示版本对比与下载链接。
- **只提醒，不自动下载或替换程序。**
- 可用环境变量 `STOCK_YY_UPDATE_URL` 指向其他地址（调试用）。