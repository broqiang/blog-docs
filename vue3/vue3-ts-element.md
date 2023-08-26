# 陪练小程序后台

采用的是 Vue3 + TypeScript + Vite + Pinia + VueRouter + ElementPlus 的技术栈

## 项目依赖

依赖了很多东西, 这里只列出来部分重点功能的插件

- [Vue3](https://cn.vuejs.org/)

- [Vite](https://cn.vitejs.dev/)

- [Vue Router](https://router.vuejs.org/zh/)

- [ElementPlus](https://element-plus.org/zh-CN/)

- [Pinia](https://pinia.vuejs.org/zh/)

## 项目初始化

通过 pnpm 来创建项目, 执行下面命令

```bash
pnpm create vite
```

然后选项 Vue 和 TypeScript

## 安装相关依赖

为了节省时间,一次性将项目计划使用的所有插件全部安装

```bash
# 运行环境的依赖
pnpm install element-plus @element-plus/icons-vue axios vue-router pinia \
pinia-plugin-persistedstate

# 开发环境的依赖
pnpm install -D eslint eslint-plugin-import eslint-plugin-vue eslint-plugin-node \
eslint-plugin-prettier eslint-config-prettier eslint-plugin-node @babel/eslint-parser \
eslint-plugin-prettier prettier eslint-config-prettier sass sass-loader \
stylelint postcss postcss-scss postcss-html stylelint-config-prettier \
stylelint-config-recess-order stylelint-config-recommended-scss \
stylelint-config-standard stylelint-config-standard-vue stylelint-scss \
stylelint-order stylelint-config-standard-scss husky husky-init lint-staged \
@commitlint/cli @commitlint/config-conventional @typescript-eslint/eslint-plugin \
unplugin-vue-components unplugin-auto-import vite-plugin-svg-icons
```

## 修改配置文件

此处是多个插件只要需要的一起配置了,就不再单独一个一个去配置

### vite.config.ts 配置

直接全部复制,替换

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

// 自动导入配置
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

// SVG 插件
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    // Element 的自动导入配置, 此步配置完成后 src 下的 components 也可以实现自动导入了
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),

    // Svg 图标插件配置
    createSvgIconsPlugin({
      // 指定需要缓存的图标文件夹
      iconDirs: [path.resolve(process.cwd(), 'src/assets/svgs')],
      // 指定symbolId格式
      symbolId: 'icon-[dir]-[name]',
    }),
  ],
  resolve: {
    alias: {
      // 相对路径别名配置, 使用 @ 代替 src
      '@': path.resolve('./src'),
    },
  },

  // scss 配置全局变量, 写入 variable.scss 的变量就可以全局使用了
  css: {
    preprocessorOptions: {
      scss: {
        javascriptEnabled: true,
        additionalData: '@import "./src/assets/styles/variable.scss";',
      },
    },
  },
})
```

### tsconfig.json 配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    // src 别名配置
    "baseUrl": "./",
    "paths": {
      // 路径映射, 相对于 baseUrl
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### package.json 配置

只替换 scripts 部分的内容

```json
"scripts": {
    "dev": "vite --open",
    "build:test": "vue-tsc && vite build --mode test",
    "build:prod": "vue-tsc && vite build --mode production",
    "preview": "vite preview",
    "lint": "eslint src",
    "lint:fix": "eslint src --fix",
    "lint:staged": "lint-staged",
    "lint:style": "stylelint src/**/*.{css,scss,vue} --cache --fix",
    "format": "prettier --write \"./**/*.{html,vue,ts,js,json,md}\"",
    "commitlint": "commitlint --config commitlint.config.cjs -e -V",
    "preinstall": "npx only-allow pnpm -y",
    "prepare": "husky install"
}
```

### 添加 eslint 配置

- 项目根目录下新建 `.eslintrc.cjs` 文件, 写入下面内容

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  // 规则继承
  extends: [
    // 全部规则默认是关闭的, 这个配置是开启推荐规则
    'eslint:recommended',
    // ts 语法规则
    'plugin:@typescript-eslint/recommended',
    // vue3 语法规则
    'plugin:vue/vue3-essential',
  ],
  overrides: [
    {
      env: {
        node: true,
      },
      files: ['.eslintrc.{js,cjs}'],
      parserOptions: {
        sourceType: 'script',
      },
    },
  ],
  // 解析器相关
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint', 'vue'],

  // 规则
  // "off" 或 0 ==> 关闭规则
  // "warn" 或 1 ==> 打开规则作为警告, 不影响代码执行
  // "error" 或 2 ==> 规则作为一个错误, 代码不能执行, 界面报错
  rules: {
    // https://eslint.org/docs/latest/rules/
    // 要求使用 let 或 const , 不能使用 var
    'no-var': 'error',
    // 不允许多个空行
    'no-multiple-empty-lines': ['warn', { max: 1 }],
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    // 禁止空余的多行
    'no-unexpected-multiline': 'error',
    // 禁止不必要的转移字符
    'no-useless-escape': 'off',
    // 不检查 ts 注释
    '@typescript-eslint/ban-ts-comment': 'off',

    // https://typescript-eslint.io/rules/
    // 禁止定义未使用的变量
    '@typescript-eslint/no-unused-vars': 'error',
    // 禁止使用 any 类型
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/no-namespace': 'off',
    '@typescript-eslint/semi': 'off',

    // https://eslint.vuejs.org/rules/
    // 关闭要求组件名称始终为 "-" 链接的单词
    'vue/multi-word-component-names': 'off',
    'vue/script-setup-uses-vars': 'error',
    'vue/no-mutating-props': 'off',
    'vue/attribute-hyphenation': 'off',
  },
}
```

- 项目根目录下新建 `.eslintignore` 文件, 写入下面内容

```bash
# eslint 不验证的目录
dist
node_modules
commitlint.config.cjs
```

### 添加 stylelint 配置

- 项目根目录下创建 `.stylelintrc.cjs` 文件, 写入下面内容

```js
// 中文文档 https://stylelint.bootcss.com/
// 英文文档 https://stylelint.io/

module.exports = {
  extends: [
    // 配置 stylelint 扩展插件
    'stylelint-config-standard',
    // 配置 vue 中 template 样式格式化
    'stylelint-config-html/vue',
    // 配置 stylelint scss 插件
    'stylelint-config-standard-scss',
    // 配置 vue 中 scss 样式格式化
    'stylelint-config-recommended-vue/scss',
    // 配置 stylelint css 书写顺序插件
    'stylelint-config-recess-order',
    // 配置 stylelint 和 prettier 兼容
    'stylelint-config-prettier',
  ],
  overrides: [
    {
      files: ['**/*.(scss|css|vue|html)'],
      customSyntax: 'postcss-scss',
    },
    {
      files: ['**/*.(html|vue)'],
      customSyntax: 'postcss-html',
    },
  ],
  ignoreFiles: [
    '**/*.js',
    '**/*.jsx',
    '**/*.tsx',
    '**/*.ts',
    '**/*.json',
    '**/*.md',
    '**/*.yaml',
  ],

  // 规则
  // null ==> 关闭该规则
  // always ==> 必须
  rules: {
    // 在 css 中使用 v-bind 不报错
    'value-keyword-case': null,
    // 禁止在具有较高优先级的选择器后出现被其覆盖
    'no-descending-specificity': null,
    // 禁止 URL 的引号, always 必须加上引号
    'function-url-quotes': 'always',
    // 关闭禁止空源码
    'no-empty-source': null,
    // 关闭强制选择器类名的格式
    'selector-class-pattern': null,
    // 禁止位置的属性, true 为不允许
    'property-no-unknown': null,
    // 大括号钱必须有一个空格
    // 'block-opening-brace-space-before': 'always',
    // 关闭属性前缀 --webkit-box
    'value-no-vendor-prefix': null,
    // 关闭属性前缀  --webkit-mask
    'property-no-vendor-prefix': null,
    'selector-pseudo-class-no-unknown': [
      // 不允许未知的选择器
      true,
      {
        ignorePseudoClasses: ['global', 'v-deep', 'deep'],
      },
    ],
  },
}
```

- 项目根目录下创建 `.stylelintignore` 文件, 写入下面内容

```bash
# stylelint 忽略的文件
/node_modules/*
/dist/*
/html/*
/public/*
```

### 添加 prettier 配置

- 项目根目录下创建 `.prettierrc.json` 文件, 写入下面内容

```json
{
  "singleQuote": true,
  "semi": false,
  "bracketSpacing": true,
  "htmlWhitespaceSensitivity": "ignore",
  "endOfLine": "auto",
  "trailingComma": "all",
  "tabWidth": 2
}
```

- 项目根目录下创建 `.prettierignore` 文件, 写入下面内容

```
/dist/*
/html/*
.local
/node_modules/**
**/*.svg
**/*.sh
/public/*
```

### 添加 env 配置

项目根目录下新建 `.env.development` `.env.test` `.env.production` 3个配置文件,
写入下面内容, 这里只写了 development 的内容,其他类似,根据实际情况写, 注意 `NODE_ENV`
变量要个文件后缀对应上

```bash
NODE_ENV = 'development'

# 注意!!!!!!!! 变量必须以 VITE_ 为前缀,才能暴露给外部读取

# 项目名称
VITE_APP_TITLE = 'UpUp Admin 开发环境'

# 接口服务器的地址
VITE_API_URL = 'http://localhost:8084'
VITE_API_PATH = '/mini'
```

### 添加 lintstaged 配置

项目根目录下添加 `.lintstagedrc` 文件, 写入下面内容

```js
{
  "src/**/*.{ts,vue}": [
    "prettier --write",
    "eslint",
    "stylelint"
  ]
}
```

### 初始化 commitlint 配置

项目根目录下新建 `commitlint.config.cjs` 文件, 写入下面内容

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],

  // 校验规则
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat', // 新增功能
        'fix', // bug 修复
        'docs', // 文档更新
        'style', // 不影响程序逻辑的代码修改(修改空白字符，补全缺失的分号等)
        'refactor', // 重构代码(既没有新增功能，也没有修复 bug)
        'perf', // 性能优化
        'test', // 增加测试
        'chore', // 构建过程或辅助工具的变动
        'revert', // 代码回滚
        'build', // 编译相关,例如发布版本,对项目构建或者依赖的改动
      ],
    ],
    'type-case': [0],
    'type-empty': [0],
    'scope-empty': [0],
    'scope-case': [0],
    'subject-full-stop': [0, 'never'],
    'subject-case': [0, 'never'],
    'header-max-length': [0, 'always', 72],
  },
}
```

### 初始化 husky 并配置

```bash
# husky 必须在 git 仓库初始化完才可以配置
git init

# 执行完会创建 .husky/pre-commit
npx husky-init
# 执行完会创建 .husky/commit-msg
npx husky add .husky/commit-msg
```

- pre-commit 配置

删除 `npm test`, 添加 `pnpm lint:staged`

- commit-msg 配置

删除 `undefined`, 添加 `pnpm commitlint`

## 修改 App.vue

将 `src/App.vue` 中的内容删除, 写入下面内容

```vue
<script setup lang="ts">
// 配置全局本地化
// @ts-ignore
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'
const locale = zhCn
</script>

<template>
  <!-- 配置全局本地化 -->
  <el-config-provider :locale="locale">
    <!-- 路由 -->
    <router-view></router-view>
  </el-config-provider>
</template>

<style lang="scss" scoped></style>
```

## 创建一些初始的 views 模版

可以随意写,现在写了等项目开始时候肯定也需要修改的

创建 `src/views` 目录

分别创建 `src/views/home/index.vue` `src/views/login/index.vue` `src/views/404/index.vue`
,然后里面随意写些 vue 的内容, 主要用于配置路由使用,后面再根据实际需求去改

## 路由的基础配置

在 `src` 目录下创建 `router` 目录, 再添加两个文件 `index.ts` 和 `routes.ts`

- routes.ts 写入下面内容
  // 对外暴露的路由
  export const constantRoute = [
  // 登录
  {
  path: '/login',
  component: () => import('@/views/login/index.vue'),
  name: 'login', // 命名路由
  },
  // 登录成功
  {
  path: '/',
  component: () => import('@/views/home/index.vue'),
  name: 'home', // 命名路由
  },
  // 404
  {
  path: '/404',
  component: () => import('@/views/404/index.vue'),
  name: '404', // 命名路由
  },
  // 任意路由, 上面全都没有匹配上的路由
  {
  path: '/:pathMatch(.*)*',
  redirect: '/404',
  name: 'any', // 命名路由
  },
  ]

```ts

```

- index.tes 写入下面内容

```ts
import { createRouter, createWebHistory } from 'vue-router'
import { constantRoute } from './routes'

const router = createRouter({
  // 地址的风格
  history: createWebHistory(),
  // 路由切换的滚动
  routes: constantRoute,
  // 加载滚动条配置
  scrollBehavior() {
    return {
      left: 0,
      top: 0,
    }
  },
})

export default router
```

## SVG 图标配置

安装 和 `vite.config.ts` 中的配置前面已经做了,这里就不需要了

需要封装一个 SvgIcon 的插件, 方便使用

先将 `src/components/HelloWorld.vue` 删除, 然后创建一个 `src/components/SvgIcon/index.vue`
的组件,写入下面内容

```vue
<script setup lang="ts">
// 接收父组件传递过来的参数
defineProps({
  // xlink:href 前缀的名称
  prefix: {
    type: String,
    default: '#icon-',
  },
  // 图片的名称
  name: String,
  // 颜色
  color: {
    type: String,
    default: '',
  },
  width: {
    type: String,
    default: '16px',
  },
  height: {
    type: String,
    default: '16px',
  },
})
</script>

<template>
  <svg :style="{ width, height }">
    <use :xlink:href="prefix + name" fill="color" />
  </svg>
</template>

<style lang="scss" scoped></style>
```

以后使用的时候可以直接用就可以了, 如:

```vue
<svg-icon name="vue" color="black" width="32px" height="32px"></svg-icon>
```

PS: 需要注意, 这个 home.vsg 文件是要存在的, 并且是保存到 `src/assets/svgs` 目录中的,
可以将 `src/assets/vue.svg` 移动到这个目录用来测试

## 配置 scss

插件和 `vite.config.ts` 中的配置前面已经做了,这里就可以直接使用了

创建一个初始的 scss 样式, 创建 `src/assets/styles` 目录, 并在目录下创建下面3个文件:

- reset.scss 内容直接从 [https://www.npmjs.com/package/reset.scss?activeTab=code](https://www.npmjs.com/package/reset.scss?activeTab=code) 复制

- variable.scss 名字不要错, 因为 `vite.config.ts` 中注册的是这个,用于写全局变量

- index.scss 写入下面内容

```scss
// 引入清除默认样式
@import './reset.scss';

// 自动导入没有 ElMessage 的样式,手动导入了
@import 'element-plus/theme-chalk/src/message.scss';
```

## axios 二次封装

安装前面已经装了, 这里就不用再装

创建 `src/common` 目录, 在里面创建 `src/common/request/index.ts` 文件,写入下面内容

```ts
import axios from 'axios'
import { ElMessage } from 'element-plus'

const instance = axios.create({
  baseURL: import.meta.env.VITE_API_PATH,
  timeout: 5000,
})

// 请求拦截器
instance.interceptors.request.use((config) => {
  // TODO 这里处理请求的 token

  return config
})

// 响应拦截器
instance.interceptors.response.use(
  (response) => {
    return response.data
  },
  (error) => {
    // 失败回调: 处理 http 网络错误
    let message = ''
    const status = error.response.status

    switch (status) {
      case 401:
        message = 'token 过期或没有权限'
        break
      case 403:
        message = '没有访问权限'
        break
      case 404:
        message = '请求地址不存在'
        break
      case 500:
        message = '请求的服务器出现问题'
        break
      default:
        message = '请检查网络是否联通'
        break
    }
    ElMessage({
      type: 'error',
      message,
    })

    return Promise.reject(error)
  },
)

export default instance
```

## 配置 Pinia

安装前面已经做了,这里不再处理

- 创建 `src/stores/modules/user.ts` 文件, 用来做个初始化的 demo , 写入下面内容

```ts
import { defineStore } from 'pinia'

import { ref } from 'vue'

// 用户处理用户相关
export const useUserStore = defineStore(
  'user',
  () => {
    // token 相关
    const token = ref<string>('')
    const setToken = (newToken: string) => {
      token.value = newToken
    }

    const removeToken = () => {
      token.value = ''
    }

    // TODO 可以有 userInfo, 等对接完接口再处理

    return {
      token,
      setToken,
      removeToken,
    }
  },
  {
    // 开启持久化
    persist: true,
  },
)
```

- 创建 `src/stores/index.ts` , 用来在 main.ts 中注册 pinia, 写入下面内容

```ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

// 使用 pinia 并配置持久化
// https://prazdevs.github.io/pinia-plugin-persistedstate/zh/guide/
const pinia = createPinia().use(piniaPluginPersistedstate)

export default pinia

export * from './modules/user'
```

## 修改 main.ts

将 `main.ts` 中的内容删除, 写入下面内容

```ts
import { createApp } from 'vue'
import App from '@/App.vue'

// svg 插件配置
import 'virtual:svg-icons-register'
import router from '@/router'

// pinia 插件
import pinia from '@/stores'

// 自定义 scss
import '@/assets/styles/index.scss'

const app = createApp(App)

// 注册 pinia
app.use(pinia)

// 注册路由
app.use(router)

app.mount('#app')
```

## vscode 配置

最后附上 `.vscode/settings.json` 的配置,主要是忽略的单次拼写, 避免出现一堆单词拼写的警告,
看着也很头疼

```json
{
    "cSpell.words": [
        "commitlint",
        "lintstaged",
        "persistedstate",
        "pinia",
        "preinstall",
        "stylelint",
        "stylelintrc",
        "svgs",
        "unplugin"
    ]
}
```