// /api/webhook.js — Apparel flow (existing) + Illustration + Canvas Art + LINE Stamps
// Runtime: Vercel Node Functions (ESM)

import crypto from "crypto";

/* ===========================
 *  Env / Constants
 * =========================== */

// 公開許可（誰でも）
const ALLOW_IDS = [];

// 静的画像のベースURL
const PUBLIC_BASE =
  (process.env.PUBLIC_BASE_URL || "https://line-webhook-eta.vercel.app")
    .trim()
    .replace(/\/+$/, "");

// Square（production / sandbox）
const SQUARE_API_BASE =
  (process.env.SQUARE_ENV || "production") === "sandbox"
    ? "https://connect.squareupsandbox.com"
    : "https://connect.squareup.com";

/* ===========================
 *  Apparel: 共通（既存のまま）
 * =========================== */

// 起動ワード（そのまま）
const START_TRIGGERS = new Set(["#limited", "新商品&期間限定商品", "新商品＆期間限定商品", "限定"]);

// サイズ画像や色など（既存）
const TSHIRT_COLOR_HERO = "/hansode_tshirts_color1.png";
const TSHIRT_SIDE_HERO  = "/print_front_back.png";
const TSHIRT_OPTION_HERO = "/option_name2.png";

// オプション（文字入れ）
const OPTION_KEY   = "text";
const OPTION_LABEL = "文字入れ（＋400円）";
const OPTION_PRICE = 400;

// プリント面（既存）
const SIDE_BY_PRODUCT = {
  tshirt_premium: [
    { v: "front", label: "前面" },
    { v: "back", label: "後面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ],
  lsl_premium: [
    { v: "front", label: "前面" },
    { v: "back", label: "後面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ],
  sweat: [
    { v: "front", label: "前面" },
    { v: "back", label: "後面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ],
  tote_s: [
    { v: "single", label: "片面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ],
  tote_m_wide: [
    { v: "single", label: "片面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ],
  tote_m_tall: [
    { v: "single", label: "片面" },
    { v: "both", label: "両面プリント：＋2000円", add: 2000 }
  ]
};

/* ===========================
 *  NEW: 商品カタログ
 *  （アパレルは価格だけ仕様に合わせて更新）
 * =========================== */
const CATALOG = {
  // 既存アパレル（価格はご指定の割引後に更新）
  tshirt_premium: {
    label: "プレミアムTシャツ",
    basePrice: 3300, // ¥4,400 → ¥3,300
    img: "/hansode_tshirts_main_1.png",
    sizeImg: "/hansodetshirts_size_1.png",
    sideImg: "/hansode_tshirts_print_front_back1.png",
    colors: ["white", "gray", "baby_pink"],
    sizes: ["kids120", "XS", "S", "M", "L", "XL", "XXL"]
  },
  lsl_premium: {
    label: "プレミアム長袖Tシャツ",
    basePrice: 3900, // ¥4,800 → ¥3,900
    img: "/long_tshirts_main_1.png",
    sizeImg: "/longtshirts_size_1.png",
    sideImg: "/long_tshirts_print_front_back.png",
    colors: ["white"],
    sizes: ["S", "M", "L", "XL", "XXL"]
  },
  tote_s: {
    label: "トートバッグS",
    basePrice: 3000,
    img: "/toteback_s_main_1.png",
    sideImg: "/toteback_s_print_front_back1.png",
    colors: ["natural"],
    sizes: []
  },
  tote_m_wide: {
    label: "トートバッグM（横長）",
    basePrice: 3200,
    img: "/toteback_m_main_1.png",
    sideImg: "/toteback_m_print_front_back_1.png",
    colors: ["natural"],
    sizes: []
  },
  tote_m_tall: {
    label: "トートバッグM（縦長）",
    basePrice: 3200,
    img: "/toteback_L_main_1.png",
    sideImg: "/toteback_L_print_front_back_1.png",
    colors: ["natural"],
    sizes: []
  },
  sweat: {
    label: "スウェット",
    basePrice: 5400,
    img: "/sweat_main_1.png",
    sizeImg: "/sweats_size_1.png",
    sideImg: "/sweat_print_front_back.png",
    colors: ["gray"],
    sizes: ["M", "L", "XL", "XXL"]
  },

  /* ---------- NEW: 映えワン / 映えニャン（イラスト） ---------- */
  illustration: {
    label: "映えワン/映えニャン（ペットのイラスト製作）",
    basePrice: 2000, // 1匹の基本価格
    img: "/illustration_main.png",
    // カスタム用フラグ
    _type: "illustration"
  },

  /* ---------- NEW: キャンバスアート ---------- */
  canvas_art: {
    label: "キャンバスアート",
    basePrice: 5400, // 1枚の基本価格
    img: "/canvas_main.png",
    _type: "canvas_art"
  },

  /* ---------- NEW: 愛犬LINEスタンプ ---------- */
  line_stamp: {
    label: "愛犬LINEスタンプ",
    basePrice: 0,
    img: "/linestamp_hero.png",
    _type: "line_stamp"
  }
};

/* ===========================
 *  Illustration: 価格表（頭数段階）
 * =========================== */
const ILL_PET_COUNT_TIERS = {
  1: 0,
  2: 1500,
  3: 2800,
  4: 4100,
  5: 5400
};
// 6匹以上は要相談（UI上は〜5、6は注意喚起）

const ILL_COMPOSE_CHOICES = [
  { v: "one_per_image", label: "1枚のイラストに1匹" },
  { v: "multi_per_image", label: "1枚のイラストに複数匹" }
];
const ILL_STYLE_CHOICES = [
  { v: "art", label: "アート系" },
  { v: "cute", label: "かわいい系" },
  { v: "emblem", label: "エンブレム系" }
];
const ILL_RATIO_CHOICES = [
  { v: "9_16", label: "縦長 9:16" },
  { v: "16_9", label: "横長 16:9" },
  { v: "1_1", label: "正方形 1:1" }
];

/* ===========================
 *  Canvas Art: 枚数段階 + オプション
 * =========================== */
// 1枚は base だけ、2枚〜は追加金
const CANVAS_ADDER_BY_QTY = {
  1: 0,
  2: 4800,
  3: 9300,
  4: 13800
};
const CANVAS_SOURCE_CHOICES = [
  { v: "photo", label: "お持ちの写真" },
  { v: "own_illust", label: "お持ちのイラスト" },
  { v: "new_illust", label: "新規制作イラスト（別途ご注文ください）" }
];
const CANVAS_OPTION_KEY = "canvas_edit";
const CANVAS_OPTION_LABEL = "画像の加工/文字入れ（＋400円）";

/* ===========================
 *  LINE Stamp: パック・頭数・掲載可否
 * =========================== */
const STAMP_PACKS = [
  { v: "p8",   label: "お試しパック 8個",  price: 4800, count: 8 },
  { v: "p16",  label: "たっぷりパック 16個", price: 8200, count: 16 },
  { v: "all",  label: "全部詰めパック",      price: 10800, count: 0 }, // 個数は表示のみ
  { v: "rep8", label: "リピート追加 8個",    price: 3800, count: 8 }
];
const STAMP_PET_SURCHARGE = [
  { n: 1, add: 0 },
  { n: 2, add: 500 },
  { n: 3, add: 1000 }
];
const STAMP_PUBLISH_CHOICES = [
  { v: "ok", label: "掲載OK" },
  { v: "ng", label: "掲載NG" }
];

/* ===========================
 *  Sessions
 * =========================== */
// userId -> {
//   step,
//   order:[{product, ...apparel fields..., custom: {...}}], // custom は illustration / canvas_art / line_stamp 用
//   pending:{...},
//   customer:{name, phone, postal, address1, address2}
// }
const sessions = new Map();

/* ===========================
 *  Handler
 * =========================== */
export default async function handler(req, res) {
  if (req.method === "GET") return res.status(200).json({ ok: true, ts: Date.now() });
  if (req.method !== "POST") return res.status(200).send("OK");

  // Verify
  const sig = req.headers["x-line-signature"];
  const raw = req.body ? JSON.stringify(req.body) : "";
  if (!raw || !sig) return res.status(200).send("OK");
  try {
    const hmac = crypto
      .createHmac("sha256", process.env.LINE_CHANNEL_SECRET || "")
      .update(raw)
      .digest("base64");
    if (hmac !== sig) return res.status(200).send("OK");
  } catch {
    return res.status(200).send("OK");
  }

  const events = req.body?.events ?? [];
  for (const ev of events) {
    const userId = ev.source?.userId;
    if (!userId) continue;

    try {
      // whoami
      if (ev.type === "message" && ev.message?.type === "text") {
        const rawTxt = ev.message.text || "";
        const norm = normalizeText(rawTxt);
        if (rawTxt.trim().toLowerCase() === "whoami") {
          await reply(ev.replyToken, [{ type: "text", text: `your userId:\n${userId}` }]);
          continue;
        }
        if (START_TRIGGERS.has(norm)) {
          const s0 = newSession();
          sessions.set(userId, s0);
          s0.step = "pick_product";
          await askPickProduct(ev.replyToken, "ありがとうございます。商品をお選びください。");
          continue;
        }
      }

      const s = sessions.get(userId) || newSession();

      // ===== Text =====
      if (ev.type === "message" && ev.message?.type === "text") {
        const txt = (ev.message.text || "").trim();

        // 入金連絡
        const allowVague = s.step === "bank";
        if (isPaidMessage(txt, { allowVague })) {
          await reply(ev.replyToken, [{ type: "text", text: "ありがとうございます。入金確認後、制作/手配を開始します。" }]);
          await push(userId, [{ type: "text", text: buildCustomerConfirmationText(s) }]);
          const admins = (process.env.ADMIN_USER_IDS || "").split(",").map(x=>x.trim()).filter(Boolean);
          for (const adminId of admins) {
            await push(adminId, [{ type: "text", text: buildAdminConfirmationText(userId, s) }]);
          }
          s.step = "completed";
          sessions.set(userId, s);
          continue;
        }

        // お客様情報入力
        if (s.step === "ask_customer_name") {
          s.customer.name = txt;
          s.step = "ask_customer_phone"; sessions.set(userId, s);
          await askCustomerPhone(ev.replyToken); continue;
        }
        if (s.step === "ask_customer_phone") {
          s.customer.phone = txt.replace(/[^\d]/g, "");
          s.step = "ask_customer_postal"; sessions.set(userId, s);
          await askCustomerPostal(ev.replyToken); continue;
        }
        if (s.step === "ask_customer_postal") {
          s.customer.postal = txt.replace(/[^\d]/g, "");
          s.step = "ask_customer_address1"; sessions.set(userId, s);
          await askCustomerAddress1(ev.replyToken); continue;
        }
        if (s.step === "ask_customer_address1") {
          s.customer.address1 = txt;
          s.step = "ask_customer_address2"; sessions.set(userId, s);
          await askCustomerAddress2(ev.replyToken); continue;
        }
        if (s.step === "ask_customer_address2") {
          s.customer.address2 = txt === "なし" ? "" : txt;
          s.step = "confirm_customer"; sessions.set(userId, s);

          await reply(ev.replyToken, [
            {
              type: "flex",
              altText: "ご注文控え",
              contents: confirmationBubble({
                title: "ご注文内容の確認",
                orderText: orderSummaryAll(s),
                customer: s.customer
              })
            },
            {
              type: "flex",
              altText: "確認",
              contents: buttonListBubble(
                CATALOG[s.order[0]?.product || "tshirt_premium"].img,
                "上記のお客様情報でよろしいですか？",
                [
                  ["OK（決済へ進む）", "cust_ok=yes"],
                  ["修正（最初から）", "cust_ok=no"]
                ]
              )
            }
          ]);
          continue;
        }
      }

      // ===== Image: サイレント受信 =====
      if (ev.type === "message" && ev.message?.type === "image") { continue; }

      // ===== Postback =====
      if (ev.type === "postback") {
        const data = ev.postback?.data || "";

        // 商品選択
        if (data.startsWith("product=")) {
          const key = data.split("=")[1];
          const p = CATALOG[key];
          if (!p) { await reply(ev.replyToken, [{ type: "text", text: "商品が見つかりませんでした。" }]); continue; }

          // 分岐：カスタム系 or 既存アパレル系
          if (p._type === "illustration") {
            s.step = "ill_petcount"; s.pending = { product: key, custom: {} }; sessions.set(userId, s);
            await askIllustrationPetCount(ev.replyToken);
            continue;
          }
          if (p._type === "canvas_art") {
            s.step = "canvas_qty"; s.pending = { product: key, custom: {} }; sessions.set(userId, s);
            await askCanvasQty(ev.replyToken);
            continue;
          }
          if (p._type === "line_stamp") {
            s.step = "stamp_pack"; s.pending = { product: key, custom: {} }; sessions.set(userId, s);
            await askStampPack(ev.replyToken);
            continue;
          }

          // 既存アパレル系：数量から
          s.pending = { product: key, quantity: 0, currentIndex: 0, items: [] };
          s.step = "ask_quantity"; sessions.set(userId, s);
          await reply(ev.replyToken, [
            { type: "text", text: `${p.label} ですね。` },
            {
              type: "flex",
              altText: "数量選択",
              contents: buttonListBubble(p.img, "ご購入点数を選択してください", [
                ["1点", "qty=1"],["2点", "qty=2"],["3点", "qty=3"],["4点", "qty=4"],["5点", "qty=5"]
              ])
            }
          ]);
          continue;
        }

        /* ---------- 既存アパレルの分岐（数量→各種選択） ---------- */
        if (data.startsWith("qty=") && s.pending && !CATALOG[s.pending.product]._type) {
          const q = Math.max(1, Math.min(5, parseInt(data.split("=")[1], 10) || 1));
          s.pending.quantity = q;
          s.pending.items = Array.from({ length: q }, () => newItemState());
          s.pending.currentIndex = 0;
          s.step = "pick_campaign_item"; sessions.set(userId, s);
          await reply(ev.replyToken, [
            { type: "text", text: q === 1 ? "ありがとうございます。次にテーマをお選びください。" : `ありがとうございます。まずは1点目のテーマをお選びください。` },
            { type: "flex", altText: "キャンペーン選択", contents: campaignSelector() }
          ]);
          continue;
        }

        if (data.startsWith("campaign=") && s.pending && !CATALOG[s.pending.product]._type) {
          const cid = data.split("=")[1];
          const camp = CAMPAIGNS[cid];
          ensurePending(s, s.pending?.product || s.order.at(-1)?.product || "tshirt_premium");
          if (!camp) {
            await reply(ev.replyToken, [
              { type: "text", text: "キャンペーンが見つかりませんでした。もう一度お選びください。" },
              { type: "flex", altText: "キャンペーン選択", contents: campaignSelector() }
            ]);
            continue;
          }
          const msg = setChoice(currentItem(s).campaign, cid, "テーマ", camp.label);
          s.step = "pick_variation_item"; sessions.set(userId, s);
          await reply(ev.replyToken, [
            ...(msg ? [{ type: "text", text: msg }] : []),
            { type: "flex", altText: "バリエーション選択", contents: variationSelectorForCampaign(cid) }
          ]);
          continue;
        }

        if (data.startsWith("variation=") && s.pending && !CATALOG[s.pending.product]._type) {
          const vid = data.split("=")[1];
          const v = VARIATIONS[vid];
          if (!v) {
            const cid = currentItem(s)?.campaign?.value;
            await reply(ev.replyToken, [
              { type: "text", text: "バリエーションが見つかりませんでした。もう一度お選びください。" },
              { type: "flex", altText: "バリエーション選択", contents: variationSelectorForCampaign(cid) }
            ]);
            continue;
          }
          const msg = setChoice(currentItem(s).variation, vid, "バリエーション", v.label);
          await gotoColorSizeSideOrOption(ev.replyToken, s, userId, msg); continue;
        }

        if (data.startsWith("color=") && s.pending && !CATALOG[s.pending.product]._type) {
          const color = data.split("=")[1];
          const msg = setChoice(currentItem(s).color, color, "カラー", colorLabel(color));
          await gotoColorSizeSideOrOption(ev.replyToken, s, userId, msg); continue;
        }

        if (data.startsWith("size=") && s.pending && !CATALOG[s.pending.product]._type) {
          const size = data.split("=")[1];
          const item = currentItem(s); const prod = s.pending.product; const p = CATALOG[prod];
          if (prod === "tshirt_premium" && size === "kids120" && item.color?.value !== "white") {
            s.step = "pick_color_item"; sessions.set(userId, s);
            await reply(ev.replyToken, [
              { type: "text", text: "子供用120は白のみ対応です。カラーを白に変更してください。" },
              { type: "flex", altText: "カラー選択", contents: buttonListBubble(p.img, "カラーを選択", p.colors.map(c=>[colorLabel(c), `color=${c}`])) }
            ]);
            continue;
          }
          const msg = setChoice(item.size, size, "サイズ", sizeLabel(size));
          await gotoColorSizeSideOrOption(ev.replyToken, s, userId, msg); continue;
        }

        if (data.startsWith("side=") && s.pending && !CATALOG[s.pending.product]._type) {
          const side = data.split("=")[1];
          const msg = setChoice(currentItem(s).side, side, "プリント面", sideLabel(s.pending.product, side));
          s.step = "pick_opts_item"; sessions.set(userId, s);
          await sendOptionsAndNextItem(ev.replyToken, s, msg); continue;
        }

        if (data.startsWith("optset=") && s.pending && !CATALOG[s.pending.product]._type) {
          const payload = data.split("=")[1]; // optset=text:on / off
          const [key, val] = payload.split(":");
          const item = currentItem(s);
          if (key === OPTION_KEY) {
            const had = item.opts.has(OPTION_KEY);
            let msg = null;
            if (val === "on" && !had) { msg = item.optionFirst ? `オプション「${OPTION_LABEL}」が選択されました。` : `オプションを「${OPTION_LABEL}」に変更しました。`; item.opts.add(OPTION_KEY); item.optionFirst = false; }
            else if (val === "off" && had) { msg = item.optionFirst ? `オプション「${OPTION_LABEL}」が選択されました。` : `オプション「${OPTION_LABEL}」を外しました。`; item.opts.delete(OPTION_KEY); item.optionFirst = false; }
            await finalizeCurrentItem(ev.replyToken, s, userId, msg); continue;
          }
        }

        if (data.startsWith("addmore=") && s.step === "ask_additional") {
          const ans = data.split("=")[1];
          if (ans === "yes") { s.step = "pick_product"; sessions.set(userId, s); await askPickProduct(ev.replyToken, "ありがとうございます。追加の商品をお選びください。"); continue; }
          s.step = "confirm_all"; sessions.set(userId, s);
          await reply(ev.replyToken, [
            { type: "text", text: orderSummaryAll(s) },
            {
              type: "flex",
              altText: "注文確認",
              contents: buttonListBubble(
                CATALOG[s.order[0]?.product || "tshirt_premium"].img,
                "上記の内容で注文しますか？",
                [["はい（お客様情報の入力へ）","confirm=yes"],["修正する（最初から）","confirm=no"]]
              )
            }
          ]);
          continue;
        }

        if (data.startsWith("confirm=") && s.step === "confirm_all") {
          const ans = data.split("=")[1];
          if (ans === "no") { resetSession(s); sessions.set(userId, s); s.step="pick_product"; await askPickProduct(ev.replyToken, "はじめからやり直します。商品をお選びください。"); continue; }
          s.customer = {}; s.step = "ask_customer_name"; sessions.set(userId, s); await askCustomerName(ev.replyToken); continue;
        }

        if (data === "cust_ok=yes" && s.step === "confirm_customer") {
          s.step = "pick_pay"; sessions.set(userId, s);
          await reply(ev.replyToken, [
            { type: "text", text: orderSummaryAll(s) },
            {
              type: "flex",
              altText: "決済方法の選択",
              contents: buttonListBubble(CATALOG[s.order[0]?.product || "tshirt_premium"].img, "決済方法を選択してください", [
                ["クレジットカード（Square）","pay=card"],
                ["口座振込","pay=bank"]
              ])
            }
          ]);
          continue;
        }
        if (data === "cust_ok=no" && s.step === "confirm_customer") {
          s.step = "ask_customer_name"; sessions.set(userId, s); await askCustomerName(ev.replyToken); continue;
        }

        if (data.startsWith("pay=") && s.step === "pick_pay") {
          const method = data.split("=")[1];
          if (method === "bank") {
            s.step = "bank"; sessions.set(userId, s);
            await reply(ev.replyToken, [{
              type: "text",
              text: [
                "【銀行振込のご案内】",
                "銀行名：多摩信用金庫 / 八王子中央支店",
                "口座種別：普通　口座番号：0286023",
                "口座名義：ミズモトカズオ",
                "※お振込後、このトークで「入金済み」とお知らせください。"
              ].join("\n")
            }]);
            continue;
          }
          if (method === "card") {
            const items = buildSquareLineItems(s);
            const link = await createSquarePaymentLink({ items, referenceId: userId });
            if (!link) {
              await reply(ev.replyToken, [{ type: "text", text: "決済リンクの作成に失敗しました。時間をおいて再度お試しください。" }]);
              continue;
            }
            s.step = "waiting_payment"; sessions.set(userId, s);
            await reply(ev.replyToken, [
              { type: "text", text: `こちらから決済してください：\n${link}` },
              { type: "text", text: "決済完了後にこのトークへ戻ってきてください。\n完了したら「入金済み」と送ってください。" }
            ]);
            continue;
          }
        }

        /* ===========================
         *  NEW: Illustration フロー
         * =========================== */
        if (data.startsWith("ill_pet=") && s.pending?.product === "illustration") {
          const n = parseInt(data.split("=")[1], 10);
          const clamp = Math.max(1, Math.min(6, n || 1));
          s.pending.custom.petCount = clamp;
          s.step = "ill_compose"; sessions.set(userId, s);
          await askIllustrationCompose(ev.replyToken, clamp);
          continue;
        }
        if (data.startsWith("ill_compose=") && s.pending?.product === "illustration") {
          const v = data.split("=")[1];
          s.pending.custom.compose = v;
          s.step = "ill_style"; sessions.set(userId, s);
          await askIllustrationStyle(ev.replyToken);
          continue;
        }
        if (data.startsWith("ill_style=") && s.pending?.product === "illustration") {
          const v = data.split("=")[1];
          s.pending.custom.style = v;
          s.step = "ill_option"; sessions.set(userId, s);
          await askIllustrationOption(ev.replyToken);
          continue;
        }
        if (data.startsWith("ill_opt=") && s.pending?.product === "illustration") {
          const on = data.split("=")[1] === "on";
          s.pending.custom.textAdd = !!on;
          s.step = "ill_ratio"; sessions.set(userId, s);
          await askIllustrationRatio(ev.replyToken);
          continue;
        }
        if (data.startsWith("ill_ratio=") && s.pending?.product === "illustration") {
          const v = data.split("=")[1];
          s.pending.custom.ratio = v;

          // 1件として確定
          const p = CATALOG["illustration"];
          const amount = calcIllustrationAmount(s.pending.custom);
          s.order.push({
            product: "illustration",
            campaign: null, variation: null, color: null, size: null, side: null,
            opts: new Set(), amount,
            custom: { ...s.pending.custom }
          });
          s.pending = null;

          // 追加購入有無
          s.step = "ask_additional"; sessions.set(userId, s);
          await reply(ev.replyToken, [{
            type: "flex",
            altText: "追加注文の有無",
            contents: buttonListBubble(p.img, "他にご注文はありますか？", [
              ["はい（商品を追加する）", "addmore=yes"],
              ["いいえ（確認へ進む）", "addmore=no"]
            ])
          }]);
          continue;
        }

        /* ===========================
         *  NEW: Canvas Art フロー
         * =========================== */
        if (data.startsWith("can_qty=") && s.pending?.product === "canvas_art") {
          const q = parseInt(data.split("=")[1], 10);
          const qty = [1,2,3,4].includes(q) ? q : 1;
          s.pending.custom.qty = qty;
          s.step = "can_source"; sessions.set(userId, s);
          await askCanvasSource(ev.replyToken);
          continue;
        }
        if (data.startsWith("can_src=") && s.pending?.product === "canvas_art") {
          const v = data.split("=")[1];
          s.pending.custom.source = v;
          s.step = "can_opt"; sessions.set(userId, s);
          await askCanvasOption(ev.replyToken);
          continue;
        }
        if (data.startsWith("can_opt=") && s.pending?.product === "canvas_art") {
          const on = data.split("=")[1] === "on";
          s.pending.custom.editAdd = !!on;

          // 1件確定
          const p = CATALOG["canvas_art"];
          const amount = calcCanvasAmount(s.pending.custom);
          s.order.push({
            product: "canvas_art",
            campaign: null, variation: null, color: null, size: null, side: null,
            opts: new Set(), amount,
            custom: { ...s.pending.custom }
          });
          s.pending = null;

          s.step = "ask_additional"; sessions.set(userId, s);
          await reply(ev.replyToken, [{
            type: "flex",
            altText: "追加注文の有無",
            contents: buttonListBubble(p.img, "他にご注文はありますか？", [
              ["はい（商品を追加する）", "addmore=yes"],
              ["いいえ（確認へ進む）", "addmore=no"]
            ])
          }]);
          continue;
        }

        /* ===========================
         *  NEW: LINE Stamp フロー
         * =========================== */
        if (data.startsWith("st_pack=") && s.pending?.product === "line_stamp") {
          const v = data.split("=")[1];
          const pack = STAMP_PACKS.find(x=>x.v===v);
          if (!pack) { await askStampPack(ev.replyToken); continue; }
          s.pending.custom.pack = v;
          s.pending.custom.price = pack.price;
          s.step = "st_pet"; sessions.set(userId, s);
          await askStampPetHeads(ev.replyToken);
          continue;
        }
        if (data.startsWith("st_pet=") && s.pending?.product === "line_stamp") {
          const n = parseInt(data.split("=")[1], 10);
          const heads = [1,2,3].includes(n) ? n : 1;
          s.pending.custom.petHeads = heads;
          s.step = "st_pub"; sessions.set(userId, s);
          await askStampPublish(ev.replyToken);
          continue;
        }
        if (data.startsWith("st_pub=") && s.pending?.product === "line_stamp") {
          const v = data.split("=")[1];
          s.pending.custom.publish = v;

          // 確定
          const amount = calcStampAmount(s.pending.custom);
          s.order.push({
            product: "line_stamp",
            campaign: null, variation: null, color: null, size: null, side: null,
            opts: new Set(), amount,
            custom: { ...s.pending.custom }
          });
          s.pending = null;

          s.step = "ask_additional"; sessions.set(userId, s);
          await reply(ev.replyToken, [{
            type: "flex",
            altText: "追加注文の有無",
            contents: buttonListBubble(CATALOG["line_stamp"].img, "他にご注文はありますか？", [
              ["はい（商品を追加する）", "addmore=yes"],
              ["いいえ（確認へ進む）", "addmore=no"]
            ])
          }]);
          continue;
        }
      }

    } catch (e) {
      console.error("flow error:", e?.stack || e);
      try {
        await reply(ev.replyToken, [{ type: "text", text: "エラーが発生しました。最初からやり直す場合は「限定」と送ってください。" }]);
      } catch {}
    }
  }

  return res.status(200).send("OK");
}

/* ===========================
 *  Session helpers
 * =========================== */
function newSession() { return { step: "idle", order: [], pending: null, customer: {} }; }
function resetSession(s) { s.step = "idle"; s.order = []; s.pending = null; s.customer = {}; }
function currentItem(s) { return s.pending?.items?.[s.pending.currentIndex]; }
function newFS() { return { value: null, isFirst: true }; }
function newItemState() {
  return {
    campaign: newFS(),
    variation: newFS(),
    color: newFS(),
    size: newFS(),
    side: newFS(),
    opts: new Set(),
    optionFirst: true,
    amount: 0
  };
}

/* ===========================
 *  Amount / Summary
 * =========================== */
// 既存アパレル
function calcAmountItem(p, item) {
  let amount = p.basePrice || 0;
  const sideVal = typeof item?.side === "string" ? item.side : item?.side?.value ?? null;
  if (sideVal === "both") amount += 2000;

  const opts = item?.opts;
  const hasText =
    opts instanceof Set ? opts.has(OPTION_KEY) :
    Array.isArray(opts) ? opts.includes(OPTION_KEY) :
    opts && typeof opts === "object" ? !!opts[OPTION_KEY] : false;

  if (hasText) amount += OPTION_PRICE;
  return amount;
}

// NEW: Illustration 合計
function calcIllustrationAmount(custom) {
  const base = CATALOG.illustration.basePrice;
  const add  = ILL_PET_COUNT_TIERS[custom.petCount] ?? 0; // 2〜5匹の加算
  const opt  = custom.textAdd ? 400 : 0;
  return base + add + opt;
}

// NEW: Canvas Art 合計
function calcCanvasAmount(custom) {
  const base = CATALOG.canvas_art.basePrice;
  const adder = CANVAS_ADDER_BY_QTY[custom.qty] ?? 0;
  const opt   = custom.editAdd ? 400 : 0;
  return base + adder + opt;
}

// NEW: LINE Stamp 合計
function calcStampAmount(custom) {
  const pack = STAMP_PACKS.find(x=>x.v===custom.pack);
  const base = pack ? pack.price : 0;
  const surcharge = STAMP_PET_SURCHARGE.find(x=>x.n===custom.petHeads)?.add ?? 0;
  return base + surcharge;
}

function totalAmountAll(s) {
  let total = 0;
  for (const it of s.order) {
    if (it.amount) { total += it.amount; continue; }
    const p = CATALOG[it.product];
    total += calcAmountItem(p, { side: it.side, opts: it.opts });
  }
  return total;
}

function orderSummaryAll(s) {
  const lines = ["【注文内容の確認】"];
  s.order.forEach((it, idx) => {
    const p = CATALOG[it.product];
    const appSub = it.amount || calcAmountItem(p, { side: it.side, opts: it.opts });

    // カスタム商品の表示
    let extra = "";
    if (it.product === "illustration") {
      const c = it.custom || {};
      const style = ILL_STYLE_CHOICES.find(x=>x.v===c.style)?.label ?? "";
      const comp  = ILL_COMPOSE_CHOICES.find(x=>x.v===c.compose)?.label ?? "";
      const ratio = ILL_RATIO_CHOICES.find(x=>x.v===c.ratio)?.label ?? "";
      extra = [ `頭数:${c.petCount}匹`, comp, style, ratio, c.textAdd ? "文字入れあり（＋400円）" : "" ].filter(Boolean).join(" / ");
    } else if (it.product === "canvas_art") {
      const c = it.custom || {};
      const src = CANVAS_SOURCE_CHOICES.find(x=>x.v===c.source)?.label ?? "";
      extra = [ `枚数:${c.qty}枚`, src, c.editAdd ? "画像の加工/文字入れ（＋400円）" : "" ].filter(Boolean).join(" / ");
    } else if (it.product === "line_stamp") {
      const c = it.custom || {};
      const pack = STAMP_PACKS.find(x=>x.v===c.pack)?.label ?? "";
      const pub = STAMP_PUBLISH_CHOICES.find(x=>x.v===c.publish)?.label ?? "";
      extra = [ pack, `頭数:${c.petHeads}匹`, pub ].filter(Boolean).join(" / ");
    } else {
      // 既存アパレル
      const sideStr = it.side === "both" ? "両面プリント（＋2,000円）" : sideLabel(it.product, it.side);
      const hasText = it.opts && it.opts instanceof Set && it.opts.has(OPTION_KEY);
      const optStr  = hasText ? `／${OPTION_LABEL}` : "";
      extra = [ colorLabel(it.color), sizeLabel(it.size), [sideStr,optStr].filter(Boolean).join("") ].filter(Boolean).join("／");
    }

    lines.push(
      [
        `\n#${idx + 1} ${p.label}`,
        extra ? extra : "",
        `→ ${appSub.toLocaleString()}円`
      ].filter(Boolean).join("\n")
    );
  });
  lines.push(`\n合計: ${totalAmountAll(s).toLocaleString()}円`);
  return lines.join("\n");
}

/* ===========================
 *  Square Line Items
 * =========================== */
function buildSquareLineItems(s) {
  const items = [];
  for (const it of s.order) {
    const p = CATALOG[it.product];

    if (it.product === "illustration") {
      const c = it.custom || {};
      const name = [p.label, `頭数:${c.petCount}匹`, ILL_COMPOSE_CHOICES.find(x=>x.v===c.compose)?.label, ILL_STYLE_CHOICES.find(x=>x.v===c.style)?.label, ILL_RATIO_CHOICES.find(x=>x.v===c.ratio)?.label, c.textAdd ? "文字入れ" : null].filter(Boolean).join(" / ");
      items.push({ name, amount: calcIllustrationAmount(c), quantity: 1 });
      continue;
    }

    if (it.product === "canvas_art") {
      const c = it.custom || {};
      const name = [p.label, `枚数:${c.qty}枚`, CANVAS_SOURCE_CHOICES.find(x=>x.v===c.source)?.label, c.editAdd ? "加工/文字入れ" : null].filter(Boolean).join(" / ");
      items.push({ name, amount: calcCanvasAmount(c), quantity: 1 });
      continue;
    }

    if (it.product === "line_stamp") {
      const c = it.custom || {};
      const pack = STAMP_PACKS.find(x=>x.v===c.pack)?.label ?? "";
      const surcharge = STAMP_PET_SURCHARGE.find(x=>x.n===c.petHeads)?.add ?? 0;
      const base = STAMP_PACKS.find(x=>x.v===c.pack)?.price ?? 0;

      items.push({ name: `${p.label} / ${pack}`, amount: base, quantity: 1 });
      if (surcharge) items.push({ name: `頭数加算（${c.petHeads}匹）`, amount: surcharge, quantity: 1 });
      continue;
    }

    // 既存アパレル
    let unit = p.basePrice || 0;
    if (it.side === "both") unit += 2000;
    if (it.opts && it.opts.has(OPTION_KEY)) unit += OPTION_PRICE;

    const optLabels = [];
    if (it.side === "both") optLabels.push("両面");
    if (it.opts && it.opts.has(OPTION_KEY)) optLabels.push("文字入れ");

    const name = [
      p.label,
      sizeLabel(it.size),
      colorLabel(it.color),
      optLabels.length ? `オプション:${optLabels.join("・")}` : null,
    ].filter(Boolean).join(" / ");

    items.push({ name, amount: unit, quantity: 1 });
  }
  return items;
}

/* ===========================
 *  UI: 共通
 * =========================== */
async function askPickProduct(replyToken, leadingMsg) {
  const msgs = [];
  if (leadingMsg) msgs.push({ type: "text", text: leadingMsg });
  msgs.push({ type: "flex", altText: "商品選択", contents: productCarousel() });
  msgs.push({ type: "text", text: "上記より、ご希望の商品をお選びください。" });
  await reply(replyToken, msgs);
}
function productCarousel() {
  const bubbles = Object.entries(CATALOG).map(([key, p]) =>
    cardBubble({
      heroUrl: absoluteUrl(p.img),
      title: `${p.label}  ${p.basePrice ? `¥${p.basePrice.toLocaleString()}` : ""}`,
      buttons: [["選ぶ", `product=${key}`]]
    })
  );
  return { type: "carousel", contents: bubbles };
}

/* ===========================
 *  UI: 既存（キャンペーン/バリエーションなど）
 * =========================== */
const CAMPAIGNS = {
  momiji: { id: "momiji", label: "紅葉に包まれるきみ", img: "/kouyounitutumarerukimi_matome.png" },
  aki_walk: { id: "aki_walk", label: "秋を散策するきみ", img: "/akiwosansakusurukimi_matome.png" }
};
const VARIATIONS = {
  normal:   { id: "normal",   label: "通常ver",    img: { momiji: "/kouyounitutumareru_kimi2.png", aki_walk: "/akiwosansakusuru_kimi2.png" } },
  halloween:{ id: "halloween",label: "ハロウィンver", img: { momiji: "/kouyounitutumareru_kimi3.png", aki_walk: "/akiwosansakusuru_kimi3.png" } }
};
function campaignSelector() {
  const bubbles = Object.values(CAMPAIGNS).map((c) =>
    cardBubble({ heroUrl: absoluteUrl(c.img), title: c.label, buttons: [["このテーマで進む", `campaign=${c.id}`]] })
  );
  return { type: "carousel", contents: bubbles };
}
function variationSelectorForCampaign(campaignId) {
  const bubbles = Object.values(VARIATIONS).map((v) => {
    const hero = v.img?.[campaignId] || Object.values(v.img || {})[0] || "";
    return cardBubble({ heroUrl: absoluteUrl(hero), title: v.label, buttons: [["このバリエーションで進む", `variation=${v.id}`]] });
  });
  return { type: "carousel", contents: bubbles };
}
async function gotoColorSizeSideOrOption(replyToken, s, userId, leadingMsg) {
  const prod = s.pending.product; const p = CATALOG[prod];
  const msgs = []; if (leadingMsg) msgs.push({ type: "text", text: leadingMsg });
  const item = currentItem(s);

  if (p.colors?.length && item.color.value == null) {
    s.step = "pick_color_item"; sessions.set(userId, s);
    const colorHero = prod === "tshirt_premium" ? TSHIRT_COLOR_HERO : p.img;
    msgs.push(
      { type: "text", text: "カラーを選択してください" },
      { type: "flex", altText: "カラー選択", contents: buttonListBubble(colorHero, "カラーを選択", p.colors.map(c=>[colorLabel(c), `color=${c}`])) }
    );
    await reply(replyToken, msgs); return;
  }
  if (p.sizes?.length && item.size.value == null) {
    s.step = "pick_size_item"; sessions.set(userId, s);
    const sizeTitle = prod === "tshirt_premium" ? "サイズを選択してください（※子供用120は白のみ）" : "サイズを選択してください";
    const sizeHero = p.sizeImg || p.img;
    msgs.push({ type: "text", text: sizeTitle }, { type: "flex", altText: "サイズ選択", contents: buttonListBubble(sizeHero, "サイズを選択", p.sizes.map(z=>[sizeLabel(z), `size=${z}`])) });
    await reply(replyToken, msgs); return;
  }
  const sides = SIDE_BY_PRODUCT[prod] || [];
  if (sides.length && item.side.value == null) {
    s.step = "pick_side_item"; sessions.set(userId, s);
    const sideHero = p.sideImg || (prod === "tshirt_premium" ? TSHIRT_SIDE_HERO : p.img);
    msgs.push(
      { type: "text", text: "プリント面を選択してください" },
      { type: "flex", altText: "プリント面選択", contents: buttonListBubble(sideHero, "プリント面を選択", sides.map(si=>[si.label, `side=${si.v}`])) }
    );
    await reply(replyToken, msgs); return;
  }
  s.step = "pick_opts_item"; sessions.set(userId, s);
  await sendOptionsAndNextItem(replyToken, s, leadingMsg);
}
async function sendOptionsAndNextItem(replyToken, s, leadingMsg) {
  const prod = s.pending.product; const p = CATALOG[prod];
  const msgs = []; if (leadingMsg) msgs.push({ type: "text", text: leadingMsg });
  const optionHero = TSHIRT_OPTION_HERO || p.img;
  const bubble = {
    type: "bubble",
    hero: { type: "image", url: absoluteUrl(optionHero), size: "full", aspectRatio: "20:13", aspectMode: "cover" },
    body: {
      type: "box", layout: "vertical", spacing: "md",
      contents: [
        { type: "text", text: "オプションを選択してください", weight: "bold", size: "lg" },
        { type: "text", text: OPTION_LABEL, size: "md" },
        { type: "button", style: "secondary", action: { type: "postback", label: "する",   data: `optset=${OPTION_KEY}:on` } },
        { type: "button", style: "secondary", action: { type: "postback", label: "しない", data: `optset=${OPTION_KEY}:off` } }
      ]
    }
  };
  msgs.push({ type: "flex", altText: "オプション選択", contents: bubble });
  await reply(replyToken, msgs);
}
async function finalizeCurrentItem(replyToken, s, userId, leadingMsg) {
  const prod = s.pending.product; const p = CATALOG[prod]; const item = currentItem(s);
  item.amount = calcAmountItem(p, item);

  if (s.pending.currentIndex + 1 < s.pending.quantity) {
    s.pending.currentIndex++; s.step = "pick_campaign_item"; sessions.set(userId, s);
    const msgs = []; if (leadingMsg) msgs.push({ type: "text", text: leadingMsg });
    msgs.push({ type: "text", text: `続いて ${s.pending.currentIndex + 1}点目のテーマをお選びください。` }, { type: "flex", altText: "キャンペーン選択", contents: campaignSelector() });
    await reply(replyToken, msgs); return;
  }

  s.order.push(
    ...s.pending.items.map((x) => ({
      product: prod,
      campaign: x.campaign.value,
      variation: x.variation.value,
      color: x.color.value,
      size: x.size.value,
      side: x.side.value,
      opts: new Set(x.opts)
    }))
  );
  s.pending = null;
  s.step = "ask_additional"; sessions.set(userId, s);
  await reply(replyToken, [{
    type: "flex",
    altText: "追加注文の有無",
    contents: buttonListBubble(CATALOG[prod]?.img || CATALOG["tshirt_premium"].img, "他にご注文はありますか？", [
      ["はい（商品を追加する）","addmore=yes"],["いいえ（確認へ進む）","addmore=no"]
    ])
  }]);
}

/* ===========================
 *  UI: Illustration
 * =========================== */
async function askIllustrationPetCount(replyToken) {
  const pairs = [
    ["1匹（＋¥0）", "ill_pet=1"],
    ["2匹（＋¥1,500）", "ill_pet=2"],
    ["3匹（＋¥2,800）", "ill_pet=3"],
    ["4匹（＋¥4,100）", "ill_pet=4"],
    ["5匹（＋¥5,400）", "ill_pet=5"],
    ["6匹以上（事前にご相談ください）", "ill_pet=6"]
  ];
  await reply(replyToken, [
    { type: "text", text: "イラストにしたいペットの頭数をお選びください。" },
    { type: "flex", altText: "頭数選択", contents: buttonListBubble(CATALOG.illustration.img, "ペット頭数を選択", pairs) }
  ]);
}
async function askIllustrationCompose(replyToken, n) {
  const note = n >= 6 ? "※6匹以上は制作可否・お見積りを事前にご相談ください。" : "";
  await reply(replyToken, [
    ...(note ? [{ type: "text", text: note }] : []),
    { type: "flex", altText: "構成選択", contents: buttonListBubble(CATALOG.illustration.img, "1枚内の構成を選択", ILL_COMPOSE_CHOICES.map(c=>[c.label, `ill_compose=${c.v}`])) }
  ]);
}
async function askIllustrationStyle(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "作風選択", contents: buttonListBubble(CATALOG.illustration.img, "希望の作風を選択", ILL_STYLE_CHOICES.map(c=>[c.label, `ill_style=${c.v}`])) }
  ]);
}
async function askIllustrationOption(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "文字入れ", contents: buttonListBubble(CATALOG.illustration.img, "文字入れを希望しますか？（＋¥400）", [["希望する","ill_opt=on"],["不要","ill_opt=off"]]) }
  ]);
}
async function askIllustrationRatio(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "アスペクト比", contents: buttonListBubble(CATALOG.illustration.img, "ご希望のアスペクト比を選択", ILL_RATIO_CHOICES.map(r=>[r.label, `ill_ratio=${r.v}`])) }
  ]);
}

/* ===========================
 *  UI: Canvas Art
 * =========================== */
async function askCanvasQty(replyToken) {
  const pairs = [
    ["1枚（基本 ¥5,400）","can_qty=1"],
    ["2枚（＋¥4,800）","can_qty=2"],
    ["3枚（＋¥9,300）","can_qty=3"],
    ["4枚（＋¥13,800）","can_qty=4"]
  ];
  await reply(replyToken, [
    { type: "flex", altText: "枚数選択", contents: buttonListBubble(CATALOG.canvas_art.img, "キャンバスアートの枚数を選択", pairs) }
  ]);
}
async function askCanvasSource(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "素材選択", contents: buttonListBubble(CATALOG.canvas_art.img, "デザインの素材を選択", CANVAS_SOURCE_CHOICES.map(c=>[c.label, `can_src=${c.v}`])) }
  ]);
}
async function askCanvasOption(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "加工/文字入れ", contents: buttonListBubble(CATALOG.canvas_art.img, "画像の加工/文字入れ（＋¥400）を希望しますか？", [["希望する","can_opt=on"],["不要","can_opt=off"]]) }
  ]);
}

/* ===========================
 *  UI: LINE Stamps
 * =========================== */
async function askStampPack(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "スタンプパック", contents: buttonListBubble(CATALOG.line_stamp.img, "パックを選択", STAMP_PACKS.map(p=>[`${p.label}：¥${p.price.toLocaleString()}`, `st_pack=${p.v}`])) }
  ]);
}
async function askStampPetHeads(replyToken) {
  await reply(replyToken, [
    { type: "flex", altText: "頭数選択", contents: buttonListBubble(CATALOG.line_stamp.img, "ペット頭数を選択", STAMP_PET_SURCHARGE.map(x=>[` ${x.n}匹 ${x.add?`（＋¥${x.add.toLocaleString()}）`:""}`, `st_pet=${x.n}`])) }
  ]);
}
async function askStampPublish(replyToken) {
  await reply(replyToken, [
    { type: "text", text: "掲載のご協力について：ワンちゃんのお写真と完成スタンプを紹介させていただくことがあります。" },
    { type: "flex", altText: "掲載可否", contents: buttonListBubble(CATALOG.line_stamp.img, "掲載の可否をお選びください", STAMP_PUBLISH_CHOICES.map(x=>[x.label, `st_pub=${x.v}`])) }
  ]);
}

/* ===========================
 *  Customer Info
 * =========================== */
async function askCustomerName(replyToken) { await reply(replyToken, [{ type: "text", text: "【お客様情報】\nお名前（フルネーム）をご入力ください。" }]); }
async function askCustomerPhone(replyToken) { await reply(replyToken, [{ type: "text", text: "お電話番号（ハイフンなし）をご入力ください。" }]); }
async function askCustomerPostal(replyToken){ await reply(replyToken, [{ type: "text", text: "郵便番号をご入力ください。" }]); }
async function askCustomerAddress1(replyToken){ await reply(replyToken, [{ type: "text", text: "ご住所をご入力ください。" }]); }
async function askCustomerAddress2(replyToken){ await reply(replyToken, [{ type: "text", text: "建物名・部屋番号があればご入力ください。（なければ「なし」）" }]); }

/* ===========================
 *  Builders: Push texts
 * =========================== */
function buildOrderLines(s) {
  const itemLines = s.order.map((it, idx) => {
    const p = CATALOG[it.product];
    const sub = it.amount || calcAmountItem(p, { side: it.side, opts: it.opts });

    let extra = "";
    if (it.product === "illustration") {
      const c = it.custom || {};
      const style = ILL_STYLE_CHOICES.find(x=>x.v===c.style)?.label ?? "";
      const comp  = ILL_COMPOSE_CHOICES.find(x=>x.v===c.compose)?.label ?? "";
      const ratio = ILL_RATIO_CHOICES.find(x=>x.v===c.ratio)?.label ?? "";
      extra = [ `頭数:${c.petCount}匹`, comp, style, ratio, c.textAdd ? "文字入れあり（＋400円）" : "" ].filter(Boolean).join(" / ");
    } else if (it.product === "canvas_art") {
      const c = it.custom || {};
      const src = CANVAS_SOURCE_CHOICES.find(x=>x.v===c.source)?.label ?? "";
      extra = [ `枚数:${c.qty}枚`, src, c.editAdd ? "加工/文字入れ（＋400円）" : "" ].filter(Boolean).join(" / ");
    } else if (it.product === "line_stamp") {
      const c = it.custom || {};
      const pack = STAMP_PACKS.find(x=>x.v===c.pack)?.label ?? "";
      const pub = STAMP_PUBLISH_CHOICES.find(x=>x.v===c.publish)?.label ?? "";
      extra = [ pack, `頭数:${c.petHeads}匹`, pub ].filter(Boolean).join(" / ");
    } else {
      const sideStr = it.side === "both" ? "両面プリント（＋2,000円）" : sideLabel(it.product, it.side);
      const hasText = it.opts && it.opts instanceof Set && it.opts.has(OPTION_KEY);
      const optStr  = hasText ? `／${OPTION_LABEL}` : "";
      extra = [ colorLabel(it.color), sizeLabel(it.size), [sideStr,optStr].filter(Boolean).join("") ].filter(Boolean).join("／");
    }

    return [`\n#${idx+1} ${p.label}`, extra, `→ ¥${sub.toLocaleString()}`].filter(Boolean).join("\n");
  });
  return itemLines;
}
function buildCustomerLines(cust) {
  return [
    `お名前：${cust.name}`,
    `電話：${cust.phone}`,
    `郵便番号：${cust.postal}`,
    `住所：${cust.address1}${cust.address2 ? " " + cust.address2 : ""}`
  ];
}
function buildCustomerConfirmationText(s) {
  const lines = ["【ご注文内容は以下の通りです】", ...buildOrderLines(s), `\n合計：¥${totalAmountAll(s).toLocaleString()}`, "", "【お届け先】", ...buildCustomerLines(s.customer), ""];
  return lines.join("\n");
}
function buildAdminConfirmationText(userId, s) {
  const lines = ["【注文確定（お客様控え送信済み）】", `userId: ${userId}`, ...buildOrderLines(s), `\n合計：¥${totalAmountAll(s).toLocaleString()}`, "", "【お届け先】", ...buildCustomerLines(s.customer)];
  return lines.join("\n");
}

/* ===========================
 *  UI building blocks
 * =========================== */
function buttonListBubble(heroImgPathOrUrl, title, pairs) {
  const buttons = pairs.map(([label, data]) => ({ type: "button", style: "secondary", action: { type: "postback", label, data } }));
  const isObj = title && typeof title === "object";
  const contents = [];
  if (isObj && title.linkText && title.link) {
    contents.push({ type: "text", text: String(title.linkText), weight: "bold", size: "lg", wrap: true, action: { type: "uri", uri: String(title.link) } });
    if (title.subtitle) contents.push({ type: "text", text: String(title.subtitle), size: "md", wrap: true });
  } else {
    contents.push({ type: "text", text: String(title), weight: "bold", size: "lg", wrap: true });
  }
  return {
    type: "bubble",
    hero: { type: "image", url: absoluteUrl(heroImgPathOrUrl), size: "full", aspectRatio: "20:13", aspectMode: "cover" },
    body: { type: "box", layout: "vertical", spacing: "md", contents: [...contents, ...buttons] }
  };
}
function cardBubble({ heroUrl, title, buttons }) {
  const btnNodes = buttons.map(([label, payload]) => ({ type: "button", style: "primary", action: { type: "postback", label, data: payload } }));
  return {
    type: "bubble",
    hero: { type: "image", url: heroUrl, size: "full", aspectRatio: "20:13", aspectMode: "cover" },
    body: { type: "box", layout: "vertical", contents: [{ type: "text", text: title, weight: "bold", size: "lg", wrap: true }] },
    footer: { type: "box", layout: "vertical", contents: btnNodes }
  };
}
function confirmationBubble({ title, orderText, customer }) {
  return {
    type: "bubble",
    body: {
      type: "box", layout: "vertical", spacing: "md",
      contents: [
        { type: "text", text: title, weight: "bold", size: "lg", wrap: true },
        { type: "text", text: "【ご注文】", weight: "bold" },
        { type: "text", text: orderText, wrap: true },
        { type: "text", text: "【お届け先】", weight: "bold", margin: "md" },
        { type: "text", text: `お名前：${customer.name}`, wrap: true },
        { type: "text", text: `電話：${customer.phone}`, wrap: true },
        { type: "text", text: `郵便番号：${customer.postal}`, wrap: true },
        { type: "text", text: `住所：${customer.address1}${customer.address2 ? " " + customer.address2 : ""}`, wrap: true }
      ]
    }
  };
}

/* ===========================
 *  Labels / Utils
 * =========================== */
function colorLabel(v) { return ({ white: "白", gray: "グレー", baby_pink: "ピンク", natural: "ナチュラル" }[v] || v || ""); }
function sizeLabel(v)  { return ({ kids120: "子供用120" }[v] || v || ""); }
function sideLabel(productKey, v) { const arr = SIDE_BY_PRODUCT[productKey] || []; return arr.find(x=>x.v===v)?.label || v || ""; }
function absoluteUrl(path) { if (!path) return PUBLIC_BASE; if (/^https?:\/\//i.test(path)) return path; const cleaned = String(path).trim().replace(/^\/+/, "/"); return `${PUBLIC_BASE}${cleaned}`; }
function normalizeText(t) { return (t || "").replace(/\s+/g, "").replace(/＆/g, "&").toLowerCase(); }

/* ===========================
 *  Square（決済リンク生成）
 * =========================== */
async function createSquarePaymentLink({ items = [], referenceId }) {
  if (!process.env.SQUARE_ACCESS_TOKEN || !process.env.SQUARE_LOCATION_ID || !SQUARE_API_BASE) {
    console.error("[Square] missing env:", { hasToken: !!process.env.SQUARE_ACCESS_TOKEN, hasLoc: !!process.env.SQUARE_LOCATION_ID, base: SQUARE_API_BASE });
    return null;
  }
  const lineItems = items.map(i => ({ name: String(i.name), quantity: String(i.quantity ?? 1), base_price_money: { amount: Math.round(Number(i.amount || 0)), currency: "JPY" } }));
  const payload = { idempotency_key: crypto.randomUUID(), order: { location_id: process.env.SQUARE_LOCATION_ID, line_items: lineItems, reference_id: referenceId || undefined } };
  try {
    const r = await fetch(`${SQUARE_API_BASE}/v2/online-checkout/payment-links`, {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${process.env.SQUARE_ACCESS_TOKEN}` },
      body: JSON.stringify(payload),
    });
    if (!r.ok) { const body = await r.text().catch(()=> ""); console.error("[Square] payment-links error:", r.status, body); return null; }
    const json = await r.json(); return json?.payment_link?.url || null;
  } catch (e) { console.error("Square link exception:", e); return null; }
}

/* ===========================
 *  LINE send (reply / push)
 * =========================== */
async function reply(replyToken, messages) {
  try {
    const r = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { Authorization: `Bearer ${process.env.LINE_CHANNEL_ACCESS_TOKEN}`, "Content-Type": "application/json" },
      body: JSON.stringify({ replyToken, messages })
    });
    if (!r.ok) { const t = await r.text().catch(()=>""); console.warn("[reply] non-2xx:", r.status, t); }
  } catch (e) { console.error("[reply] fetch error:", e); }
}
async function push(toId, messages) {
  try {
    const r = await fetch("https://api.line.me/v2/bot/message/push", {
      method: "POST",
      headers: { Authorization: `Bearer ${process.env.LINE_CHANNEL_ACCESS_TOKEN}`, "Content-Type": "application/json" },
      body: JSON.stringify({ to: toId, messages })
    });
    if (!r.ok) { const t = await r.text().catch(()=> ""); console.warn("[push] non-2xx:", r.status, t); }
  } catch (e) { console.error("[push] fetch error:", e); }
}

/* ===========================
 *  入金連絡メッセージ判定
 * =========================== */
const PAID_CORE =
  '(?:ご?入金\\s*(?:済み?|完了|(?:いた)?しました?)|入金[済了]|にゅうきん\\s*(?:ずみ|すみ|完了|(?:いた)?しました?)|お?振(?:り)?込(?:み)?\\s*(?:済み?|完了|(?:いた)?しました?)|振込\\s*(?:済み?|完了|(?:いた)?しました?)|ふりこみ\\s*(?:ずみ|すみ|完了|(?:いた)?しました?)|お?送金\\s*(?:済み?|完了|(?:いた)?しました?)|そうきん\\s*(?:ずみ|すみ|完了|(?:いた)?しました?)|お?支払(?:い)?\\s*(?:済み?|完了|(?:いた)?しました?)|お?支払い\\s*(?:済み?|完了|(?:いた)?しました?)|支払[済了]|決済\\s*(?:完了|(?:いた)?しました?))';
const PAID_TEXT_RE = new RegExp('^\\s*[「『〝（(\\[]?\\s*(?:' + PAID_CORE + ')\\s*[」』〟）)\\]]?\\s*[。.!！…〜ー]*\\s*$', 'i');
const VAGUE_PAID_RE = /^(?:済み|すみ|完了|かんりょう)[。.!！…〜ー]?\s*$/i;
function isPaidMessage(text, { allowVague = false } = {}) { if (!text) return false; if (PAID_TEXT_RE.test(text)) return true; return allowVague && VAGUE_PAID_RE.test(text); }

/* ===========================
 *  Misc
 * =========================== */
function cardText(s){ return s; }
function setChoice(fs, newValue, jpLabel, jpValueText) {
  if (fs.value === newValue) return null;
  let msg;
  if (fs.isFirst) { msg = jpLabel === "バリエーション" ? `「${jpValueText}」が選択されました。` : `${jpLabel}を「${jpValueText}」が選択されました。`; fs.isFirst = false; }
  else { msg = jpLabel === "バリエーション" ? `「${jpValueText}」に変更しました。` : `${jpLabel}を「${jpValueText}」に変更しました。`; }
  fs.value = newValue; return msg;
}

/* Vercel Body Size */
export const config = { api: { bodyParser: { sizeLimit: "1mb" } } };
