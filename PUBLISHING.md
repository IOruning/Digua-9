# 发布到 GitHub 的步骤（PUBLISHING.md）

这三个仓库目前都在本地（`nodehub/`、`rdk-x5-webrtc/`、`web-client/` 互为兄弟目录），`.gitmodules` 里 submodule 的 url 暂时指向本地相对路径，方便先在本地验证结构是否正确。正式发布前需要按下面顺序操作。

## 0. 环境准备

推荐直接用 **PowerShell**：这三个文件夹本来就在 Windows 的 G 盘上，PowerShell 里的 Git 直接读写 NTFS，不涉及路径转换，最省事。如果你更习惯 WSL Ubuntu 也完全可以，见下方"WSL 用户注意"。

检查/安装：

```powershell
git --version   # 没有的话去 https://git-scm.com/download/win 装 Git for Windows
gh --version    # 没有的话: winget install --id GitHub.cli
```

用 GitHub CLI 登录一次（比配置 SSH key 简单，登录后 `gh` 和普通 `git push` 都能用）：

```powershell
gh auth login
```

按提示选 `GitHub.com` → `HTTPS` → 用浏览器登录授权即可，登录状态会一直保留。

**WSL 用户注意**：G 盘在 WSL 里的路径是 `/mnt/g/项目/26嵌入式大赛/nodehub/opensource-release/`，`cd` 进去操作方式和下面完全一样；只是 Windows 路径下的文件权限位在 WSL 里可能显示怪异，属于正常现象，不影响 git 操作。两边选一个终端从头做到尾，不要中途切换。

## 1. 建三个空仓库并推送两个子模块

用 `gh repo create` 可以一步完成"建仓库 + 加 remote + 推送"，在对应文件夹里执行：

```powershell
cd G:\项目\26嵌入式大赛\nodehub\opensource-release\rdk-x5-webrtc
gh repo create rdk-x5-webrtc --public --source=. --remote=origin --push

cd ..\web-client
gh repo create web-client --public --source=. --remote=origin --push
```

`--public` 换成 `--private` 可以先建成私有仓库，比赛结束/确认无误后再在 GitHub 网页上改成 Public。

如果不用 `gh`，就去 GitHub 网页手动建两个**空**仓库（不要勾选自动生成 README/.gitignore/LICENSE），然后：

```powershell
cd G:\项目\26嵌入式大赛\nodehub\opensource-release\rdk-x5-webrtc
git remote add origin https://github.com/<你的账号或组织>/rdk-x5-webrtc.git
git push -u origin main

cd ..\web-client
git remote add origin https://github.com/<你的账号或组织>/web-client.git
git push -u origin main
```

## 2. 把主仓库的 submodule 地址改成真实远程地址

```powershell
cd ..\nodehub
git submodule set-url rdk-x5-webrtc https://github.com/<你的账号或组织>/rdk-x5-webrtc.git
git submodule set-url web-client    https://github.com/<你的账号或组织>/web-client.git
git submodule sync
```

`git submodule set-url` 会更新 `.gitmodules`，需要额外提交一次这个改动：

```powershell
git add .gitmodules
git commit -m "Point submodules to public remotes"
```

## 3. 建主仓库并推送

```powershell
gh repo create nodehub --public --source=. --remote=origin --push
```

或手动建仓库后：

```powershell
git remote add origin https://github.com/<你的账号或组织>/nodehub.git
git push -u origin main
```

## 4. 验证

找一个干净的空目录（别在 `opensource-release` 里面），执行：

```powershell
cd G:\项目\26嵌入式大赛
git clone --recurse-submodules https://github.com/<你的账号或组织>/nodehub.git nodehub-verify
```

确认两个 submodule 目录（`nodehub-verify\rdk-x5-webrtc`、`nodehub-verify\web-client`）都能正常拉到内容，`git -C nodehub-verify submodule status` 显示的是 GitHub 提交而不是本地占位。web-client 可以顺手跑一遍：

```powershell
cd nodehub-verify\web-client
npm install
npm run build
npm test
```

验证完这个目录可以直接删掉，不影响已推送的仓库。

## 关于匿名评审

- 三个仓库当前的 git 提交作者统一写的是 `地瓜⑨号 Team <opensource@example.invalid>`，不含学校、参赛者真实姓名/邮箱。推送前如果要改成你们自己的对外身份，可以在推送前用 `git commit --amend --reset-author` 调整，或者直接推送使用当前占位身份也可以。
- 推送到 GitHub 的账号/组织名本身如果和学校或个人强关联，注意单独确认是否符合比赛匿名要求；这一层不是本次整理范围内能自动处理的。
