---
title: GitHub Actions 升级与部署问题解决
date: 2025-03-15 16:28:08
tags: github action deployment
---

# GitHub Actions 升级与部署问题解决

在使用 GitHub Actions 部署项目时，遇到了构建和部署相关的问题。本文记录了问题的原因、解决方案以及升级后的配置。

---

## 问题描述

### 1. 构建报错

- **时间**：2025 年 1 月 14 日
- **错误信息**：

  ```bash
  Install Dependencies
  Process completed with exit code 1.
  ```

- **详细信息**：
  ```bash
  Your workflow is using a version of actions/cache that is scheduled for deprecation, actions/cache@v2.
  Please update your workflow to use either v3 or v4 of actions/cache to avoid interruptions.
  Learn more: https://github.blog/changelog/2024-12-05-notice-of-upcoming-releases-and-breaking-changes-for-github-actions/#actions-cache-v1-v2-and-actions-toolkit-cache-package-closing-down
  ```

### 2. 环境警告

- **警告信息**：
  ```bash
  ubuntu-latest pipelines will use ubuntu-24.04 soon.
  For more details, see https://github.com/actions/runner-images/issues/10636
  ```

---

## 问题分析

1. **actions/cache 版本过时**：

   - 当前使用的是 `actions/cache@v2`，但该版本即将被弃用，需要升级到 `v3` 或 `v4`。

2. **Node.js 版本过低**：

   - 使用的 Node.js 版本为 `16`，建议升级到更高版本（如 `20`）以获得更好的支持。

3. **GitHub Actions 部署插件版本过时**：
   - 使用的 `actions/deploy-pages@v2` 版本较旧，建议升级到 `v4`。

---

## 解决方案

### 1. 升级构建脚本

#### 修改前

```yaml
node-version: "16"
uses: actions/cache@v2
uses: actions/upload-pages-artifact@v2
```

#### 修改后

```yaml
node-version: "20"
uses: actions/cache@v4
uses: actions/upload-pages-artifact@v3
```

### 2. 升级部署脚本

#### 修改前

```yaml
steps:
  - name: Deploy to GitHub Pages
    id: deployment
    uses: actions/deploy-pages@v2
```

#### 修改后

```yaml
steps:
  - name: Deploy to GitHub Pages
    id: deployment
    uses: actions/deploy-pages@v4
```

---

## 结果

- **构建成功**：升级后，`build` 阶段顺利完成。
- **部署成功**：`actions/deploy-pages@v4` 成功将静态文件部署到 GitHub Pages。

---

## 总结

通过升级 GitHub Actions 的相关插件和配置文件，解决了构建和部署失败的问题。以下是关键点：

1. 定期检查 GitHub Actions 插件的版本更新。
2. 使用最新的 Node.js 和 Ubuntu 环境以避免兼容性问题。
3. 参考官方文档和变更日志，及时调整工作流配置。

---

## 参考链接

- [GitHub Actions 官方文档](https://docs.github.com/en/actions)
- [actions/cache 更新公告](https://github.blog/changelog/2024-12-05-notice-of-upcoming-releases-and-breaking-changes-for-github-actions/#actions-cache-v1-v2-and-actions-toolkit-cache-package-closing-down)
- [Ubuntu 24.04 更新详情](https://github.com/actions/runner-images/issues/10636)

```

---
```
