# React 源码调试

## 关于前端开源项目的研究

貌似前端学习一段时间之后，我们其实都开始有意无意的开始研究一些优秀开源的项目代码。但是刚开始的时候通常都是不知道如何入手的。通常的源码项目都是需要 build 之后，才是我们平时使用的 module 包，所以我们无法直接通过修改项目的源码来实现调试。所以我们一般都需要 clone 或者 fork 相关开源项目的代码到本地来进行开发，但是呢，由于开源项目本身的源码不像我们平时的业务代码一样通过 webpack 实现热更新，能实时看到代码变更之后的影响。所以我们需要借助别的办法来完成代码的调试。而这个办法就是：

> **单元测试**

是的，单元测试。通常来说开源项目的源码，如果是非业务类型，类似 react 项目这样的项目，我们见到的 package.json 通常是这样：

```json
// 以下 package.json 由于展示，所以调整部分字段的位置
{
  "private": true,
  "version": "16.6.1",
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "build": "node ./scripts/rollup/build.js",
    "linc": "node ./scripts/tasks/linc.js",
    "lint": "node ./scripts/tasks/eslint.js",
    "lint-build": "node ./scripts/rollup/validate/index.js",
    "postinstall": "node node_modules/fbjs-scripts/node/check-dev-engines.js package.json && node ./scripts/flow/createFlowConfigs.js",
    "debug-test": "cross-env NODE_ENV=development node --inspect-brk node_modules/.bin/jest --config ./scripts/jest/config.source.js --runInBand",
    "test": "cross-env NODE_ENV=development jest --config ./scripts/jest/config.source.js",
    "test-fire": "cross-env NODE_ENV=development jest --config ./scripts/jest/config.source-fire.js",
    "test-prod": "cross-env NODE_ENV=production jest --config ./scripts/jest/config.source.js",
    "test-fire-prod": "cross-env NODE_ENV=production jest --config ./scripts/jest/config.source-fire.js",
    "test-prod-build": "yarn test-build-prod",
    "test-build": "cross-env NODE_ENV=development jest --config ./scripts/jest/config.build.js",
    "test-build-prod": "cross-env NODE_ENV=production jest --config ./scripts/jest/config.build.js",
    "flow": "node ./scripts/tasks/flow.js",
    "flow-ci": "node ./scripts/tasks/flow-ci.js",
    "prettier": "node ./scripts/prettier/index.js write-changed",
    "prettier-all": "node ./scripts/prettier/index.js write",
    "version-check": "node ./scripts/tasks/version-check.js"
  }
  "devDependencies": {
    ...
  },
  "devEngines": {
    "node": "8.x || 9.x || 10.x || 11.x"
  },
  "jest": {
    "testRegex": "/scripts/jest/dont-run-jest-directly\\.js$"
  },
}
```

是的，这个 package.json 中不存在我们熟悉的 **npm start** 或者 **npm run serve** 的相关命令，导致我们无法用平时一样的开发方式来完成源码的调试。但是如果我们稍加注意的话，我们便可以意识到 **npm run test** 命令能帮助我们解决相关的问题，是的，这就是我所说的另一个入口，单元测试。 嗯，如果你不了解单元测试，建议先阅读(此内容)[]哦。接着，我们已以下命令为例：

```js
{
  ...
  "scripts": {
    ...
    // 就是这个命令哦，但是每个项目都不一样，需要根据项目本身进行调整
    "debug-test": "cross-env NODE_ENV=development node --inspect-brk node_modules/.bin/jest --config ./scripts/jest/config.source.js --runInBand",
    ...
  }
  ...
}
```

在该命令中，我们可以看到熟悉的 **node --inspect-brk** 命令哦，当然如果你不熟悉的话，你可以先阅读(此篇文章)[]。该命令通常是用来对 nodejs 的代码调试的时候使用的。是的，我们同样可以使用的该命令来调试的开源项目。接着我们从 **node_modules/.bin/jest** 中知道测试工具使用的 jest，而 **--config ./scripts/jest/config.source.js --runInBand** 这段我们如果来了解过 jest 的话，其实知道这是 jest 的运行配置。那么通过此命令我们便可以方便的对 react 的源码进行调试了，而且是实时的哦，不需要经过 **npm run build** 命令之后，就能看到代码的更新之后的结果。

## IDE 调试

当然，我们不可能一直使用命令行来进行调试是吧，因为我们也不喜欢一直敲命令，那么我们便是需要使用 IDE 的调试工具了。嗯，相当抱歉的是这个用例是基于 vscode 来实现的。是的，vscode 的常用调试命令 F5，那么我们来看一下根据上面的命令来，来生成相关调试配置：

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "name": "vscode-jest-tests",
            "request": "launch",
            // 命令运行参数
            "args": [
                "${file}",
                "--config",
                "./scripts/jest/config.source.js",
                "--runInBand"
            ],
            "cwd": "${workspaceFolder}",
            "console": "integratedTerminal",
            "internalConsoleOptions": "neverOpen",
            "program": "${workspaceFolder}/node_modules/jest/bin/jest",
            "env": {
                "NODE_ENV": "development"
            }
        }
    ]
}
```

那么我们便可以开始愉快的调试了。当然别开心的太早哦，如果我们直接运行 vscode 的命令的话，其实是与运行 **npm run debug-test** 命令无疑的，而且在 window 中还会命中如下BUG：

```shell
PS D:\Coding\learn-react\react> yarn debug-test
yarn run v1.12.3
$ cross-env NODE_ENV=development node --inspect-brk node_modules/.bin/jest --config ./scripts/jest/config.source.js --runInBand
Debugger listening on ws://127.0.0.1:9229/bf276262-87c1-4dc3-8575-ee920cff4466
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.
D:\Coding\learn-react\react\node_modules\.bin\jest:2
basedir=$(dirname "$(echo "$0" | sed -e 's,\\,/,g')")
          ^^^^^^^

SyntaxError: missing ) after argument list
    at new Script (vm.js:79:7)
    at createScript (vm.js:251:10)
    at Object.runInThisContext (vm.js:303:10)
    at Module._compile (internal/modules/cjs/loader.js:656:28)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:699:10)
    at Module.load (internal/modules/cjs/loader.js:598:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:537:12)
    at Function.Module._load (internal/modules/cjs/loader.js:529:3)
    at Function.Module.runMain (internal/modules/cjs/loader.js:741:12)
    at startup (internal/bootstrap/node.js:285:19)
Waiting for the debugger to disconnect...
```

所以我们需要指定某个单元来进行测试，而不是运行直接测试，以下面的步骤为例：

- 进入对应模块的测试文件夹，以 React 的 setState 测试用例为例，如下图所示：

- 同样的，我们需要在对应的代码中，设置对应的调试拦截，或者是 debug 的代码，如下图所示：
