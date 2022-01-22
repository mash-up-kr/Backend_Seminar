# Hot reload

발표 날짜: 2021년 11월 6일
발표 여부: Yes
발표자: 주성민

# Hot reload

- ts 컴파일 하는 것이 application bootstrap process에 가장 큰 영향을 미친다.
- 이 때,  webpack HMR(Hot-Module Replacement)을 사용하면 변경 사항이 있을 때마다 전체 프로젝트를 다시 컴파일 하지 않아도 된다.

그래서 왜 쓰나~~? 

→ 변경 사항만 찾아서 컴파일하기 때문에 개발 시간이 단축된다.

### With CLI

```bash
$ npm i --save-dev webpack-node-externals run-script-webpack-plugin webpack
```

그리고, 루트 디렉토리에 webpack-hmr.config.js 파일을 만든다.

```jsx
const nodeExternals = require('webpack-node-externals');
const { RunScriptWebpackPlugin } = require('run-script-webpack-plugin');

module.exports = function (options, webpack) {
  return {
    ...options,
    entry: ['webpack/hot/poll?100', options.entry],
    externals: [
      nodeExternals({
        allowlist: ['webpack/hot/poll?100'],
      }),
    ],
    plugins: [
      ...options.plugins,
      new webpack.HotModuleReplacementPlugin(),
      new webpack.WatchIgnorePlugin({
        paths: [/\.js$/, /\.d\.ts$/],
      }),
      new RunScriptWebpackPlugin({ name: options.output.filename }),
    ],
  };
};
```

- 이 구성은 웹 팩에 애플리케이션에 대한 몇 가지 필수 사항을 알려준다.
    - 어디에 엔트리 파일이 있고, **컴파일 된** 파일을 보유하기 위해 어떤 디렉토리를 사용해야 하며, 소스 파일을 컴파일하기 위해 어떤 종류의 로더를 사용하길 원하는지 등..

HMR을 활성화하기 위해 main.ts에 웹팩 지침 추가.

```tsx
declare const module: any;

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }
}
bootstrap();
```

그 다음에 개발용 스크립트에 추가

```json
"start:dev": "nest build --webpack --webpackPath webpack-hmr.config.js --watch"
```

- Without CLI
    
    ```bash
    $ npm i --save-dev webpack webpack-cli webpack-node-externals ts-loader run-script-webpack-plugin
    ```
    
    webpack.config.js
    
    ```jsx
    const webpack = require('webpack');
    const path = require('path');
    const nodeExternals = require('webpack-node-externals');
    const { RunScriptWebpackPlugin } = require('run-script-webpack-plugin');
    
    module.exports = {
      entry: ['webpack/hot/poll?100', './src/main.ts'],
      target: 'node',
      externals: [
        nodeExternals({
          allowlist: ['webpack/hot/poll?100'],
        }),
      ],
      module: {
        rules: [
          {
            test: /.tsx?$/,
            use: 'ts-loader',
            exclude: /node_modules/,
          },
        ],
      },
      mode: 'development',
      resolve: {
        extensions: ['.tsx', '.ts', '.js'],
      },
      plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new RunScriptWebpackPlugin({ name: 'server.js' }),
      ],
      output: {
        path: path.join(__dirname, 'dist'),
        filename: 'server.js',
      },
    };
    ```
    
    main.ts
    
    ```jsx
    declare const module: any;
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      await app.listen(3000);
    
      if (module.hot) {
        module.hot.accept();
        module.hot.dispose(() => app.close());
      }
    }
    bootstrap();
    ```
    
    ```json
    "start:dev": "webpack --config webpack.config.js --watch"
    ```