
[如何给 Next.js 项目配置代码格式化和校验（ESLint + Prettier + husky）本文介绍在 Nex - 掘金](https://juejin.cn/post/7268594193932533823)

### 开发配置

在开发时，配置项目自动化工具，有助于提升开发效率和统一代码风格、规范。使用 `ESLint` 给项目加上代码校验，在编写代码时即遵守规范提前发现错误，使用 `Prettier` 格式化代码让团队内不同成员风格一致，使用 `Git` 钩子在提交时校验提交内容和自动修复格式化等等。

#### `eslint`

`eslint` 是一个插件化的 JavaScript 代码检查工具，可以识别并报告代码问题，提高代码风格一致性和避免错误。

安装，
```bash
npm install --save-dev eslint
```

`.eslintrc.json` ，
```json
{
    "root": true,
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended"
    ],
    "parser": "@typescript-eslint/parser",
    "parserOptions": { "project": ["./tsconfig.json"] },
    "plugins": [
        "@typescript-eslint"
    ],
    "rules": {
        "@typescript-eslint/strict-boolean-expressions": [
            2,
            {
                "allowString" : false,
                "allowNumber" : false
            }
        ]
    },
    "ignorePatterns": ["src/**/*.test.ts", "src/frontend/generated/*"]
}
```

#### `prettier`

安装，
```bash
npm i --save-dev prettier eslint-config-prettier prettier-plugin-organize-imports prettier-plugin-tailwindcss
```

`.prettierrc.json` ，
```json
{
  "plugins": [
    "prettier-plugin-organize-imports",
    "prettier-plugin-tailwindcss"
  ],
  "tailwindFunctions": ["classNames"],
  "singleQuote": true,
  "trailingComma": "es5"
}
```


#### `husky` + `lint-staged`

使用 `husky` 和 `lin-staged` 可以在 `Git` 提交代码时对提交的部分进行 `ESLint` 的代码校验和 `prettier` 的格式化。

首先安装，
```bash
npm install --save-dev husky lint-staged
npx husky init
```

`.lintstagedrc.js` ，
```js

const path = require('path');

const buildEslintCommand = (filenames) =>
  `next lint --fix --file ${filenames
    .map((f) => path.relative(process.cwd(), f))
    .join(' --file ')}`;

module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand], // 这些格式的文件在提交时交给 ESLint 校验
  '**/*.{js,jsx,tsx,ts,less,md,json}': ['prettier --write'], // 这些格式的文件在提交时让 prettier 格式化
};
```

#### `.vscode`

`.vscode/extensions.json` ，
```json
{ 
  "recommendations": [ 
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "bradlc.vscode-tailwindcss"
  ] 
}
```

`.vscode/settings.json` ，
```json
{ 
  // Formatting using Prettier by default for all languages 
  "editor.defaultFormatter": "esbenp.prettier-vscode", 
  "editor.formatOnSave": true, 
  "editor.codeActionsOnSave": { 
    "source.fixAll.eslint": true 
  }, 
  // Formatting using Prettier for JavaScript, overrides VSCode default. "
  [javascript]": { 
    "editor.defaultFormatter": "esbenp.prettier-vscode" 
  }, 
  // Linting using ESLint. 
  "eslint.validate": [ 
    "javascript", 
    "javascriptreact", 
    "typescript", 
    "typescriptreact" 
  ], 
  // Enable file nesting. 
  "explorer.fileNesting.enabled": true, 
  "explorer.fileNesting.patterns": { 
    "*.ts": "$(capture).test.ts, $(capture).test.tsx", 
    "*.tsx": "$(capture).test.ts, $(capture).test.tsx" 
  } 
}
```


#### `commitlint`

安装，
```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

`commitlint.config.js` ，
```js
const config = {
    extends: ['@commitlint/config-conventional']
};

module.exports = config;
```

本地配置，
```BASH
npm pkg set scripts.commitlint="commitlint --edit"
echo "npm run commitlint \${1}" > .husky/commit-msg
```

