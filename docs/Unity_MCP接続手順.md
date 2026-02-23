# Unity MCP (Model Context Protocol) 接続手順

本プロジェクトでは、AIによる直接的なUnityデータ・シーン構造の把握と編集を目的として、**MCP for Unity** を導入しています。安定した動作のため、HTTPLocal接続ではなく、プロセスを直接繋ぐ **STDIO方式** を採用します。

## 1. 動作要件・事前準備
*   **Node.js**: 本環境（STDIO方式）では**不要**です。
*   **uv**: AI側がUnityへの接続元プロセスを立ち上げるために、高速なPythonツールチェーンである「[`uv`](https://github.com/astral-sh/uv)」を使用します。

## 2. 接続の手順（Unity側）
Unityエディタを立ち上げるたびに、MCPサーバーを「待機状態（STDIO）」にする必要があります。

1.  Unityエディタの上部メニューより、**[Window] > [MCP for Unity]** （または **[Tools] > [MCP]**等）を選択し、コントロールウィンドウを開く。
2.  ウィンドウ内の **Transport** の項目を `HTTPLocal` から **`STDIO`** に変更する。
3.  **「Start Server」** ボタンをクリックする。
    *   状態が `Running` になっていれば準備完了です。
    *   Consoleに赤文字のエラーが出ていないか確認してください。

## 3. 接続の手順（Antigravity側）
Antigravity（AI）の設定ファイル `mcp_config.json` において、`unityMCP` の項目が以下のように `uvx` 経由で直接起動する設定になっているか確認します。
（通常は自動で読み込まれます）

```json
{
  "mcpServers": {
    "unityMCP": {
      "command": "uvx",
      "args": [
        "--prerelease",
        "explicit",
        "--from",
        "mcpforunityserver>=0.0.0a0",
        "mcp-for-unity",
        "--project-scoped-tools"
      ]
    }
  }
}
```

## 4. 動作確認
Antigravityに対して、「今のシーンの名前を教えて」や「適当なゲームオブジェクトを生成して」と指示し、問題なく情報が返ってきたり、Unity上のヒエラルキーにオブジェクトが生成されれば接続成功です。

## （参考）なぜHTTPLocalを使わないのか？
HTTPLocal方式では、AI側からSSEクライアント（Node.jsベースの `npx` コマンド等）を使って通信URLへ接続しに行く手順が必要になります。これにはNode.jsの別途インストールが必要なうえ、プロセス管理が二度手間になる場合があります。
STDIO方式であれば、通信経路がプロセス間で閉じるため、HTTP特有のセキュリティ（Session IDの要求等）に弾かれることなく、安全かつスピーディーに通信を確立できます。
