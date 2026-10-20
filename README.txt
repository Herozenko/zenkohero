スタッフ管理（GitHub ＋ Cloudflare・チーム共有版）セットアップ手順
=================================================================
全員が、スマホ・PCのどこからでも同じ最新データを見て・更新できます（他の端末へ十数秒で自動反映）。
無料枠で動きます。アクセスキー：zenko1234（変更方法は最後）

■ 構成（1つのCloudflare Workerで完結）
  public/   画面（index.html など）
  src/      データ保存・同期のサーバー処理（Durable Objects）＋初期データ（名簿など）
  wrangler.toml / package.json   Cloudflare用の設定

■ 手順（所要 約15分・すべてブラウザだけでOK）
 【1】GitHub に非公開リポジトリを作る  ★必ず Private
   ・名簿の個人情報とアクセスキーがコードに入っているため、公開(Public)にしないでください
   1. GitHub → New repository → 名前（例：staff-app）→ Private → Create
   2. 「uploading an existing file」→ このフォルダの中身（public / src / wrangler.toml / package.json / .gitignore）を
      フォルダごとドラッグ → Commit changes
      （src/index.mjs は約1.4MBですがアップロードできます）
 【2】Cloudflare でリポジトリをつなぐ
   1. https://dash.cloudflare.com に無料登録 → 左メニュー「Workers & Pages」→ Create → 「Import a repository（Connect to Git）」
   2. GitHub と連携 → 先ほどのリポジトリを選択
   3. 設定はそのまま（Build command 空／Deploy command は npx wrangler deploy）→ Save and Deploy
   4. 完了後、https://staff-app.（あなたのサブドメイン）.workers.dev のURLが出ます
 【3】みんなに共有
   ・URLの末尾に ?k=zenko1234 を付けて送ると、キー入力なしで開けます
     例）https://staff-app.xxxx.workers.dev/?k=zenko1234
   ・スマホはホーム画面に追加しておくと便利です

■ 旧Netlify版のデータを引き継ぐ
   ・同梱の初期データ＝旧版の元データ（Excel＋キャスナビ名簿）が、最初のアクセス時に自動で入ります
   ・旧版で画面から入力した分は、旧アプリをPCで開く →「⇅ CSV」→「バックアップを書き出す」で保存し、
     新アプリをPCで開く →「⇅ CSV」→ 復元 で取り込んでください（1回だけ）

■ 更新するとき
   ・画面を直す：GitHub の public/index.html を差し替え → 自動で再公開されます（保存データはそのまま）
   ・保存データは Cloudflare 側に残ります。Worker を作り直さないでください（データは Worker に紐づきます）

■ アクセスキーを変える
   Cloudflare → Workers & Pages → staff-app → Settings → Variables and Secrets →
   TOKEN という名前で新しいキーを追加 → Deploy。
   （未設定の間は zenko1234 です。10回続けて間違えると10分ロックされます）

■ スタッフの削除と「復活させない」仕組み
  ・名簿の各カードの 🗑、スタッフ詳細の「🗑 削除」、編集画面の「🗑 このスタッフを削除」から削除できます（確認画面あり）
  ・名簿の「☑ 選んで削除」：カードをタップして複数チェック →「表示中をすべて選ぶ」も可 → 下の「🗑 選んだ○名を削除」で一括削除
  ・名簿のカードに常時「🗑 削除」ボタンがあります（1名ずつ）
  ・「🗑 退職者を整理」ボタン：退職扱いの人を一覧からチェックして、まとめて削除できます
  ・削除したIDは「削除済みリスト」としてサーバーに保存され、全端末に共有されます
  ・元データ（キャスナビCSV・簡易CSV）を何度読み込み直しても、削除済みのIDは取り込まれません（プレビューに「削除済みのため除外 ○名」と表示）
  ・戻したいとき：「⇅ CSV」→「削除済みスタッフ」の「戻す」→ もう一度CSVを取り込むと名簿に戻ります
    （または、CSV取り込み画面の「削除済みの人も復活させて取り込む」にチェック）
  ・削除すると、その人のシフトの割り当ても消えます

■ 注意
   ・同じ項目を同時に別の人が編集すると、あとから保存した内容が残ります
   ・CSVの取り込み／出力はPCのみ。キャスナビCSVの全データ（口座等）はサーバーに送らず、取り込んだPCのブラウザ内だけに保存されます
   ・念のため、定期的にPCから「バックアップを書き出す」で控えを保存してください
   ・無料枠にはリクエスト数の上限があります（画面を開いている間だけ15秒ごとに確認します）
   ・ブラウザ内だけで動かす簡易モード：public/config.js を  window.APP_CONFIG = { api: "local" };  にすると、サーバーなし（同期なし）で動きます
