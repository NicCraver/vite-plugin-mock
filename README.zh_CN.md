# vite-plugin-mock

**中文** | [English](./README.md)

[![npm][npm-img]][npm-url] [![node][node-img]][node-url]

提供本地和生产模拟服务。

vite 的数据模拟插件，是基于 vite.js 开发的。 并同时支持本地环境和生产环境。 Connect 服务中间件在本地使用，mockjs 在线使用

### 安装 (yarn or npm)

**node version:** >=12.0.0

**vite version:** >=2.0.0-beta.4

`yarn add mockjs` or `yarn add mockjs -S`

and

`yarn add vite-plugin-mock -D` or `npm i vite-plugin-mock -D`

### 示例

**运行示例**

```bash

# ts example
cd ./examples/ts-examples

yarn install

yarn serve

# js example

cd ./examples/js-examples

yarn install

yarn serve
```

## 使用

**开发环境**

开发环境是使用 Connect 中间件实现的。

与生产环境不同，您可以在 Google Chrome 控制台中查看网络请求记录

- vite.config.ts 配置

```ts
import { UserConfigExport, ConfigEnv } from 'vite';

import { viteMockServe } from 'vite-plugin-mock';
import vue from '@vitejs/plugin-vue';

export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      vue(),
      viteMockServe({
        // default
        mockPath: 'mock',
        localEnabled: command === 'serve',
      }),
    ],
  };
};
```

- viteMockServe 配置

```ts
{
    mockPath?: string;
    supportTs?: boolean;
    ignore?: RegExp | ((fileName: string) => boolean);
    watchFiles?: boolean;
    localEnabled?: boolean;
    ignoreFiles?: string[];
    configPath?: string;
    prodEnabled?: boolean;
    injectFile?: string;
    injectCode?: string;
}
```

## 配置说明

### mockPath

**type:** string

**default:** 'mock'

设置模拟.ts 文件的存储文件夹

如果`watchFiles：true`，将监视文件夹中的文件更改。 并实时同步到请求结果

如果 `configPath` 具有值，则无效

### supportTs

**type:** boolean

**default:** true

打开后，可以读取 ts 文件模块。 请注意，打开后将无法监视.js 文件。

### ignore

**type:** RegExp | ((fileName: string) => boolean);

**default:** undefined

自动读取模拟.ts 文件时，请忽略指定格式的文件

### watchFiles

**type:** boolean

**default:** true

设置是否监视模拟.ts 文件中的更改

### localEnabled

**type:** boolean

**default:** command === 'serve'

设置是否启用本地 xxx.ts 文件，不要在生产环境中打开它.设置为 false 将禁用 mock 功能

### prodEnabled

**type:** boolean

**default:** command !== 'serve'

设置打包是否启用 mock 功能

### injectCode

**type:** string

**default:** ''

如果生产环境开启了 mock 功能,即`localEnabled=true`.则该代码会被注入到`injectFile`对应的文件的底部。默认为`main.ts`

这样做的好处是,可以动态控制生产环境是否开启 mock 且在没有开启的时候 mock.js 不会被打包。

如果代码直接写在`main.ts`内，则不管有没有开启,最终的打包都会包含`mock.js`

### injectFile

**type:** string

**default:** `path.resolve(process.cwd(), 'src/main.ts')`

`injectCode`代码注入的文件,默认为项目根目录下`src/main.ts`

### configPath

**type:** string

**default:** vite.mock.config.ts

设置模拟读取的数据条目。 当文件存在并且位于项目根目录中时，将首先读取并使用该文件。 配置文件返回一个数组

### ignoreFiles

**type:** string[]

**default:** []

该项目使用 glob 来读取模拟路径设置的文件夹。 此参数用作 glob 模块的参数

### showTime

**type** boolean|string

**default** `YYYY-MM-DD HH:mm:ss`

是否显示请求时间。 如果是字符串，使用 dayjs 格式化

## Mock file example

`/path/mock`

```ts
// test.ts

import { MockMethod } from 'vite-plugin-mock';
export default [
  {
    url: '/api/get',
    method: 'get',
    response: ({ query }) => {
      return {
        code: 0,
        data: {
        	name: 'vben'
        }
      };
    },
  },
   {
    url: '/api/post',
    method: 'post',
    timeout:2000,
    response: {
      	code: 0,
       data: {
        	name: 'vben'
        }
      };
  },
] as MockMethod[];


```

### MockMethod

```ts
{
  // request url
  url: string;
  // request method
  method?: MethodType;
  // Request time in milliseconds
  timeout?: number;
  // default: 200
  statusCode?:number;
  // response data
  response: ((opt: { [key: string]: string; body: Record<string,any>; query:  Record<string,any> }) => any) | any;
}

```

## 在生产环境中的使用

### 示例一(2.0.0-rc1 已经不推荐)

创建`mockProdServer.ts`文件

```ts
//  mockProdServer.ts

import { createProdMockServer } from 'vite-plugin-mock/es/createProdMockServer';

// 逐一导入您的模拟.ts文件
// 如果使用vite.mock.config.ts，只需直接导入文件
import testModule from '../mock/test';

export function setupProdMockServer() {
  createProdMockServer([...testModule]);
}
```

In mian.ts

```ts
// src/main.ts

import { setupProdMockServer } from './mockProdServer';

if (process.env.NODE_ENV === 'production') {
  setupProdMockServer();
}
```

### 示例二(2.0.0-rc1+ 推荐)

创建`mockProdServer.ts`文件

```ts
//  mockProdServer.ts

import { createProdMockServer } from 'vite-plugin-mock/es/createProdMockServer';

// 逐一导入您的模拟.ts文件
// 如果使用vite.mock.config.ts，只需直接导入文件
import testModule from '../mock/test';

export function setupProdMockServer() {
  createProdMockServer([...testModule]);
}
```

配置 `vite-plugin-mock`

```ts
import { viteMockServe } from 'vite-plugin-mock';

import { UserConfigExport, ConfigEnv } from 'vite';

export default ({ command }: ConfigEnv): UserConfigExport => {
  // 根据项目配置。可以配置在.env文件
  let prodMock = true;
  return {
    plugins: [
      viteMockServe({
        mockPath: 'mock',
        localEnabled: command === 'serve',
        prodEnabled: command !== 'serve' && prodMock,
        injectCode: `
          import { setupProdMockServer } from './mockProdServer';
          setupProdMockServer();
        `,
      }),
    ],
  };
};
```

## 示例项目

[Vben Admin](https://github.com/anncwb/vue-vben-admin)

## Note

- 无法在 mock.ts 文件中使用节点模块，否则生产环境将失败
-
- 模拟数据用于生产环境，仅适用于某些测试环境。 不要在正式环境中打开它，以避免不必要的错误。 同时，在生产环境中，它可能会影响正常的 Ajax 请求，例如文件上传失败等。

## License

MIT

[npm-img]: https://img.shields.io/npm/v/vite-plugin-mock.svg
[npm-url]: https://npmjs.com/package/vite-plugin-mock
[node-img]: https://img.shields.io/node/v/vite-plugin-mock.svg
[node-url]: https://nodejs.org/en/about/releases/
