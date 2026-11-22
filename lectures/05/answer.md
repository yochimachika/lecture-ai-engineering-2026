# 第5回 解答

## 演習

- `--list-tools` で見えたツール名と、第3回（`--mcp` なし）との違い：
--mcp なしでは、Python関数が search_glossary などの名前で直接ツールとして公開された。一方、--mcp ありでは、glossary_search_glossary のようにMCPサーバー名を付加した名前で表示された。また、get_lecture_summary をMCPサーバーに追加した後は、glossary_get_lecture_summary も認識された。
- `--mcp --trace` で見えたツール呼び出し（ツール名と引数）と、`requests` の回数：
「AIE2026の講座の宿題にある『3レベル』とは何ですか？」という質問に対し、glossary_search_glossary({'query': '3レベル'}) が呼び出された。requests=2、tool_calls=1 であった。--mcp なしでも同様の質問に回答できたが、呼び出されたツール名は search_glossary であった。
- MCP サーバーに公開したツールと、その説明文：
app/mcp_server.py に server.tool()(tools.get_lecture_summary) を追加し、glossary_get_lecture_summary を公開した。説明文は「指定した回（1始まりの整数）の講義の要点を返す。」であり、講義内容に関する質問で利用することが示されている。追加後のテストでは11件すべてが成功した。
- fetch サーバーを足したときのツール呼び出し（ツール名と引数）：
app/mcp_servers.json に外部のfetchサーバーを追加したところ、fetch_fetch が認識され、利用可能なツールが合計3個になった。公式サイト https://modelcontextprotocol.io/ の取得を試み、HTTPアクセスが実行されることを確認した。ただし、初回はモデル側の503エラー、再実行時はrobots.txtの取得に関する接続エラーが発生し、最終的な日本語要約は完了しなかった。

## 宿題

- 取り組んだレベル：基礎
- やったこと：
MCPサーバーのstdio方式とStreamable HTTP方式を比較した。まず、uv run python -m app.mcp_server --http でHTTPサーバーを起動し、app/mcp_servers.json の glossary の設定を http://127.0.0.1:8000/mcp に変更した。さらに、app/tools.py の get_lecture_summary のdocstringに、講義内容について質問された場合は推測せずツールで確認する旨を追記した。
- 結果：
stdio方式とHTTP方式のどちらでもMCPツールが認識され、glossary_get_lecture_summary({'lecture': 2}) により第2回の講義内容を取得できた。stdio方式ではクライアントがサーバープロセスを起動するのに対し、HTTP方式では別プロセスとしてサーバーを起動し、ポート8000を使用する違いがあった。また、HTTPサーバーを再起動すると、変更したdocstringが --list-tools の表示に反映された。
- 気づき・疑問：
MCPを利用すると、ツールの実装とAIエージェントを分離でき、接続方式を変更しても同じ機能を利用できることがわかった。また、docstringがAIモデルに提示されるツール説明文となるため、ツールの機能だけでなく、使用場面を明確に記述することが重要だと感じた。一方、外部fetchツールでは接続エラーが発生したため、外部サービスを利用する場合のエラー処理や再試行方法について、さらに理解を深めたい。