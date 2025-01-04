---
title: Package bug处理方式

date: 2024-07-12 10:11:49
tags:
---

# Package bug

## 1. 使用补丁工具

patch-package

安装：npm install patch-package postinstall

使用步骤：发现有问题的包代码，进行修复，验证后运行：

```typescript
npx patch-package <package-name>
```

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
