---
title: Git 回滚命令快查表
date: 2024-02-25
categories: [技术, 教程]
tags: [git, 回滚]
---
### ⚡ 快查表（先收藏）

| 目的                                    | 命令 / 思路                                                   | 是否改写历史      |
| ------------------------------------- | --------------------------------------------------------- | ----------- |
| **工作区改坏了，想撤销**                        | `git restore <file>` 或 `git checkout -- <file>`           | ❌           |
| **add 多了，想撤销暂存**                      | `git restore --staged <file>` 或 `git reset HEAD <file>`   | ❌           |
| **commit 错了，只回退一次，**<br>**但保留工作区改动**  | `git reset --soft HEAD~1`                                 | ✅（仅本地）      |
| **commit 错了，回退并扔掉**<br>**工作区改动**      | `git reset --hard HEAD~1`                                 | ✅（仅本地）      |
| **已经 push 到远程，想“反做”**<br>**某次或某几次提交** | `git revert <commit>`                                     | ❌（生成新提交）    |
| **整个分支回到远古版本，**<br>**并强制同步到远程**       | `git reset --hard <旧commit>`  <br>`git push origin +HEAD` | ✅（危险，需团队同步） |


### 1. 本地还没 commit——只改工作区
```bash
# 把 file.java 改回暂存区（或 HEAD）的样子
git restore file.java
```
> Git 2.23+ 推荐用 `restore`，老版本用 `git checkout -- file.java`。

---

### 2. 已经 add 但没 commit——撤销暂存
```bash
git restore --staged file.java      # Git 2.23+
# 或
git reset HEAD file.java
```
文件内容不动，只是从“待提交”变回“已修改”。

---

### 3. 刚 commit 了一次，想“抹掉”这次
```bash
# 方式 A：保留改动（回到暂存区）
git reset --soft HEAD~1

# 方式 B：连改动也扔掉（谨慎）
git reset --hard HEAD~1
```
> 只是本地操作，**push 之前随便玩**。

---

### 4. 已经 push 了，再改历史会冲突——用“反做”
```bash
# 生成一次“反向提交”抵消错误
git revert 34f8a1d
git push
```
远程历史不会被改写，**团队安全**。

---

### 5. 必须让整个分支回到旧版本（已 push）
```bash
# 本地回到旧 commit
git reset --hard 1a2b3c4
# 强制推到远程（所有人需重新拉取）
git push origin +HEAD
```
⚠️ **高风险**：协作者如果已基于新提交继续开发，会混乱；提前沟通！

---

### 6. 回滚 merge 提交
merge commit 有两个 parent，revert 时要指定主线：
```bash
git revert -m 1 <merge-commit>
```
`-m 1` 表示保留“主分支”那一边的更改。

---

### 7. 想“找回”被回滚的代码
任何回滚（reset/revert）**都不会立即删除对象库**；  
用下面命令可找回：
```bash
git reflog
# 找到之前的 HEAD 编号
git checkout 1a2b3c4
```

---

### ✅ 总结
- **还在本地**：`git reset` 三部曲（--soft / --mixed / --hard）。  
- **已共享**：`git revert` 生成反向提交，**绝不改写历史**。  
- **极端情况**才用 `git push +HEAD` 强制回退，**务必团队同步**。
