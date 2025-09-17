# やってみる
```
npm create vite@latest moz-todo-react -- --template react
```

**出力**(viteをインストールしていなかったので実行時にインストールするかどうか聞かれた)
```
Need to install the following packages:
create-vite@7.1.1
Ok to proceed? (y) y


> npx
> create-vite yuchord --template react

│
◇  Scaffolding project in C:\haru\src\yuchord...
│
└  Done. Now run:

  cd yuchord
  npm install
  npm run dev
```

出力にある通りにコマンドを実行するとlocalhostに開発サーバーが起動するので
ブラウザでアクセスするとテンプレートのビルド結果(実行結果)が見れる

**ビルド**
```
npm run build
```
distフォルダに出力される