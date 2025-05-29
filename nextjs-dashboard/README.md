## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

### command

pnpm run dev
Starts the development server.

pnpm run build
Builds the app for production.

pnpm start
Runs the built app in production mode.

### folder structure

- /app : アプリケーションのすべてのルート、コンポーネント、ロジックを置く
  - /lib : 再利用可能なユーティリティ関数やデータ取得関数など、アプリケーションで使用する関数を置く
  - /ui : カード、テーブル、フォームなど、アプリケーションの UI コンポーネントを置く
- /public : 画像など、アプリケーションのすべての静的アセットを置く

#### files

/app/lib/placeholder-data.ts

ユーザインターフェースを構築する際、プレースホルダデータを用意しておけば便利

データベースや API がまだ利用できない場合は以下を利用する

- プレースホルダー データを JSON 形式または JavaScript オブジェクトとして使用
- [mockAPI](https://mockapi.io/) のようなサードパーティのサービスを使用

### CSS

ルートレイアウト： /app/layout.tsx

グローバルスタイル（/app/ui/global.css）はここにインポート

#### clsx

https://github.com/lukeed/clsx

条件付きでクラスを適用するために を使用

```tsx
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
```

### Font

#### プライマリフォント

/app/ui/fonts.ts に定義

```ts
import { Inter } from "next/font/google";

export const inter = Inter({ subsets: ["latin"] });
```

/app/layout.tsx にインポート

```tsx
import "@/app/ui/global.css";
import { inter } from "@/app/ui/fonts";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

### Image

Next.js の `<Image>` コンポーネントを使用して、画像の最適化とレスポンシブデザインを実現

自動画像最適化機能が付属

- 画像の読み込み時にレイアウトシフトが自動的に発生するのを防ぎます。
- 大きな画像がビューポートの小さいデバイスに送信されないように、画像のサイズを変更します。
- デフォルトでは画像を遅延読み込みします（画像はビューポートに入ると読み込まれます）。
- 画像のフォーマットを自動的に選択し、WebP などの最新のフォーマットを使用して、画像のサイズを小さくします。

### Link

`<a href="_">` ではなく、Next.js の `<Link href="_">` を使用

ページ全体の更新を気にせず、ページ間の移動ができる

Next.js は、リンクされたルートのコードをバックグラウンドで自動的にプリフェッチし、ユーザーがリンクをクリックしたときにすばやく表示できるようにする

#### アクティブなリンク

usePathname() を使用

`use client` が必要（usePathname は React Hook なので）

clsx を使用して、リンクがアクティブなときにクラスを適用可能

```tsx
className={clsx(
    '～',
    {
    'bg-sky-100 text-blue-600': pathname === link.href,
    },
)}
```
