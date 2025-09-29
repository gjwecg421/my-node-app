# my-node-app


# 工作流名称：显示在 GitHub Actions 页面上
name: Node.js CI

# 触发器：定义何时运行此工作流
on:
  push:
    # 当代码推送到以下分支时运行
    branches: [ "main", "master", "develop" ]
  pull_request:
    # 当针对以下分支创建或更新 Pull Request 时运行
    branches: [ "main", "master", "develop" ]

# 定义一个或多个任务 (Jobs)
jobs:
  # 任务名称：构建和测试 (Build and Test)
  build:
    # 运行环境：使用最新的 Ubuntu 操作系统作为运行器
    runs-on: ubuntu-latest
    
    # 策略：通过矩阵 (Matrix) 运行，以在多个 Node.js 版本下测试
    strategy:
      matrix:
        # 定义需要测试的 Node.js 版本列表
        node-version: [18.x, 20.x, 22.x]
        # 也可以在此处添加操作系统，例如 os: [ubuntu-latest, windows-latest]

    # 步骤 (Steps)：任务将按顺序执行以下操作
    steps:
    - name: ⬇️ Checkout 仓库代码
      # 使用官方 actions/checkout 动作将仓库代码克隆到运行器上
      uses: actions/checkout@v4
      
    - name: ⚙️ 设置 Node.js ${{ matrix.node-version }} 环境
      # 使用官方 actions/setup-node 动作安装指定的 Node.js 版本
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        # 启用缓存：缓存 node_modules，加速后续运行
        cache: 'npm' # 或 'yarn'，取决于你使用的包管理器
        
    - name: 📦 安装依赖
      # 执行包安装命令
      run: npm ci 
      # npm ci 比 npm install 更适合 CI 环境，因为它要求 package-lock.json 存在且完整
      
    - name: 🧹 运行代码风格检查 (Lint)
      # 运行 package.json 中定义的 lint 脚本
      run: npm run lint
      
    - name: 🧪 运行单元测试
      # 运行 package.json 中定义的 test 脚本
      run: npm test

    - name: 🏗️ 构建项目 (可选)
      # 运行 package.json 中定义的 build 脚本。如果项目是前端应用或需要编译，则运行此步骤
      run: npm run build
      if: success() # 只有前面的步骤成功时才执行此步骤
      
    - name: 📤 上传构建产物 (可选)
      # 如果有构建产物 (例如 /dist 文件夹)，将其上传为 Artifact，供后续部署 Job 或下载使用
      uses: actions/upload-artifact@v4
      if: success()
      with:
        name: my-node-app-build-${{ github.sha }} # Artifact 名称
        path: dist/ # 你的构建输出目录