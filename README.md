# task_manager

## 概要

`task_manager` は、PythonのFlaskフレームワークとSQLAlchemyをバックエンドに、JavaScript（SortableJS、Frappe Gantt）、HTML、CSS（Tailwind CSS）をフロントエンドに使用したシンプルなタスク管理アプリケーションです。タスクの追加、編集、削除（論理削除）に加え、Ganttチャートによる視覚化、完了済み・削除済みタスクの管理、ドラッグ＆ドロップによるタスクの並び替えなどの機能を提供します。

## 主な機能

*   **タスクの登録**: タスク名、詳細、期限日時、予定開始/終了日時、メインタスクではないフラグを設定して新規タスクを追加できます。
*   **タスクの表示**: アクティブなタスク（「todo」または「doing」ステータス）を一覧表示します。
*   **タスクのソート**: タスクの表示順序（ドラッグ＆ドロップで設定）または期限日（昇順）でソートできます。ソート順を切り替えるボタンも用意されています。
*   **タスクの編集**: 既存のタスクの詳細情報を更新できます。
*   **タスクの状態変更**:
    *   **開始 (`doing`)**: タスクを開始し、`actual_start_date` を記録します。
    *   **一時停止 (`todo`)**: 実行中のタスクを一時停止し、「todo」状態に戻します。
    *   **完了 (`completed`)**: タスクを完了し、`actual_end_date` を記録します。
    *   **削除 (`deleted`)**: タスクを論理削除し、`delete_reason` を記録できます。
*   **タスクの復元**: 完了済みまたは削除済みのタスクを「todo」状態に復元できます。
*   **ドラッグ＆ドロップによる並び替え**: アクティブなタスクの表示順序を直感的に変更できます（「表示順でソート」時のみ）。
*   **Ganttチャート表示**: スケジュールが設定されたタスクをGanttチャートで視覚化します。
    *   チャートのビューモード（ズームレベル）をボタンまたは`Ctrl + ホイールスクロール`で変更できます。
    *   当日を示す赤い線が表示されます。
    *   タスクの状態（実行中、サブタスクなど）に応じて色分けされます。
*   **完了済み/削除済みタスクの表示**: 各カテゴリー専用のリストで、該当するタスクを表示します。
*   **タスク詳細ビュー**: どのタスクもクリックで詳細情報を確認できます。詳細ビューから直接編集ポップアップを開くことも可能です。

## セットアップ方法

### 前提条件

*   Python 3.x (推奨バージョン 3.8 以上)
*   pip (Pythonパッケージインストーラ)
*   Git (リポジトリのクローンに使用)

### インストール手順

以下のステップに従って、アプリケーションをローカル環境にセットアップします。

1.  **リポジトリをクローンします。**
    ```bash
    git clone https://github.com/KinoshitaMisaki/task_manager.git
    cd task_manager
    ```

2.  **仮想環境を作成し、アクティベートします（強く推奨）。**
    仮想環境を使用することで、プロジェクトの依存関係がシステムのPython環境と衝突するのを防ぎます。

    ```bash
    python -m venv venv
    ```

    作成した仮想環境をアクティベートします。
    *   **Windows:**
        ```cmd
        .\venv\Scripts\activate
        ```
    *   **macOS / Linux:**
        ```bash
        source venv/bin/activate
        ```
    
    (ターミナルの行頭に `(venv)` が表示されたら、アクティベート成功です。)

3.  **必要なPythonパッケージをインストールします。**
    仮想環境がアクティベートされていることを確認し、`requirements.txt` に記載されているすべてのライブラリをインストールします。

    ```bash
    pip install -r requirements.txt
    ```

### アプリケーションの実行

#### データベースの初期化と初回起動時の注意点

本アプリケーションはSQLiteデータベース（`tasks.db`）を使用しています。初めてアプリケーションを起動する際や、既存のデータベースを完全にリセットしたい場合は、データベースの初期化が必要です。

**最も確実なデータベースの初期化方法として、`python app.py` コマンドを使用することを推奨します。**

1.  **古いデータベースファイルとキャッシュの削除（初期化時のみ）:**
    アプリケーションを起動する前に、もし `tasks.db` ファイルや `__pycache__` ディレクトリ（Pythonの実行時キャッシュ）が残っている場合は、クリーンな状態から始めるために削除してください。

    ```bash
    # プロジェクトのルートディレクトリで実行してください
    # Linux / macOS の場合
    rm -f tasks.db
    rm -rf __pycache__

    # Windows の場合
    del tasks.db
    rmdir /s /q __pycache__
    ```

2.  **データベースの初期化を含む起動:**
    `app.py` ファイルには、アプリケーションが直接実行された場合にのみ動作する特別なPythonのブロック (`if __name__ == '__main__':`) が含まれています。このブロック内でデータベースのテーブルを作成する `init_database()` 関数が呼び出されるため、この方法で起動することでデータベースが確実に初期化されます。

    ```bash
    # プロジェクトのルートディレクトリで実行してください
    python app.py
    ```
    ターミナルに `Initializing database...` と `Database initialized.` のメッセージが表示されたら成功です。

    **【重要】なぜ `flask run` では初期化されないのか？**
    `flask run` コマンドはFlask CLIを介してアプリケーションを起動します。この方法では `app.py` がPythonインタープリタによって「直接実行」されるのではなく、「モジュールとしてインポート」される形でロードされます。
    Pythonの仕様により、モジュールがインポートされる際には `if __name__ == '__main__':` ブロック内のコードは実行されません。その結果、`flask run` で起動した場合、データベースの初期化コードがスキップされ、`tasks` テーブルが作成されないままアプリケーションが動作しようとし、「`(sqlite3.OperationalError) no such table: tasks`」のようなエラーが発生する可能性があります。

#### 通常の開発時の起動方法

一度データベースが初期化され、`tasks.db` ファイル内にテーブルが正しく作成されていれば、以降は以下の方法で開発サーバーを起動できます。

1.  **Flaskアプリケーションを起動します。**
    ```bash
    python app.py
    ```
    この方法は、開発時にも利用でき、デバッグモード（`debug=True`）が有効になります。

    もし `FLASK_APP` 環境変数を設定していれば、Flask CLIを使って起動することも可能です。
    ```bash
    # 環境変数の設定 (一度設定すればOK)
    # Linux/macOS: export FLASK_APP=app.py
    # Windows: set FLASK_APP=app.py

    # その後、起動
    flask run
    ```
    ただし、`flask run` はデータベースの初期化コード（`if __name__ == '__main__':` 内のコード）を実行しない点にご注意ください。

2.  **ウェブブラウザでアクセスします。**
    アプリケーションが起動したら、通常以下のURLでアクセスできます。

    ```
    http://127.0.0.1:5000/
    ```

## 使用技術

*   **バックエンド**: Python 3, Flask, Flask-SQLAlchemy, SQLite
*   **フロントエンド**: HTML5, CSS3 (Tailwind CSS), JavaScript (SortableJS, Frappe Gantt)