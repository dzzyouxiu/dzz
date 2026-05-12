# Git版本控制

## 一、Git基础

### 1. Git简介
- **Git**：分布式版本控制系统
- **特点**：
  - 分布式架构
  - 高效的分支管理
  - 强大的合并能力
  - 本地仓库和远程仓库

### 2. Git安装
- **Linux**：
  ```bash
  # CentOS
  yum install -y git
  
  # Ubuntu
  sudo apt install -y git
  ```

- **Windows**：
  下载并安装Git for Windows：[https://gitforwindows.org/](https://gitforwindows.org/)

- **macOS**：
  ```bash
  brew install git
  ```

### 3. Git配置
- **全局配置**：
  ```bash
  # 设置用户名
  git config --global user.name "Your Name"
  
  # 设置邮箱
  git config --global user.email "your.email@example.com"
  
  # 设置默认编辑器
  git config --global core.editor vim
  
  # 启用颜色
  git config --global color.ui true
  ```

- **查看配置**：
  ```bash
  git config --list
  git config user.name
  ```

## 二、Git基本操作

### 1. 仓库操作
- **初始化仓库**：
  ```bash
  # 在当前目录初始化
  git init
  
  # 克隆远程仓库
  git clone https://github.com/username/repository.git
  ```

- **查看仓库状态**：
  ```bash
  git status
  ```

### 2. 文件操作
- **添加文件**：
  ```bash
  # 添加单个文件
  git add file.txt
  
  # 添加所有文件
  git add .
  
  # 添加指定目录
  git add directory/
  ```

- **提交更改**：
  ```bash
  # 提交并添加提交信息
  git commit -m "Commit message"
  
  # 提交所有修改
  git commit -a -m "Commit message"
  
  # 修改上次提交
  git commit --amend -m "New commit message"
  ```

- **查看提交历史**：
  ```bash
  # 查看所有提交
  git log
  
  # 查看简洁历史
  git log --oneline
  
  # 查看指定文件的提交历史
  git log -- file.txt
  ```

### 3. 分支操作
- **查看分支**：
  ```bash
  # 查看本地分支
  git branch
  
  # 查看所有分支（包括远程）
  git branch -a
  ```

- **创建分支**：
  ```bash
  # 创建分支
  git branch feature-branch
  
  # 创建并切换分支
  git checkout -b feature-branch
  ```

- **切换分支**：
  ```bash
  git checkout master
  ```

- **删除分支**：
  ```bash
  # 删除本地分支
  git branch -d feature-branch
  
  # 强制删除分支
  git branch -D feature-branch
  ```

### 4. 合并操作
- **合并分支**：
  ```bash
  # 切换到目标分支
  git checkout master
  
  # 合并源分支
  git merge feature-branch
  ```

- **解决冲突**：
  - 当合并发生冲突时，Git会标记冲突文件
  - 手动编辑冲突文件，解决冲突
  - 标记冲突已解决：`git add file.txt`
  - 完成合并：`git commit -m "Resolve merge conflict"`

### 5. 远程仓库操作
- **查看远程仓库**：
  ```bash
  git remote -v
  ```

- **添加远程仓库**：
  ```bash
  git remote add origin https://github.com/username/repository.git
  ```

- **推送更改**：
  ```bash
  # 推送分支
  git push origin master
  
  # 推送并设置上游分支
  git push -u origin feature-branch
  ```

- **拉取更改**：
  ```bash
  # 拉取并合并
  git pull origin master
  
  # 仅拉取
  git fetch origin
  ```

- **克隆远程仓库**：
  ```bash
  git clone https://github.com/username/repository.git
  ```

## 三、Git高级操作

### 1. 标签管理
- **创建标签**：
  ```bash
  # 创建轻量标签
  git tag v1.0
  
  # 创建附注标签
  git tag -a v1.0 -m "Version 1.0"
  ```

- **查看标签**：
  ```bash
  git tag
  ```

- **推送标签**：
  ```bash
  git push origin v1.0
  # 推送所有标签
  git push origin --tags
  ```

### 2. 撤销操作
- **撤销工作区修改**：
  ```bash
  git checkout -- file.txt
  ```

- **撤销暂存区修改**：
  ```bash
  git reset HEAD file.txt
  ```

- **回退提交**：
  ```bash
  # 回退到指定提交
  git reset --hard HEAD~1
  
  # 回退到指定版本
  git reset --hard <commit-hash>
  ```

### 3. 贮藏操作
- **贮藏修改**：
  ```bash
  git stash
  ```

- **查看贮藏**：
  ```bash
  git stash list
  ```

- **应用贮藏**：
  ```bash
  # 应用最近的贮藏
  git stash apply
  
  # 应用指定的贮藏
  git stash apply stash@{0}
  ```

- **删除贮藏**：
  ```bash
  git stash drop stash@{0}
  ```

### 4. 子模块
- **添加子模块**：
  ```bash
  git submodule add https://github.com/username/submodule.git
  ```

- **克隆包含子模块的仓库**：
  ```bash
  git clone --recursive https://github.com/username/repository.git
  ```

- **更新子模块**：
  ```bash
  git submodule update --remote
  ```

## 四、Git工作流

### 1. Git Flow
- **分支类型**：
  - `master`：主分支，用于发布
  - `develop`：开发分支，集成功能
  - `feature/*`：功能分支，开发新功能
  - `release/*`：发布分支，准备发布
  - `hotfix/*`：热修复分支，修复生产问题

### 2. GitHub Flow
- **分支类型**：
  - `main`：主分支
  - `feature/*`：功能分支

- **流程**：
  1. 从main分支创建feature分支
  2. 提交更改
  3. 创建Pull Request
  4. 代码审查
  5. 合并到main分支
  6. 部署

### 3. GitLab Flow
- 结合了Git Flow和GitHub Flow的优点
- 强调环境分支

## 五、Git钩子

### 1. 客户端钩子
- **pre-commit**：提交前执行
- **prepare-commit-msg**：准备提交信息
- **commit-msg**：提交信息验证
- **post-commit**：提交后执行

### 2. 服务端钩子
- **pre-receive**：接收推送前执行
- **update**：更新分支前执行
- **post-receive**：接收推送后执行

## 六、Git最佳实践

### 1. 提交规范
- **提交信息格式**：
  ```
  <type>(<scope>): <subject>
  
  <body>
  
  <footer>
  ```

- **类型**：
  - `feat`：新功能
  - `fix`：修复bug
  - `docs`：文档更新
  - `style`：代码风格
  - `refactor`：重构
  - `test`：测试
  - `chore`：构建/依赖

### 2. 分支规范
- **功能分支**：`feature/功能名称`
- **修复分支**：`fix/问题描述`
- **发布分支**：`release/版本号`
- **热修复分支**：`hotfix/问题描述`

### 3. 代码审查
- 使用Pull Request/Merge Request
- 建立代码审查流程
- 制定代码审查标准

### 4. 忽略文件
- 使用`.gitignore`文件
- 忽略编译产物、依赖、配置文件等

## 七、推荐学习资源

### 书籍
- 《Pro Git》
- 《Git权威指南》
- 《GitHub入门与实践》

### 在线资源
- [Git官方文档](https://git-scm.com/doc)
- [GitHub学习资源](https://guides.github.com/)
- [GitLab学习资源](https://docs.gitlab.com/ee/gitlab-basics/)

### 视频教程
- [Git教程 - 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)
- [GitHub视频教程](https://youtube.com/github)
- [GitLab视频教程](https://youtube.com/gitlab)

### 练习平台
- [Learn Git Branching](https://learngitbranching.js.org/)
- [GitHub Learning Lab](https://lab.github.com/)

## 八、实践项目

### 项目一：Git仓库管理
- 创建一个Git仓库
- 实现分支管理
- 处理合并冲突
- 与远程仓库同步

### 项目二：开源贡献
- Fork一个开源项目
- 创建功能分支
- 提交Pull Request
- 响应代码审查

### 项目三：Git工作流实践
- 实现Git Flow工作流
- 发布版本
- 处理热修复

## 九、总结

通过本章节的学习，你应该已经掌握了以下Git技能：

1. **Git基础操作**：
   - 仓库管理
   - 文件操作
   - 分支管理
   - 远程仓库操作

2. **Git高级操作**：
   - 标签管理
   - 撤销操作
   - 贮藏操作
   - 子模块管理

3. **Git工作流**：
   - Git Flow
   - GitHub Flow
   - GitLab Flow

4. **Git最佳实践**：
   - 提交规范
   - 分支规范
   - 代码审查
   - 忽略文件

Git是DevOps中最基础也是最重要的工具之一，掌握Git将为你的DevOps之旅打下坚实的基础。