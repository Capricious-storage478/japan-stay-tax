# japan-stay-tax

Calculate Japan's accommodation tax (宿泊税) for any municipality. Zero dependencies. Fully typed.

日本の宿泊税を自治体ごとに計算できるTypeScriptライブラリです。依存関係ゼロ、完全型付き。

---

Japan's accommodation tax varies by municipality — Tokyo, Osaka, Kyoto, and 20+ other cities each have different rates, tiers, and rules. This package provides a single, accurate, programmatic source for all of them.

日本の宿泊税は自治体ごとに税率・段階・ルールが異なります。東京、大阪、京都をはじめ20以上の自治体に対応。公式情報源をもとに正確なデータを一元的に提供します。

## Install / インストール

```bash
npm install japan-stay-tax
```

## Usage / 使い方

```typescript
import { calculateTax, searchAreas } from "japan-stay-tax";

// Calculate tax for a guest in Tokyo paying 15,000 yen/night
// 東京で1泊15,000円の宿泊税を計算
const result = calculateTax({
  areaId: "tokyo",
  ratePerNight: 15000,
});
// → { total: 200, currency: "JPY", breakdown: [...] }

// Kyoto's 5-tier system (revised March 2026)
// 京都市の5段階制（2026年3月改定）
calculateTax({ areaId: "kyoto", ratePerNight: 75000 });
// → { total: 4000, ... }

// Fukuoka City has dual tax (prefecture + city)
// 福岡市は県税＋市税の二重課税
const fuk = calculateTax({ areaId: "fukuoka_city", ratePerNight: 25000 });
// → { total: 500, breakdown: [
//     { authority: { en: "Fukuoka Prefecture", ja: "福岡県" }, amount: 50 },
//     { authority: { en: "Fukuoka City", ja: "福岡市" }, amount: 450 }
//   ]}

// Kutchan (Niseko) uses percentage-based tax
// 倶知安町（ニセコ）は定率制
calculateTax({ areaId: "kutchan", ratePerNight: 30000, date: "2026-04-15" });
// → { total: 900 } (3% from April 2026 / 2026年4月から3%)

// Search areas by name (English or Japanese)
// エリア名で検索（英語・日本語対応）
searchAreas("hokkaido");
// → [{ id: "kutchan", ... }, { id: "niseko", ... }, { id: "sapporo", ... }, ...]

searchAreas("京都");
// → [{ id: "kyoto", ... }]
```

## API

### `calculateTax(options): TaxResult`

Calculate the accommodation tax for a stay. / 宿泊税を計算します。

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `areaId` | `string` | Yes | Area ID (e.g. `"tokyo"`, `"kyoto"`, `"fukuoka_city"`) |
| `ratePerNight` | `number` | Yes | Room rate per person per night in JPY (1人1泊あたりの宿泊料金), excluding meals, consumption tax, and service charges |
| `date` | `string` | No | ISO date string (e.g. `"2026-04-01"`). Defaults to today. Used to select applicable rules for areas with date-dependent rates. |

Returns a `TaxResult`:

```typescript
{
  total: number;          // Total tax in JPY / 税額合計（円）
  currency: "JPY";
  areaId: string;
  areaName: { en: string; ja: string };
  breakdown: Array<{      // Breakdown by authority / 課税主体ごとの内訳
    authority: { en: string; ja: string };
    amount: number;
    type: "fixed" | "percentage";
  }>;
  ratePerNight: number;
  date: string;
}
```

### `getArea(areaId): TaxArea | undefined`

Get detailed tax area info including rate tiers, exemptions, and source URLs.

エリアの詳細情報（税率段階、免税対象、公式URL等）を取得します。

### `getAreaIds(): string[]`

List all available area IDs. / 対応エリアIDの一覧を返します。

### `searchAreas(query): TaxArea[]`

Search areas by name (English or Japanese) or prefecture.

エリア名（英語・日本語）や都道府県名で検索します。

### `getAllAreas(): TaxArea[]`

Get all tax areas with full details. / 全エリアの詳細情報を返します。

## Available Areas / 対応エリア

Currently covers **25+ municipalities** including:

| Area ID | Name / 名称 | Tax Type / 課税方式 |
|---------|-------------|-------------------|
| `tokyo` | Tokyo / 東京都 | Fixed tiers / 定額段階制 |
| `osaka` | Osaka / 大阪府 | Fixed tiers / 定額段階制 |
| `kyoto` | Kyoto / 京都市 | Fixed 5 tiers / 定額5段階制 |
| `kanazawa` | Kanazawa / 金沢市 | Fixed tiers / 定額段階制 |
| `kutchan` | Kutchan (Niseko) / 倶知安町 | Percentage / 定率制 |
| `fukuoka_city` | Fukuoka City / 福岡市 | Dual tax / 二重課税（県＋市） |
| `kitakyushu` | Kitakyushu / 北九州市 | Dual tax / 二重課税（県＋市） |
| `nagasaki` | Nagasaki / 長崎市 | Fixed tiers / 定額段階制 |
| `niseko` | Niseko Town / ニセコ町 | Fixed tiers / 定額段階制 |
| `sendai` | Sendai / 仙台市 | Dual tax / 二重課税（県＋市） |
| `sapporo` | Sapporo / 札幌市 | Dual tax (from Apr 2026) / 二重課税（2026年4月〜） |
| `hakodate` | Hakodate / 函館市 | Dual tax (from Apr 2026) / 二重課税（2026年4月〜） |
| `hiroshima` | Hiroshima / 広島県 | Fixed (from Apr 2026) / 定額制（2026年4月〜） |
| ... | [and more / その他](./src/data.ts) | |

Run `getAreaIds()` for the complete list. / `getAreaIds()` で全エリアIDを確認できます。

## Room Rate Definition / 宿泊料金の定義

The `ratePerNight` parameter should be the **per-person-per-night room charge** excluding:

- Meals (食事代)
- Consumption tax (消費税)
- Service charges (サービス料)

This matches the standard definition used by all Japanese municipalities (素泊まり料金).

`ratePerNight` は **1人1泊あたりの素泊まり料金**（税抜・食事代・サービス料を除く）を指定してください。各自治体の宿泊税条例で定められた課税標準と同じ定義です。

## Data Sources / データソース

All tax rates are sourced from official municipal government websites. Each `TaxArea` includes a `source` field linking to the authoritative page. See [src/data.ts](./src/data.ts) for all sources.

全税率データは各自治体の公式サイトを出典としています。各 `TaxArea` オブジェクトの `source` フィールドに公式ページへのリンクがあります。詳細は [src/data.ts](./src/data.ts) をご覧ください。

Data is current as of **March 2026**. Contributions to keep rates up-to-date are welcome.

データは **2026年3月時点** の情報です。最新情報への更新PRを歓迎します。

## Contributing / コントリビューション

Found an outdated rate or a missing municipality? PRs welcome!

税率の変更や未対応の自治体を見つけた場合、PRをお待ちしています。

To add a new area / 新しいエリアの追加手順:

1. Add the entry to `src/data.ts` / `src/data.ts` にエントリを追加
2. Add tests in `tests/calculator.test.ts` / `tests/calculator.test.ts` にテストを追加
3. Run `npm test` to verify / `npm test` で検証
4. Submit a PR with a link to the official municipal source / 公式出典リンク付きでPRを提出

## License / ライセンス

MIT
