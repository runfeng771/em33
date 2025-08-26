# Render 部署指南

## 🚀 部署前准备

### 1. 依赖版本冲突解决

已解决 `next-auth` 与 `nodemailer` 的版本冲突问题：

- **问题**: `next-auth@4.24.11` 需要 `nodemailer@^6.6.5`，但项目使用 `nodemailer@7.0.5`
- **解决方案**: 将 `nodemailer` 降级到 `^6.6.5`
- **配置**: 添加 `.npmrc` 文件设置 `legacy-peer-deps=true`

### 2. 项目结构

```
/home/z/my-project/
├── src/
│   ├── app/
│   │   ├── page.tsx                    # 主页面
│   │   ├── api/                        # API 路由
│   │   │   ├── refresh-all/route.ts    # 强制刷新全部账号
│   │   │   ├── force-refresh/route.ts  # 强制刷新单个账号
│   │   │   ├── send/route.ts          # 发送邮件
│   │   │   └── ... (其他 API)
│   ├── lib/
│   │   ├── cache.ts                    # 邮件缓存服务
│   │   ├── email-service.ts           # 邮件服务
│   │   ├── db.ts                      # 数据库连接
│   │   └── ...
│   └── components/
│       └── ui/                        # UI 组件
├── prisma/
│   └── schema.prisma                  # 数据库模式
├── package.json                       # 项目依赖
├── .npmrc                             # npm 配置
└── server.ts                          # 服务器入口
```

## 📦 Render 平台部署配置

### 1. 基本配置

在 Render 平台上创建新的 Web Service：

- **环境**: Node.js
- **构建命令**: `npm install && npm run build`
- **启动命令**: `npm start`
- **Node.js 版本**: 推荐 18.x 或 20.x

### 2. 环境变量设置

在 Render 平台的 Environment 标签页中设置以下环境变量：

```bash
# 生产环境
NODE_ENV=production

# 数据库连接 (使用 Render 的 PostgreSQL)
DATABASE_URL=postgresql://username:password@hostname:port/database

# 邮件服务配置 (可选)
EMAIL_SERVER_HOST=your-email-server.com
EMAIL_SERVER_PORT=587
EMAIL_SERVER_USER=your-email@domain.com
EMAIL_SERVER_PASS=your-email-password
```

### 3. 构建优化

项目已配置以下优化：

- **依赖冲突解决**: `.npmrc` 文件设置 `legacy-peer-deps=true`
- **构建缓存**: Next.js 自动优化构建过程
- **静态资源**: 自动生成静态页面和优化资源

## 🔧 本地测试

### 1. 安装依赖

```bash
npm install
```

### 2. 构建测试

```bash
npm run build
```

### 3. 启动生产服务器

```bash
npm start
```

## 🚀 部署步骤

### 1. 推送代码到 GitHub

确保所有代码已推送到 GitHub 仓库：

```bash
git add .
git commit -m "修复依赖版本冲突，优化部署配置"
git push origin main
```

### 2. 配置 Render 服务

1. 登录 [Render Dashboard](https://render.com/)
2. 点击 "New +" 创建 Web Service
3. 连接你的 GitHub 仓库
4. 选择项目分支（通常是 main 或 master）
5. 配置以下设置：

   - **Name**: `nextjs-email-client`
   - **Environment**: `Node`
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `npm start`
   - **Instance Type**: 根据需求选择（推荐 Starter 或 Standard）

6. 点击 "Create Web Service"

### 3. 设置环境变量

在服务设置页面：

1. 点击 "Environment" 标签页
2. 添加以下环境变量：

   ```env
   NODE_ENV=production
   DATABASE_URL=your_postgresql_connection_string
   ```

3. 点击 "Save Changes"

### 4. 部署监控

- Render 会自动开始构建和部署
- 可以在 "Logs" 标签页查看构建日志
- 部署成功后，服务会在 "Events" 标签页显示为 "Live"

## 🐛 常见问题解决

### 1. 依赖冲突错误

如果遇到依赖冲突错误：

```bash
npm error ERESOLVE could not resolve
npm error While resolving: next-auth@4.24.11
npm error Found: nodemailer@7.0.5
```

**解决方案**:
- 确保 `package.json` 中 `nodemailer` 版本为 `^6.6.5`
- 确保 `.npmrc` 文件包含 `legacy-peer-deps=true`
- 删除 `node_modules` 和 `package-lock.json` 后重新安装

### 2. 构建失败

如果构建失败：

1. 检查 Node.js 版本是否兼容
2. 确保所有依赖都已正确安装
3. 查看构建日志中的具体错误信息

### 3. 数据库连接问题

如果数据库连接失败：

1. 检查 `DATABASE_URL` 环境变量是否正确
2. 确保 Render PostgreSQL 服务正在运行
3. 验证数据库连接字符串格式

## 📊 性能优化

项目已包含以下性能优化：

### 1. 邮件获取优化
- 智能搜索策略：区分未读和已读邮件
- 减少处理邮件数量：限制为最新100封
- 优化连接超时：设置合理的超时时间

### 2. 缓存优化
- 增量缓存更新：基于 `messageId` 去重
- 缓存保护机制：确保不会清空现有邮件
- 智能缓存管理：只添加新邮件

### 3. 用户体验优化
- 异步邮件发送：不阻塞用户界面
- 飘窗提示：友好的状态反馈
- 后台邮件检查：不影响用户操作

## 🔍 部署验证

部署成功后，验证以下功能：

1. **主页访问**: 访问 Render 提供的 URL
2. **邮件账号管理**: 添加、删除邮件账号
3. **邮件获取**: 刷新单个账号和全部账号
4. **邮件发送**: 撰写和发送邮件
5. **邮件缓存**: 确保邮件不会被意外清空

## 📈 监控和维护

### 1. 日志监控
- 定期检查 Render 的 "Logs" 标签页
- 监控错误和异常信息

### 2. 性能监控
- 观察邮件获取和发送的响应时间
- 监控内存和 CPU 使用情况

### 3. 数据库维护
- 定期清理过期邮件
- 监控数据库大小和性能

## 🎉 总结

通过以上配置，项目已准备好部署到 Render 平台。主要解决了：

1. ✅ **依赖版本冲突**: 降级 nodemailer 到兼容版本
2. ✅ **构建优化**: 配置 `.npmrc` 和构建流程
3. ✅ **性能优化**: 邮件获取、缓存和用户体验优化
4. ✅ **部署配置**: 完整的 Render 平台部署指南

现在可以安全地将项目部署到生产环境！