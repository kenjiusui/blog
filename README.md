# blog

## リンクカード

記事内のリンクをOGP付きのカードとして表示できます。

```
{{< link-card "https://example.com/path" >}}
```

- ビルド時にリンク先ページを取得し、`og:title` / `og:description` / `og:image` / `og:site_name` を自動抽出してカードを生成します
- `og:title` が無いページは `<title>` タグにフォールバックします
- サイト名が無ければURLのホスト名を表示します
- 実装は [layouts/_shortcodes/link-card.html](layouts/_shortcodes/link-card.html)、[layouts/_partials/extract-og.html](layouts/_partials/extract-og.html)、スタイルは [assets/css/extended/link-card.css](assets/css/extended/link-card.css)

注意: リンク先の取得に失敗する（サイトがダウン、URLが誤りなど）とビルド自体がエラーで止まります。安定して使えるURLに対して使ってください。