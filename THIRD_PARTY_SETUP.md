# 第三者向けセットアップ手順書

このフォルダは、Streamlit で動かすダッシュボードです。  
この手順書は、相手が次のものを自分で用意する前提です。

- 自分の GitHub リポジトリ
- 自分の Supabase project
- 自分の Streamlit Community Cloud

## 1. 受け取るもの

相手には `hurokenchi-dashboard` フォルダを渡してください。  
GitHub の招待は不要です。相手が自分の GitHub に新しい repo を作ってアップロードします。

## 2. 相手が先に作るもの

- GitHub アカウント
- GitHub リポジトリ `hurokenchi-dashboard`
- Supabase project 1 個
- Streamlit Community Cloud アカウント

`Supabase` は相手が自分で作ります。  
`Streamlit` へのデプロイも相手が自分のアカウントで行います。

## 3. GitHub にアップロード

相手は `hurokenchi-dashboard` フォルダを自分の GitHub repo にアップロードします。

### 方法 A: GitHub の画面からアップロード

1. GitHub で新しい repo `hurokenchi-dashboard` を作る
2. `uploading an existing file` を選ぶ
3. `hurokenchi-dashboard` フォルダの中身をアップロードする

### 方法 B: Git を使ってアップロード

```powershell
cd C:\path\to\hurokenchi-dashboard
git init
git add .
git commit -m "Initial import"
git branch -M main
git remote add origin https://github.com/<your-account>/hurokenchi-dashboard.git
git push -u origin main
```

`<your-account>` は相手の GitHub ユーザー名です。

## 4. Supabase project を作成

相手は自分の Supabase で project を 1 つ作ります。  
`hurokenchi` と `hurokenchi-dashboard` は同じ project を使います。

作成後、次を控えます。

- `Project URL`
- `anon public key`

そのあと SQL Editor で次を実行します。

```sql
CREATE TABLE sensor_state (
    id INTEGER PRIMARY KEY,
    mode TEXT DEFAULT 'location',
    status INTEGER DEFAULT 0,
    is_drowning BOOLEAN DEFAULT FALSE,
    connected BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO sensor_state (id, mode, status, is_drowning, connected)
VALUES (1, 'location', 0, FALSE, FALSE)
ON CONFLICT (id) DO NOTHING;
```

## 5. Streamlit Community Cloud にデプロイ

1. 相手が Streamlit Community Cloud にログインする
2. GitHub を連携する
3. `New app` を押す
4. 自分の GitHub repo `hurokenchi-dashboard` を選ぶ
5. Branch は `main`
6. Main file path は `app.py`
7. Deploy を押す

デプロイが完了すると、Streamlit から `https://<custom-subdomain>.streamlit.app` の URL が発行されます。  
これが本番用の URL です。

## 6. Streamlit Secrets を設定

デプロイ後、App settings から Secrets を開いて次を設定します。

```toml
SUPABASE_URL = "https://your-project.supabase.co"
SUPABASE_KEY = "your-supabase-anon-key"
```

保存後、アプリを再起動します。

## 7. 誰が見られるか

- app を public にすると、その `*.streamlit.app` の URL を知っている人はブラウザで閲覧できます
- app を private にすると、招待した相手だけが閲覧できます

まずは public にして URL を共有する運用がいちばん簡単です。

## 8. ローカルで確認したい場合

```powershell
cd C:\path\to\hurokenchi-dashboard
conda create -n huro-dashboard python=3.10
conda activate huro-dashboard
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
Copy-Item .streamlit\secrets.toml.example .streamlit\secrets.toml
python -m streamlit run app.py
```

`.streamlit\secrets.toml` には、相手の Supabase の値を入れてください。  
この `pip` も `conda` 環境の中で動くので、他の Python 環境とは分離されます。

## 9. URL

URL には 2 種類あります。

### 1. 本番用の公開 URL

```text
https://<custom-subdomain>.streamlit.app
```

相手が Streamlit Community Cloud にデプロイしたあとに発行される URL です。  
public にすれば、この URL で誰でも見られます。

### 2. ローカル確認用 URL

```text
http://localhost:8501
```

`localhost` はその PC の中だけで見える URL なので、他人には共有できません。

## 10. このアプリの役割

- Supabase の `sensor_state` テーブルを読む
- `hurokenchi/web_app` が書き込んだ状態を表示する

## 11. よくある詰まりどころ

- 画面が接続待ちのまま: `web_app` が同じ Supabase に書いているか確認する
- Streamlit でエラー: Secrets の URL / key を確認する
- GitHub repo が Streamlit に出ない: GitHub 連携と repo 所有者を確認する
