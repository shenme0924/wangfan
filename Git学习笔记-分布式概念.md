# Git 学习笔记 —— 分布式概念与基础命令

## 一、集中式 vs 分布式版本工具

本质区别：**代码的完整历史副本存在哪里。**

| 维度 | 集中式（SVN） | 分布式（Git） |
|------|--------------|--------------|
| 历史副本 | 只有中央服务器有完整历史 | 每个本地仓库都有完整历史 |
| 本地状态 | 只是某个版本的工作副本 | 完整的 commits、branches、tags |
| 提交行为 | commit 必须联网，写入服务器 | commit 只写入本地，不需要联网 |
| 同步方式 | 无 | push 同步到远端，pull 同步到本地 |
| 服务器宕机 | 历史丢失 | 任何本地仓库都能恢复全部历史 |

> SVN 的本地没有历史，Git 的本地就是完整的历史。

## 二、Git 分布式 vs 微服务分布式

两者都叫"分布式"，但分布的对象完全不同。

### Git 分布式 —— 数据冗余维度

- 分布的是：**同一份数据的完整副本**
- 每个开发者本地都有完整的版本历史
- 节点间地位平等，可互相同步（push/pull）
- 来源学科：分布式存储 / P2P 系统

### 微服务分布式 —— 职责拆分维度

- 分布的是：**不同功能到不同进程**
- 每个服务只负责自己的业务领域（用户、订单、支付等）
- 节点间通过 HTTP/gRPC 协作，组合成完整系统
- 来源学科：分布式计算 / SOA（面向服务架构）

### 异同对比

| 维度 | Git 分布式 | 微服务分布式 |
|------|-----------|------------|
| 分布的是什么 | 完整数据副本 | 不同的业务职责 |
| 节点是否相同 | 相同，地位平等 | 不同，各司其职 |
| 节点间关系 | 可互相 push/pull，冗余容灾 | 通过协议协作，组合成系统 |
| 单点故障影响 | 无影响，任何副本可恢复 | 该功能不可用，其他服务正常 |
| 核心目的 | 容灾 + 离线协作 | 解耦 + 独立扩展 |

**相同点**：都满足"多节点 + 网络协作 + 共同目标"的分布式基本定义。

**不同点**：Git 分布的是数据副本（同质冗余），微服务分布的是业务职责（异质分工），两者完全正交，可在同一项目中并存。

## 三、"分布式"概念为什么容易混淆

"分布式"是一个宽泛的上位概念，不同子领域分布的对象不同：

| 子领域 | 分布的是什么 | 代表技术 |
|--------|-------------|---------|
| 分布式存储 | 数据冗余副本 | Git、HDFS |
| 分布式计算 | 计算任务 | MapReduce、Spark |
| 分布式服务 | 业务职责 | 微服务、SOA |
| 分布式锁 | 并发控制 | Redis、ZooKeeper |
| 分布式事务 | 跨服务一致性 | Seata、2PC |

都叫"分布式"是因为共享了"多节点网络协作"这个最小公约数，但分布的对象和解决的问题不同，属于不同子领域的概念。

---

## 四、Git 本地基础命令

### 4.1 初始化与日常操作

```bash
git init                    # 初始化仓库，生成 .git 目录
git status                  # 看当前状态（什么改了、什么暂存了）
git add <file>              # 工作区 → 暂存区
git add .                   # 暂存所有改动
git commit -m "描述信息"    # 暂存区 → 本地仓库
```

三个区域：
```
工作区 (Working Directory)
  ↓ git add
暂存区 (Staging Area)
  ↓ git commit
本地仓库 (Local Repository)
```

### 4.2 查看历史

```bash
git log                     # 完整提交历史
git log --oneline           # 一行一个提交，简洁
git log --oneline --graph   # 带分支图，看分支走向
git diff                    # 工作区 vs 暂存区（还没 add 的改动）
git diff --staged           # 暂存区 vs 本地仓库（已 add 还没 commit 的）
```

### 4.3 分支操作

```bash
git branch                  # 查看本地分支（* 号表示当前所在）
git branch <name>           # 创建分支
git checkout <name>         # 切换到分支
git checkout -b <name>      # 创建 + 切换（最常用）
git merge <name>            # 把 <name> 合并到当前分支
git branch -d <name>        # 删除分支
```

### 4.4 查看配置

```bash
git config user.email          # 查看邮箱
git config user.name           # 查看用户名
git config --list             # 看全部配置（全局+仓库）
git config --local --list     # 只看当前仓库的
git config --global --list    # 只看全局的
```

优先级：**仓库级 > 全局级**。仓库级配置在 `.git/config`，全局配置在 `~/.gitconfig`。

### 4.5 git reset 的三种模式

核心作用是**移动 HEAD 指针**，根据模式决定暂存区和工作区是否跟着变。

| 模式 | HEAD | 暂存区 | 工作区 | 典型场景 |
|------|------|--------|--------|---------|
| `--soft` | 回退 | 保留 | 保留 | 提交信息写错了，想重新 commit |
| `--mixed`（默认） | 回退 | 清空 | 保留 | add 错了文件，想重新 add |
| `--hard` | 回退 | 清空 | 清空 | 彻底丢弃提交和所有改动 |

```bash
git reset --soft HEAD~1     # 撤销 commit，保留所有改动在暂存区
git reset HEAD~1            # 撤销 commit + 撤销暂存，保留工作区
git reset --hard HEAD~1     # 彻底丢弃，回退后什么都不剩

git reset HEAD <file>       # 撤销单个文件的暂存（不影响其他文件）
git reset --hard <commit-id>  # 彻底回退到指定提交
```

> 一句话：**soft 留暂存、mixed 清暂存、hard 全丢弃**。

---

## 五、Fast-forward vs 非 Fast-forward

### Fast-forward（快进）

- 含义：中文"快进"，main 指针直接跳到 dev 的最新提交
- 条件：main 没有新提交，只有 dev 往前走
- 结果：无 merge commit，历史是**一条直线**
- 特点：看不出开过分支

```
合并前：C1 → C2 (main)
                  C3 (dev)

合并后：C1 → C2 → C3 (main, dev 一起)
```

### 非 Fast-forward（非快进）

- 条件：main 和 dev 都有各自的新提交，两条线分叉
- 结果：**产生 merge commit**，把两条线缝在一起，历史有**分叉**
- 特点：能看出谁在什么时候开了分支

```
合并前：    C4 (main)
          /
    C1 → C2
          \
            C3 (dev)

合并后：    C4 ────┐
          /        M (merge commit, main)
    C1 → C2       /
          \     /
            C3
```

### `--no-ff` 的意义

即使能 fast-forward，也强制产生 merge commit，**保留分支痕迹**：

```bash
git merge --no-ff dev      # 能快进也不快进，强制留分叉记录
git merge --ff-only dev    # 只允许快进，不能就报错
```

---

## 六、git merge 的四种情况

| 情况 | 触发条件 | 结果 | 你看到的 |
|------|---------|------|---------|
| **Already up to date** | 目标分支没新提交 | 无变化 | `Already up to date.` |
| **Fast-forward** | main 没动，只有分支往前走 | 指针快进，无 merge commit | `Fast-forward` |
| **非 FF 自动合并** | 两边都动了，但改的不冲突 | 产生 merge commit，自动合并成功 | 弹编辑器写合并信息 |
| **合并冲突** | 两边改了同一文件的同一区域 | 合并中断，等你手动解决 | `CONFLICT` + 冲突标记 |

### 冲突解决

冲突文件长这样：
```
<<<<<<< HEAD
当前分支的内容
=======
被合并分支的内容
>>>>>>> dev
```

解决方法：
1. 手动删掉 `<<<<<<<`、`=======`、`>>>>>>>` 标记
2. 保留想要的内容
3. `git add <file>` + `git commit`

> 一句话：**没动/快进/自动合并/冲突，就这四种。**

---

## 七、完整工作流示例

```bash
# 1. 初始化
git init

# 2. 创建文件后看状态
git status                    # → Untracked files: readme.txt

# 3. 暂存 + 提交
git add readme.txt            # → 暂存区
git commit -m "初始化项目"     # → 本地仓库

# 4. 改文件后看改动
git diff                      # → 红绿对比，看改了什么

# 5. 再提交
git add .
git commit -m "添加第二行内容"

# 6. 看历史
git log --oneline             # → 3fac78e 添加第二行内容
                                 94ce8cf 初始化项目

# 7. 开分支开发
git checkout -b feature       # 创建并切换到 feature 分支
git add . && git commit -m "feature: 新功能模块"

# 8. 切回 main 合并
git checkout main             # 切回主分支
git merge feature             # 合并 feature 分支

# 9. 看分支图
git log --oneline --graph --all
```

核心循环：**改代码 → status 看状态 → diff 看改动 → add 暂存 → commit 提交**。
