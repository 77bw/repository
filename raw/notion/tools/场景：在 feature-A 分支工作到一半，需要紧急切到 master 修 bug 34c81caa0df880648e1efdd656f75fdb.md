---
source_url: ""
ingested: 2026-04-29
sha256: 855e0d8b8d20783cae74a4e142cb2eb237778ea145b8ea396b26df2fbb6cb6a0
---

# 场景：在 feature-A 分支工作到一半，需要紧急切到 master 修 bug

# 1. 暂存 feature-A 的工作

git stash push -u -m "feature-A: 实现到一半的登录功能"

# 2. 切到 master

git checkout master

# 2. 切换到目标分支

git checkout target-branch

# 3. 在目标分支完成开发工作...

# （提交、推送等操作）

# 4. 切回原分支

git checkout original-branch

# 5. 恢复暂存的工作

git stash pop