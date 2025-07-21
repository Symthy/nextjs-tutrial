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

### Postgres

/app/lib/data.ts

```ts
import postgres from "postgres";

const sql = postgres(process.env.POSTGRES_URL!, { ssl: "require" });
```

### リクエストウォーターフォール

直列に実行される = リクエストウォーターフォール

```ts
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

並列実行したい場合は Promise.all を使う

```ts
const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

const data = await Promise.all([
  invoiceCountPromise,
  customerCountPromise,
  invoiceStatusPromise,
]);
```

### ストリーミング

ルートを小さな「チャンク」に分割し、準備が整うとサーバーからクライアントに段階的にストリーミングできるデータ転送技術

ストリーミングにより、低速なデータリクエストによってページ全体がブロックされることを防ぐことができる

Next.js でストリーミングを実装する方法は 2 つ：

- ページレベルでは、loading.tsx ファイルを作成 (これにより<Suspense>が作成される) 。

```ts
export default function Loading() {
  return <div>Loading...</div>;
}
```

- コンポーネントレベルでは、<Suspense>を使用してより細かく制御できる。

loading.tsx がある階層すべてに適用されるため、それを避ける場合はルートグループを使用する

#### Suspense の境界をどこにおくか？

いくつかの要素で決まる

- ページのストリーミング中にユーザーにどのようにページを体験してもらいたいか。
- 優先したいコンテンツか。
- コンポーネントがデータ取得に依存している場合。

### 部分的な事前レンダリング（Implementing Partial Prerendering）

部分的な事前レンダリングは、Next.js の新しい機能で、ページの一部を事前にレンダリングし、他の部分はクライアントサイドで動的にレンダリングすることができる。
これにより、ページの一部を事前にレンダリングして高速に表示し、他の部分は必要なときにクライアントサイドでレンダリングすることができます。
部分的な事前レンダリングを実装するには、以下の手順を実行します。

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {
    ppr: "incremental",
  },
};

export default nextConfig;
```

以下を追加

/app/dashboard/layout.tsx

```ts
export const experimental_ppr = true;
```

### 検索とページネーション

Next の機能

- useSearchParams：URL の検索パラメータを取得
- usePathname：現在の URL のパス名を取得
- useRouter：クライアントコンポーネント内のルート間のナビゲーションをプログラムで可能にする

コンポーネント

- app/ui/search.tsx ： 検索
- app/ui/invoices/pagination.tsx : ページ間移動
- app/ui/invoices/table.tsx : 表示

#### デバウンス（use-debounce）

- キーを押すたびに URL が更新され、キーを押すたびにデータベースにクエリが送信されるのを防ぐ（リソース節約）
- デバウンスとは、関数の実行頻度を制限するプログラミング手法
- ユーザーが入力を停止してから特定の時間 (↓ の場合は 300 ミリ秒) が経過した後にのみコードを実行

```tsx
import { useDebouncedCallback } from "use-debounce";

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set("query", term);
  } else {
    params.delete("query");
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```

### Server Action

- サーバーコンポーネント内でサーバーアクションを呼び出す利点は、プログレッシブエンハンスメント（漸進的拡張）
  - つまり、クライアントに JavaScript がまだ読み込まれていなくてもフォームが機能する
- サーバアクションは Next.js のキャッシュと深く統合される
