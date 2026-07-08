# 发布到 GitHub 的步骤（PUBLISHING.md）

这三个仓库目前都在本地，`.gitmodules` 里 submodule 的 url 暂时指向本地路径，方便先在本地验证结构是否正确。正式发布前需要按下面顺序操作。

## 1. 建三个空仓库

在 GitHub（或其他代码托管平台）上新建三个**空**仓库，例如：

- `rdk-x5-webrtc`
- `web-client`
- `nodehub`（主仓库）

新建时不要勾选自动生成 README/.gitignore/LICENSE，保持完全空仓库。

## 2. 依次推送两个子模块

```bash
cd rdk-x5-webrtc
git remote add origin git@github.com:<你的账号或组织>/rdk-x5-webrtc.git
git push -u origin main

cd ../web-client
git remote add origin git@github.com:<你的账号或组织>/web-client.git
git push -u origin main
```

## 3. 把主仓库的 submodule 地址改成真实远程地址

```bash
cd ../nodehub
git submodule set-url rdk-x5-webrtc git@github.com:<你的账号或组织>/rdk-x5-webrtc.git
git submodule set-url web-client    git@github.com:<你的账号或组织>/web-client.git
git submodule sync
```

`git submodule set-url` 会更新 `.gitmodules`，需要额外提交一次这个改动：

```bash
git add .gitmodules
git commit -m "Point submodules to public remotes"
```

## 4. 推送主仓库

```bash
git remote add origin git@github.com:<你的账号或组织>/nodehub.git
git push -u origin main
```

## 5. 验证

在一台新机器上（或删掉本地目录重新 clone）执行：

```bash
git clone --recurse-submodules git@github.com:<你的账号或组织>/nodehub.git
```

确认两个 submodule 目录都能正常拉到内容，且指向的是 GitHub 地址而不是本地路径。

## 关于匿名评审

- 三个仓库当前的 git 提交作者统一写的是 `地瓜⑨号 Team <opensource@example.invalid>`，不含学校、参赛者真实姓名/邮箱。推送前如果要改成你们自己的对外身份，可以在推送前用 `git commit --amend --reset-author` 调整，或者直接推送使用当前占位身份也可以。
- 推送到 GitHub 的账号/组织名本身如果和学校或个人强关联，注意单独确认是否符合比赛匿名要求；这一层不是本次整理范围内能自动处理的。
