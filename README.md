# japan-stay-tax

> Japan accommodation tax (宿泊税) calculator for Tokyo, Osaka, Kyoto, Niseko & 25+ municipalities

Calculate Japan's accommodation tax for any municipality. Zero dependencies. Fully typed. Bilingual (EN/JA).

日本の宿泊税を自治体ごとに計算できるTypeScript/JavaScriptライブラリ。依存関係ゼロ、完全型付き、日英バイリンガル対応。

---

Japan's accommodation tax varies by municipality — Tokyo, Osaka, Kyoto, and 20+ other cities each have different rates, tiers, and rules. This package provides a single, accurate, programmatic source for all of them.

Built for PMS developers, channel managers, booking engines, and accommodation operators who need to calculate 宿泊税 correctly across Japan.

日本の宿泊税は自治体ごとに税率・段階・ルールが異なります。東京、大阪、京都をはじめ20以上の自治体に対応。PMS開発者、チャネルマネージャー、予約エンジン、宿泊事業者向けに、正確な宿泊税データを一元的に提供します。

## Install / インストール

```bash
npm install japan-stay-tax
```

## Quick Start / クイックスタート

```typescript
import { calculateTax, searchAreas } from "japan-stay-tax";

// Calculate tax for a guest in Tokyo paying 15,000 yen/person/night
// 東京で1人1泊15,000円の宿泊税を計算
const result = calculateTax({
  areaId: "tokyo",
  ratePerNight: 15000,  // per person, per night
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

## Important: Per-Person Pricing / 重要：1人あたりの料金

**Japan's accommodation tax is levied per person, not per room.** The `ratePerNight` parameter must be the price **per person per night**.

**日本の宿泊税は1人あたりに課税されます（1室あたりではありません）。** `ratePerNight` には **1人1泊あたりの料金** を指定してください。

### Handling per-room prices from OTAs / OTAの1室あたり料金を扱う場合

Most foreign OTAs (Airbnb, Booking.com, Expedia, etc.) provide per-room prices, not per-person. You must divide by the number of guests before calling `calculateTax`.

Airbnb・Booking.com・Expediaなどの海外OTAは1室あたりの料金を提供します。`calculateTax` を呼ぶ前に宿泊人数で割ってください。

```typescript
// Example: Booking.com sends ¥30,000/room/night for 2 guests
// 例：Booking.comから1室1泊30,000円、2名の場合
const roomRate = 30000;
const guests = 2;
const perPerson = Math.floor(roomRate / guests); // ¥15,000

const tax = calculateTax({
  areaId: "tokyo",
  ratePerNight: perPerson,  // ¥15,000 per person
});
// → { total: 200 } (per person — multiply by guests for total)

const totalTax = tax.total * guests;
// → ¥400 total for the room
```

> **Note:** This per-person calculation is how all Japanese municipalities define the tax base. A ¥30,000 room with 1 guest is taxed differently than the same room with 2 guests.
>
> **注意：** 1人あたりの計算方式は全自治体共通の課税基準です。同じ30,000円の部屋でも、1名利用と2名利用では宿泊税が異なります。

### What to exclude from the rate / 料金から除外すべきもの

The rate must exclude the following (this is defined by municipal ordinance):

以下は料金から除外してください（各自治体の条例で定められた課税標準）：

- Meals (食事代)
- Consumption tax (消費税)
- Service charges (サービス料)

This matches the Japanese legal definition: 素泊まり料金 (room charge only, tax-exclusive).

## API

### `calculateTax(options): TaxResult`

Calculate the accommodation tax for a stay. / 宿泊税を計算します。

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `areaId` | `string` | Yes | Area ID (e.g. `"tokyo"`, `"kyoto"`, `"fukuoka_city"`) |
| `ratePerNight` | `number` | Yes | **Per person per night** in JPY (1人1泊あたり), excluding meals/tax/service |
| `date` | `string` | No | ISO date string (e.g. `"2026-04-01"`). Defaults to today. Used to select applicable rules for areas with date-dependent rates. |

Returns a `TaxResult`:

```typescript
{
  total: number;          // Tax per person in JPY / 1人あたりの税額（円）
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

## Covered Municipalities / 対応自治体一覧

Currently covers **25+ municipalities** across Japan:

| Area ID | Name / 名称 | Tax Type / 課税方式 |
|---------|-------------|-------------------|
| `tokyo` | Tokyo / 東京都 | Fixed tiers / 定額段階制 |
| `osaka` | Osaka / 大阪府 | Fixed tiers / 定額段階制 |
| `kyoto` | Kyoto / 京都市 | Fixed 5 tiers / 定額5段階制 |
| `kanazawa` | Kanazawa / 金沢市 | Fixed tiers / 定額段階制 |
| `kutchan` | Kutchan (Niseko) / 倶知安町 | Percentage / 定率制 |
| `fukuoka_city` | Fukuoka City / 福岡市 | Dual tax / 二重課税（県＋市） |
| `fukuoka_other` | Fukuoka Pref. (other) / 福岡県（その他） | Fixed / 定額制 |
| `kitakyushu` | Kitakyushu / 北九州市 | Dual tax / 二重課税（県＋市） |
| `nagasaki` | Nagasaki / 長崎市 | Fixed tiers / 定額段階制 |
| `niseko` | Niseko Town / ニセコ町 | Fixed tiers / 定額段階制 |
| `sendai` | Sendai / 仙台市 | Dual tax / 二重課税（県＋市） |
| `miyagi_other` | Miyagi Pref. (outside Sendai) / 宮城県（仙台市以外） | Fixed / 定額制 |
| `sapporo` | Sapporo / 札幌市 | Dual tax (Apr 2026~) / 二重課税（2026年4月〜） |
| `hakodate` | Hakodate / 函館市 | Dual tax (Apr 2026~) / 二重課税（2026年4月〜） |
| `otaru` | Otaru / 小樽市 | Dual tax (Apr 2026~) / 二重課税（2026年4月〜） |
| `furano` | Furano / 富良野市 | Dual tax (Apr 2026~) / 二重課税（2026年4月〜） |
| `hokkaido_other` | Hokkaido (other) / 北海道（その他） | Fixed tiers (Apr 2026~) / 定額段階制（2026年4月〜） |
| `hiroshima` | Hiroshima Pref. / 広島県 | Fixed (Apr 2026~) / 定額制（2026年4月〜） |
| `tokoname` | Tokoname / 常滑市 | Fixed / 定額制 |
| `atami` | Atami / 熱海市 | Fixed / 定額制 |
| `takayama` | Takayama / 高山市 | Fixed tiers / 定額段階制 |
| `gero` | Gero / 下呂市 | Fixed tiers / 定額段階制 |
| `matsue` | Matsue / 松江市 | Fixed tiers / 定額段階制 |
| `hirosaki` | Hirosaki / 弘前市 | Fixed / 定額制 |
| `toba` | Toba / 鳥羽市 | Fixed (Apr 2026~) / 定額制（2026年4月〜） |
| `gifu` | Gifu / 岐阜市 | Fixed (Apr 2026~) / 定額制（2026年4月〜） |
| `yugawara` | Yugawara / 湯河原町 | Fixed tiers (Apr 2026~) / 定額段階制（2026年4月〜） |

Run `getAreaIds()` for the programmatic list. / `getAreaIds()` で全エリアIDを取得できます。

## Data Sources / データソース

All tax rates are sourced from official municipal government websites. Each `TaxArea` includes a `source` field linking to the authoritative page. See [src/data.ts](./src/data.ts) for all sources.

全税率データは各自治体の公式サイトを出典としています。各 `TaxArea` の `source` フィールドに公式ページへのリンクがあります。詳細は [src/data.ts](./src/data.ts) をご覧ください。

Data is current as of **March 2026**. Contributions to keep rates up-to-date are welcome.

データは **2026年3月時点** の情報です。最新情報への更新PRを歓迎します。

## Contributing / コントリビューション

Found an outdated rate or a missing municipality? PRs welcome!

税率の変更や未対応の自治体を見つけた場合、PRをお待ちしています。

### Adding a new area / 新しいエリアの追加手順

1. Add the entry to `src/data.ts` / `src/data.ts` にエントリを追加
2. Add tests in `tests/calculator.test.ts` / `tests/calculator.test.ts` にテストを追加
3. Run `npm test` to verify / `npm test` で検証
4. Submit a PR with a link to the official municipal source / 公式出典リンク付きでPRを提出

## Use Cases / ユースケース

- **PMS (Property Management Systems)** — auto-calculate tax at checkout / チェックアウト時の自動計算
- **Channel managers** — sync correct tax amounts to OTAs / OTAへの正確な税額連携
- **Booking engines** — show tax breakdown during reservation / 予約時の税額内訳表示
- **Accounting software** — validate collected accommodation tax / 徴収済み宿泊税の検証
- **Travel apps** — display estimated tax for trip planning / 旅行計画時の税額目安表示

## License / ライセンス

MIT
