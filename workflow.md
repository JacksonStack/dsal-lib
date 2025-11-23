workflow by kimi

------------------------------------------------
0. 前置全局配置（只做一次）
------------------------------------------------
```bash
# 让以后所有 git pull/push 自动递归子模块（git 2.34+）
git config --global submodule.recurse true
git config --global push.recurseSubmodules check   # 子模块未推会拒绝
```

------------------------------------------------
1. 首次创建主仓库并引入子模块
------------------------------------------------
```bash
# ① 建主仓库
mkdir ds-lib && cd ds-lib && git init
echo "# ds-lib" >README.md && git add . && git commit -m "init"

# ② 新建空仓库 github.com/yourid/dht-rust（网页点一下）

# ③ 引入为 submodule
git submodule add -b main https://github.com/yourid/dht-rust dht/rust
git commit -m "add dht/rust submodule"
```

------------------------------------------------
2. 同事第一次克隆（必带 recursive）
------------------------------------------------
```bash
git clone --recursive https://github.com/yourid/ds-lib
# 如果忘了加，事后补救：
git submodule update --init --recursive
```

------------------------------------------------
3. 日常开发：我改的是子模块代码
------------------------------------------------
```bash
cd dht/rust
git switch main               # 确保在分支上，而不是游离 HEAD
...coding...
git add . && git commit -m "feat: add finger table"
git push origin main          # ① 必须先推子模块
cd ../..                      # 回到主仓库
git add dht/rust              # ② 更新指针
git commit -m "chore: bump dht/rust pointer"
git push origin main          # ③ 再推主仓库
```

------------------------------------------------
4. 日常开发：我改的是主仓库文档，没动子模块
------------------------------------------------
```bash
echo "- new algo" >>README.md
git add README.md
git commit -m "docs: update readme"
git push
# 子模块指针没变，无需额外操作
```

------------------------------------------------
5. 拉取同事更新（最常见冲突来源）
------------------------------------------------
```bash
git pull --recurse-submodules        # 自动更新指针 + 拉子模块
# 如果指针冲突：
git merge origin/main                # 先解决主仓库冲突
git submodule update --recursive     # 再同步子模块代码
```

------------------------------------------------
6. 切换分支（含子模块指针变化）
------------------------------------------------
```bash
git switch feature/x --recurse-submodules
# 如果忘记加，事后补救：
git submodule update --recursive
```

------------------------------------------------
7. 子模块需要切分支做实验
------------------------------------------------
```bash
cd dht/rust
git switch -b experiment
...coding...
git push -u origin experiment
# 主仓库暂不关心，指针仍指向原 main；实验完 merge 回 main 再走第 3 步
```

------------------------------------------------
8. 删除某个子模块（完全移除）
------------------------------------------------
```bash
git submodule deinit -f dht/rust         # ① 清除 git 配置
rm -rf .git/modules/dht/rust             # ② 删除本地裸库
git rm -f dht/rust                       # ③ 删除工作区目录
git commit -m "remove dht/rust submodule"
```

------------------------------------------------
9. 把已有独立仓库嵌进来（迁移场景）
------------------------------------------------
```bash
# 假设已有一个成熟仓库 github.com/yourid/sort-rust
git submodule add -b main https://github.com/yourid/sort-rust sort/rust
git commit -m "add sort/rust submodule"
```

------------------------------------------------
10. CI 完整模板（GitHub Actions，可照抄）
------------------------------------------------
```yaml
name: ci
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive          # 关键行

      - uses: dtolnay/rust-toolchain@stable
      - run: |
          cd dht/rust
          cargo test --all-features
```

------------------------------------------------
11. 可视化检查指针状态（调试神器）
------------------------------------------------
```bash
git submodule status
# 输出示例
+3f4a2c1d8b9... dht/rust (heads/main)
```
最前面符号含义  
` ` 已同步  
`+` 子模块有更新但主仓库指针未提交  
`-` 子模块未初始化

------------------------------------------------
12. 一键批量更新所有子模块到上游最新
------------------------------------------------
```bash
git submodule update --remote --recursive
# 此时指针会走动，记得
git add .
git commit -m "chore: update all submodules to upstream"
```

------------------------------------------------
一张图总结（时间轴）
------------------------------------------------
```
clone  ──>  开发子模块  ──>  push 子模块  ──>  提交主仓库指针
  ↑              ↑              ↑
--recursive   switch main     push.recurseSubmodules=check
```
