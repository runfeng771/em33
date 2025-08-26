# Render 部署数据库连接问题修复

## 🔍 问题诊断

### 错误信息
```
Error [PrismaClientInitializationError]: 
Invalid `prisma.emailAccount.findMany()` invocation:
error: Environment variable not found: DATABASE_URL.
```

### 根本原因
1. `.env` 文件中的 `DATABASE_URL` 路径格式不正确
2. Render 平台可能无法正确读取本地 `.env` 文件
3. SQLite 数据库文件路径在部署环境中可能不同

## ✅ 解决方案

### 1. 修复本地开发环境

#### 更新 `.env` 文件
```bash
# 原来的格式（有问题）
DATABASE_URL=file:/home/z/my-project/db/custom.db

# 修复后的格式
DATABASE_URL="file:./db/custom.db"
```

#### 验证数据库连接
```bash
# 生成 Prisma 客户端
npx prisma generate

# 推送数据库架构
npx prisma db push

# 测试连接
npm run dev
```

### 2. Render 平台部署配置

#### 方案 A: 使用 SQLite（推荐用于开发/测试）

在 Render 平台设置环境变量：
```env
DATABASE_URL="file:./db/custom.db"
NODE_ENV=production
```

#### 方案 B: 迁移到 PostgreSQL（推荐用于生产）

##### 步骤 1: 在 Render 创建 PostgreSQL 服务
1. 在 Render Dashboard 创建 PostgreSQL 服务
2. 获取数据库连接字符串

##### 步骤 2: 更新 Prisma 配置
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

##### 步骤 3: 设置环境变量
```env
DATABASE_URL="postgresql://username:password@hostname:port/database"
NODE_ENV=production
```

##### 步骤 4: 迁移数据
```bash
# 安装 PostgreSQL CLI
sudo apt-get install postgresql-client

# 导出 SQLite 数据
sqlite3 db/custom.db .dump > backup.sql

# 导入到 PostgreSQL
psql $DATABASE_URL < backup.sql
```

### 3. 数据库文件管理

#### SQLite 文件持久化
在 Render 平台，SQLite 文件需要持久化存储：

1. **使用 Render Disk**：
   - 创建 Render Disk 服务
   - 挂载到 `/var/data` 目录
   - 设置 `DATABASE_URL="file:/var/data/custom.db"`

2. **或者使用环境变量**：
   ```env
   DATABASE_URL="file:/opt/render/project/src/db/custom.db"
   ```

## 🚀 部署步骤

### 方案 A: SQLite 部署（简单快速）

1. **更新代码**
   ```bash
   # 更新 .env 文件
   echo 'DATABASE_URL="file:./db/custom.db"' > .env
   
   # 提交更改
   git add .env
   git commit -m "修复数据库连接路径"
   git push origin main
   ```

2. **Render 配置**
   - 在 Web Service 设置中添加环境变量：
     ```
     DATABASE_URL=file:./db/custom.db
     NODE_ENV=production
     ```

3. **验证部署**
   - 检查构建日志
   - 测试 API 接口
   - 验证数据库连接

### 方案 B: PostgreSQL 部署（生产推荐）

1. **创建 PostgreSQL 服务**
   - 在 Render 创建 PostgreSQL 服务
   - 等待服务启动完成

2. **更新项目配置**
   ```bash
   # 更新 Prisma schema
   # 将 provider 从 "sqlite" 改为 "postgresql"
   
   # 重新生成客户端
   npx prisma generate
   
   # 推送架构
   npx prisma db push
   ```

3. **设置环境变量**
   ```env
   DATABASE_URL=postgresql://user:pass@host:port/db
   NODE_ENV=production
   ```

4. **数据迁移**
   ```bash
   # 导出现有数据
   sqlite3 db/custom.db .dump > backup.sql
   
   # 导入到 PostgreSQL
   psql $DATABASE_URL < backup.sql
   ```

## 🔧 故障排除

### 1. 检查环境变量
```bash
# 本地测试
echo $DATABASE_URL

# 在代码中检查
console.log('DATABASE_URL:', process.env.DATABASE_URL);
```

### 2. 验证数据库文件
```bash
# 检查文件是否存在
ls -la db/custom.db

# 检查文件权限
ls -la db/
```

### 3. 测试 Prisma 连接
```javascript
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

async function test() {
  try {
    await prisma.$connect();
    console.log('✅ 连接成功');
    
    const accounts = await prisma.emailAccount.findMany();
    console.log(`✅ 找到 ${accounts.length} 个账号`);
    
  } catch (error) {
    console.error('❌ 连接失败:', error);
  } finally {
    await prisma.$disconnect();
  }
}

test();
```

### 4. 检查 Render 日志
- 查看 "Build" 日志：检查构建过程中的错误
- 查看 "Service" 日志：检查运行时错误
- 查看 "Events" 日志：检查部署状态

## 📋 部署清单

### SQLite 部署清单
- [ ] 更新 `.env` 文件中的 `DATABASE_URL`
- [ ] 提交代码到 GitHub
- [ ] 在 Render 设置环境变量
- [ ] 验证构建成功
- [ ] 测试 API 接口
- [ ] 验证数据库连接

### PostgreSQL 部署清单
- [ ] 创建 Render PostgreSQL 服务
- [ ] 更新 Prisma schema
- [ ] 重新生成 Prisma 客户端
- [ ] 设置环境变量
- [ ] 迁移数据（可选）
- [ ] 验证数据库连接
- [ ] 测试所有功能

## 🎯 推荐方案

### 开发/测试环境
使用 SQLite，配置简单，部署快速：
```env
DATABASE_URL="file:./db/custom.db"
```

### 生产环境
使用 PostgreSQL，更稳定和可靠：
```env
DATABASE_URL="postgresql://user:pass@host:port/db"
```

## 🚨 注意事项

1. **SQLite 限制**：
   - 不适合高并发访问
   - 单文件存储，有大小限制
   - 备份和恢复需要手动操作

2. **PostgreSQL 优势**：
   - 支持高并发
   - 自动备份和恢复
   - 更好的性能和稳定性

3. **Render 平台建议**：
   - 生产环境推荐使用 PostgreSQL
   - 开发环境可以使用 SQLite
   - 确保设置正确的环境变量

通过以上解决方案，应该能够成功解决 Render 平台的数据库连接问题！