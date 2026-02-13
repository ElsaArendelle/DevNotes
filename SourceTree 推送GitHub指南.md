# SourceTree 推送GitHub指南

## 1. 在 GitHub 生成 Token
1. 登录 GitHub，点击右上角头像 -> **Settings**。
2. 在左侧菜单最底部找到 **Developer settings**。
3. 选择 **Personal access tokens** -> **Tokens (classic)**。
4. 点击 **Generate new token (classic)**。
5. **Note**（备注）：填写 `SourceTree`。
6. **Expiration**（过期时间）：建议选择 **No expiration**（永不过期）。
7. **Select scopes**（权限）：**必须勾选 `repo` (包含所有子项) 和 `workflow`**。
8. 点击底部的 **Generate token**。
9. **立即复制**生成的 Token（以 `ghp_` 开头的字符串），它只显示一次。

## 2. 在 SourceTree 中使用 Token

### 第一步：清除旧的错误凭据
1.在搜索区搜索并打开 **“凭据管理器”**。

2. 点击 **“Windows 凭据”**。
3. 在列表里找到 `git:https://github.com` (或者类似 `GitHub` 的条目)。
4. 点击它，选择 **删除 (Remove)**。

### 第二步：重新登录
1. 回到 SourceTree，直接点击推送（Push）或者拉取（Pull）。
2. SourceTree 会弹出一个登录框（或者是 Git 的登录框）。
3. **用户名**：输入你的 GitHub 用户名。
     > **如何查看用户名**：
     > 1. 登录 GitHub 网页。
     > 2. 点击右上角头像 -> **Your profile**。
     > 3. 查看浏览器地址栏，URL 最后一部分就是用户名（例如 `https://github.com/这里是用户名`）。
     > 4. 或者点击右上角头像，第一行加粗的是昵称，**第二行灰色小字**才是用户名。
4. **密码**：**粘贴**你在第1步生成的 Token (`ghp_...`)。
   - **注意**：千万**不要**输入你的 GitHub 登录密码，必须用 Token。

这样就配置完成了！以后它会自动记住这个 Token。
