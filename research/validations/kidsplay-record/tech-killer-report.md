# Tech Killer Report: KidsPlay Record

## 判定: 🟡 WOUNDED

致命傷には至らないが、ソロ開発者が「動画アプリ」を選ぶこと自体が戦略的に間違っている。4週間MVPは動画機能を大幅に削らない限り不可能。コストが利益を食い潰す構造的問題がある。

---

## 致命的ではないが深刻な問題

1. **動画ストレージコストがフリーミアム月額480円を構造的に破壊する** -- 1ユーザーが月10本の動画（各2分、圧縮後約50MB）をアップロードすると月500MBの増加。有料ユーザー100人で月50GBの増加。Supabase Pro（$25/月）の100GB枠は2ヶ月で枯渇する。月額480円（約$3.2）の課金では、1ユーザーあたりのストレージコストすら回収できない可能性が高い。
2. **expo-videoの成熟度が本番運用に不安** -- expo-avは非推奨となりexpo-videoへの移行が必要だが、Androidでの黒画面バグ、複数VideoViewの制限、ProGuard破壊など既知の問題が多数報告されている。
3. **COPPA/Kids Category対応がソロ開発者の工数を大幅に食う** -- 子供のスポーツ動画を扱う以上、子供の顔・個人情報が含まれるデータを保存・管理する。App StoreのKids Categoryに該当する可能性があり、その場合サードパーティSDK制限、保護者同意フロー、プライバシーポリシー整備が必須。

---

## ソロ開発者フィージビリティ

| 項目 | 評価 |
|---|---|
| MVP開発期間（推定） | **8-12週間**（4週間は非現実的。動画圧縮・アップロード・再生だけで2-3週間。成績記録UI+マルチスポーツ対応で2-3週間。認証・課金・ストア審査対応で2-3週間） |
| 必要技術スタック | Expo (React Native) + Supabase Auth/DB + 外部ストレージ（Cloudflare R2 or AWS S3）+ 動画トランスコーディングサービス + react-native-compressor + Stripe/RevenueCat |
| 月間インフラコスト（0ユーザー時） | **$25-30**（Supabase Pro $25 + ドメイン等） |
| 月間インフラコスト（1000ユーザー時） | **$150-400**（ストレージ500GB-1TB: $10-21 + エグレス: $20-50 + トランスコーディング: $30-100 + Supabase Pro: $25 + CDN: $10-20。有料転換率10%で月収$320。赤字または薄利。） |
| 週間保守時間（推定） | **5-10時間**（動画再生バグ対応、Expo SDKアップデート、ストア審査対応、ユーザーサポート） |
| ソロ開発者判定 | **困難** |

---

## 発見した技術リスク

### リスク1: 動画ストレージコストがユニットエコノミクスを破壊する
- **深刻度**: 重大
- **詳細**: Supabase Storage無料枠は1GB、Proプランでも100GB。1本の試合動画は圧縮後でも30-100MB。無料ユーザーが動画30本（上限）をアップすると最大3GB/ユーザー。100人の無料ユーザーだけで300GB。Supabase Proの100GB枠を超えた分は$0.021/GB/月だが、エグレス（動画再生時のデータ転送）が$0.09/GBと高額。1000ユーザーが月に各10回動画を再生するだけで、エグレスコストが月$45-90に達する。
- **なぜソロ開発者にとって問題か**: 月額480円のサブスクでは1ユーザーあたり月$3.2の収入。ストレージ+エグレスで1ユーザーあたり$1-3/月のコストが発生し、利益がほぼ残らない。Cloudflare R2に移行すればエグレス無料だが、Supabaseとの統合が複雑になり開発工数が増える。

### リスク2: expo-videoの未成熟さとプラットフォーム固有バグ
- **深刻度**: 重大
- **詳細**: expo-avはExpo SDK 52で非推奨となり、expo-videoへの移行が推奨されている。しかしexpo-videoには以下の既知問題がある:（1）Androidで動画が黒画面になりオーディオだけ再生されるバグ、（2）expo-avとexpo-videoの共存不可、（3）Android ProGuardによる破壊、（4）複数VideoViewのAndroidでの制限（ExoPlayer由来）、（5）Web版でのアプリ全体クラッシュ（v1.2.6）。スポーツ動画アプリでは動画再生が核心機能であり、ここが不安定だとアプリの存在意義が消える。
- **なぜソロ開発者にとって問題か**: 動画再生バグの調査・修正はネイティブコード（Java/Kotlin, Swift/ObjC）の知識が必要。Expo の抽象化レイヤーの下に潜る必要があり、ソロ開発者の工数を大幅に消費する。ユーザーから「動画が再生できない」というサポート問い合わせが頻発する可能性が高い。

### リスク3: 動画トランスコーディングの必要性
- **深刻度**: 重大
- **詳細**: Supabase Storageは動画のトランスコーディング（フォーマット変換・解像度変換）を一切サポートしていない。iPhoneはHEVC (H.265)で撮影するが、Android端末では再生できないケースがある。また、4K動画をそのままアップロードするとストレージコストが爆発する。クライアント側でreact-native-compressorを使って圧縮できるが、（1）圧縮処理中のUX劣化（数十秒〜数分の待ち時間）、（2）端末のバッテリー消費、（3）バックグラウンド処理の信頼性問題がある。サーバーサイドトランスコーディングを導入すると、Google Cloud Transcoder APIで$0.015-0.03/分のコストが追加で発生する。
- **なぜソロ開発者にとって問題か**: クライアントサイド圧縮とサーバーサイドトランスコーディングのどちらを選んでも、実装の複雑度が大幅に上がる。サーバーサイドの場合はCloud Functions等の追加インフラが必要で、Supabase単体では完結しない。

### リスク4: COPPA / App Store Kids Category / 個人情報保護への対応
- **深刻度**: 重大
- **詳細**: このアプリは子供の顔が映った動画・写真を保存する。App StoreのKids Categoryに分類される場合、（1）サードパーティSDK（Analytics, 広告等）の使用制限、（2）保護者のverifiable parental consent取得フロー、（3）アプリ内課金への保護者ゲート、（4）外部リンクへの保護者ゲート、（5）プライバシーポリシーの詳細開示が必須。2025年6月施行のCOPPA改正ルールでは、データ保持ポリシーの開示や第三者へのデータ共有に対する個別同意が追加要件となった。日本のAPPI（個人情報保護法）にも子供の個人情報に関する改正議論が進行中。
- **なぜソロ開発者にとって問題か**: 法的要件の調査・実装・維持だけで数週間の工数がかかる。弁護士への相談費用も発生する。要件を満たさないとApp Storeのリジェクトやアプリ削除のリスクがある。FTCによるCOPPA違反の罰金は最大$50,120/件。

### リスク5: オフライン対応の必要性
- **深刻度**: 中程度
- **詳細**: スポーツグラウンドは屋外にあり、モバイル通信の電波が弱い・圏外の場所が多い。試合中に動画を撮って即座にアップロードしたいユースケースでは、（1）オフラインキューイング、（2）再接続時の自動アップロード、（3）アップロード途中でのリジューム対応が必要。大容量動画のバックグラウンドアップロードはiOS/Androidで挙動が異なり、特にiOSではバックグラウンドタスクの実行時間に制限がある（expo-file-systemのバックグラウンドアップロードが極端に遅いという報告あり）。
- **なぜソロ開発者にとって問題か**: オフライン対応はモバイルアプリ開発で最も複雑な領域の一つ。Supabaseのオフライン対応は限定的で、動画ファイルのオフラインキューイングは自前実装が必要。

### リスク6: Expo SDKのアップデート頻度と破壊的変更
- **深刻度**: 中程度
- **詳細**: Expo SDKはReact Nativeの2リリースごとに新バージョンを出しており、年2-3回のメジャーアップデートがある。SDK 52でexpo-avが非推奨、SDK 53でNew Architecture強制移行、SDK 55でBlur APIの破壊的変更など、各バージョンで破壊的変更が発生している。動画関連ライブラリは特に変更が激しい領域。
- **なぜソロ開発者にとって問題か**: 年2-3回のメジャーアップデート対応にそれぞれ1-2週間かかる。対応しないとApp Storeの最新iOS要件を満たせなくなるリスクがある。

### リスク7: マルチスポーツ対応のデータモデル複雑性
- **深刻度**: 中程度
- **詳細**: 野球の打率・防御率、サッカーのゴール数・アシスト数、バスケのシュート率・リバウンド数など、スポーツごとに記録すべき成績項目が全く異なる。汎用的なデータモデル（JSON型のフレキシブルスキーマ）にすると入力UIが複雑になり、固定スキーマにするとスポーツ追加のたびにDB変更が必要。
- **なぜソロ開発者にとって問題か**: MVPでは1-2スポーツに絞るべきだが、「マルチスポーツ対応」を謳うと全スポーツの成績定義を網羅する圧力がかかる。スポーツごとのUI/バリデーション/集計ロジックの実装工数が積み上がる。

### リスク8: 強力な既存競合の存在
- **深刻度**: 中程度
- **詳細**: GameChanger（DICK'S Sporting Goods傘下）は150以上の統計項目、ライブストリーミング、スプレーチャートを提供。TeamSnapは2400万ユーザーを持ち、リアルタイム更新・写真共有・統計追跡を備える。MOJO Sportsはハイライト自動生成・ライブストリーミングを提供。これらは豊富な資金と開発チームで運営されている。
- **なぜソロ開発者にとって問題か**: 機能面で勝てない。差別化ポイントが「成績と動画の紐付け」だが、GameChangerは既にそれに近い機能を持っている。

---

## 推奨技術スタック（作るとしたら）

動画をアプリの核心機能にする場合:

| レイヤー | 推奨 | 理由 |
|---|---|---|
| フロントエンド | Expo (React Native) | 妥当な選択だが、expo-videoの不安定さに備えてreact-native-videoも検討 |
| 認証 | Supabase Auth | 十分 |
| データベース | Supabase PostgreSQL | 十分 |
| 動画ストレージ | **Cloudflare R2**（Supabase Storageではなく） | エグレス無料。S3互換API。ストレージ$0.015/GB/月でSupabaseより安い |
| 動画圧縮 | react-native-compressor | クライアントサイドで圧縮してからアップロード |
| 動画配信 | Cloudflare Stream or Mux | トランスコーディング+アダプティブビットレート配信。ただしコスト増 |
| 課金 | RevenueCat | App Store/Google Play課金を統合管理 |
| CDN | Cloudflare（R2と統合） | 動画配信高速化 |

ただし、このスタックはSupabase + 1サービスでは済まず、3-4サービスの統合が必要になる。ソロ開発者の管理負荷が大幅に増える。

---

## より簡単な代替アプローチ

### 代替案1: 動画を捨てて「成績記録特化アプリ」にする
- 動画はカメラロールへのディープリンク（参照）だけ保持し、アプリ内にはアップロードしない
- 成績データ（テキスト+数値）だけをSupabaseに保存
- ストレージコスト問題が消滅し、月額$25のSupabase Proで数千ユーザーを捌ける
- MVP4週間が現実的になる
- **これが最も合理的な選択**

### 代替案2: Google Photos + スプレッドシートの組み合わせ
- Google Photosのアルバム機能で試合ごとに動画を整理（無料15GB、Google One 100GBで月額250円）
- Google Sheetsで成績記録
- アプリ開発コストゼロ。ユーザーが既に持っているツールで解決可能
- 問題: UXが悪く、紐付けが手動。しかし「それで困っている人がどれだけいるか」は検証が必要

### 代替案3: Notionテンプレート + 動画リンク
- Notionのデータベース機能で選手・試合・成績を管理
- 動画はYouTube限定公開 or Google Photosのリンクを貼る
- 月額無料〜$10で実現可能
- 問題: モバイルUXがNotionに依存。保護者層にはハードルが高い

### 代替案4: 「みてね」+ メモ機能
- みてね（FamilyAlbum）は無制限の動画・写真アップロードが無料
- コメント機能で成績メモを残す
- 問題: 成績の構造化データ管理・集計ができない

---

## 結論

KidsPlay Recordは「作れなくはないが、ソロ開発者が作って維持するにはコストと複雑度が高すぎる」プロダクトである。

**核心的な問題は3つ:**

1. **動画ストレージのユニットエコノミクス崩壊**: 月額480円では動画保存・配信コストを回収できない。値上げすれば競合（GameChanger等の無料・低価格サービス）に負ける。コストを下げるためにCloudflare R2等に移行すると、Supabase単体の簡潔さが失われ、ソロ開発者の前提が揺らぐ。

2. **動画関連技術の複雑度**: 圧縮・トランスコーディング・マルチフォーマット対応・バックグラウンドアップロード・オフラインキューイング・ストリーミング再生。これらすべてを1人で実装・保守するのは非現実的。expo-videoの未成熟さがリスクを増幅する。

3. **COPPA/法的要件の重さ**: 子供の個人情報（顔が映った動画）を扱う以上、法的要件への対応は避けられない。これだけで数週間の工数と弁護士費用が必要。

**私の推奨**: 動画アップロード機能を完全に削除し、「成績記録 + カメラロール参照」アプリとして再定義すること。動画はデバイスのカメラロールに残し、アプリからはタイムスタンプとメタデータでリンクするだけにする。これなら4週間MVPは可能で、Supabase無料枠でも運用でき、ストレージコスト問題は消滅する。「動画と成績の紐付け」という価値は、動画をアプリ内に保存しなくても実現できる。

---

*調査日: 2026-03-28*

## 調査ソース

- [Supabase Pricing](https://supabase.com/pricing)
- [Supabase Storage Pricing Docs](https://supabase.com/docs/guides/storage/pricing)
- [Supabase Storage File Limits](https://supabase.com/docs/guides/storage/uploads/file-limits)
- [Supabase Storage Bandwidth & Egress](https://supabase.com/docs/guides/storage/serving/bandwidth)
- [Supabase Storage Scaling](https://supabase.com/docs/guides/storage/production/scaling)
- [Supabase Discussion: Video intensive platform](https://github.com/orgs/supabase/discussions/26958)
- [expo-video Documentation](https://docs.expo.dev/versions/latest/sdk/video/)
- [expo-video npm](https://www.npmjs.com/package/expo-video)
- [expo-video Android bugs](https://github.com/expo/expo/issues/35648)
- [expo-video MediaLibrary issue](https://github.com/expo/expo/issues/36218)
- [expo-video Web crash](https://github.com/expo/expo/issues/31565)
- [Debugging Expo Video Playback (Medium)](https://medium.com/code-sense/raw-debugging-expo-video-playback-technical-timeline-ab6eb8501af4)
- [expo-av deprecation / migration](https://github.com/Expensify/App/issues/64846)
- [react-native-compressor](https://www.npmjs.com/package/react-native-compressor)
- [iOS background upload slowness](https://github.com/expo/expo/issues/26754)
- [Expo SDK 55 changelog](https://expo.dev/changelog/sdk-55)
- [COPPA Compliance 2025 Guide](https://blog.promise.legal/startup-central/coppa-compliance-in-2025-a-practical-guide-for-tech-edtech-and-kids-apps/)
- [FTC COPPA Rule](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa)
- [Apple Kids Category Guidelines](https://developer.apple.com/kids/)
- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Japan APPI Children's Privacy](https://www.nishimura.com/en/news/20251201-117211)
- [Google Cloud Transcoder Pricing](https://cloud.google.com/transcoder/pricing)
- [GameChanger Youth Sports App](https://gc.com/youth-sports-app)
- [TeamSnap](https://apps.apple.com/us/app/teamsnap/id393048976)
- [Mastering Media Uploads in React Native 2026](https://dev.to/fasthedeveloper/mastering-media-uploads-in-react-native-images-videos-smart-compression-2026-guide-5g2i)
