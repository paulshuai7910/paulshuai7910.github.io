---
title: Package bug处理方式

date: 2024-07-12 10:11:49
tags:
---

# Package bug

## 1. 使用补丁工具

patch-package

1. 安装：npm install patch-package postinstall
2. 添加脚本：

```javascript
{
  "scripts": {
    "postinstall": "patch-package"
  }
}

```

3. 修改 node_modules 中的代码
4. 生成补丁文件
   修改完代码后，你需要使用 patch-package 生成一个补丁文件来记录你的修改。补丁文件会记录你对 node_modules 中依赖包的更改，并保存为 .patch 文件。

```bash
npx patch-package some-package

```

这将生成一个补丁文件，并将其保存在项目根目录下的 patches/ 文件夹中。补丁文件的命名格式通常是：<package-name>+<version>.patch

5. 确保补丁文件生效
   在补丁文件生成后，你不需要做任何手动的应用操作，因为 postinstall 脚本已经配置好了，它会确保每次执行 npm install 时自动应用补丁。

生成新文件：`<package-name>+<version>.patch`，保存在`patches`目录下
在项目的`package.json`文件中，在`scripts`部分添加一个`postinstall`脚本，例如：`"postinstall": "patch-package"`。这样每次安装依赖时，都会自动应用补丁

## 2. 手动修改并使用本地版本

1. 找到有问题的包在项目的`node_modules`目录中的位置。
2. 复制该包的整个目录到项目中的其他位置，比如`src/vendorPatched`。
3. 对复制出来的版本进行手动修改以修复 bug。
4. 在项目代码中，使用相对路径导入修改后的本地版本，而不是从`node_modules`中导入原始有问题的版本。例如，如果是一个 JavaScript 模块，可以使用`import { someFunction } from '../vendorPatched/some-package/some-file.js';`。

## 3. 使用 fork 和替代源

如果 package 是开源的，可以在代码平台进行 fork

对 fork 后的项目进行 bug 修复，推送上去

在 package.json 中将有问题的 package 依赖地址指向 fork 后的地址，例如：

```typescript
"some-package": "git+https://github.com/your-username/forked-package.git"
```

## 4. 通过脚本 copy 覆盖

还是用`postinstall`这个勾子，在这个勾子执行`cp`修改过的文件  `./node_modules/包名/原始文件`拷贝过去，最终`node_modules`下的文件就变成了修改后的文件了，比如：

```typescript
"scripts": {
    "postinstall": "cp ./patches/upload/* ./node_modules/antd/lib/"
}
```

修改引用
配置一个`webpack alias`别名，如`'原始文件的引用路径': '修改后文件的引用路径'`，使得最终修改后的文件被引用，如：

```typescript
resolve: {
      alias: {
          'antd/upload': path.resolve(__dirname, './patched/upload/*'),
      }
  }
```
