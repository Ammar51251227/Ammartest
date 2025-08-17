
# DR.iPhone — KNET → USDT Bridge (Netlify Functions)

> ⚠️ تنبيه: KNET تسويتها تكون **بالدينار الكويتي KWD إلى حسابك البنكي** عبر مزوّد الدفع (مثل Tap أو MyFatoorah)،
> ولا تقوم بالتحويل مباشرةً إلى عملة رقمية أو إلى عنوان USDT. لتحقيق تحويل تلقائي إلى **USDT/ERC20**
> تحتاج إلى خادم يقوم بعد نجاح الدفع بتنفيذ **تحويل/سحب** من رصيدك المسبق في منصة تداول (Binance مثالًا)
> إلى محفظتك: `0x21e36C957fFA87B379aFe41Cf842F49f8Dc7B676`.

## ما الذي تحتويه الحزمة؟
- `site/` نسخة المتجر مع تعديل زر KNET ليستدعي دالة خادمية (`/.netlify/functions/create_knet_session`).
- `netlify/functions/` توابع Serverless:
  - `create_knet_session.js` : إنشاء جلسة دفع KNET عبر Tap أو MyFatoorah.
  - `payment_webhook.js` : استقبال Webhook، تحقق الدفع، ثم تنفيذ تحويل USDT (من رصيدك المسبق).
  - `ping.js` : فحص سريع.

## الإعداد السريع
1) أنشئ موقعًا على Netlify وارفع هذا المجلد (سيقرأ `netlify.toml` تلقائيًا).
2) في إعدادات البيئة (Environment variables) ضَع:
   - **اختَر المزود**: `PAYMENT_PROVIDER` = `TAP` أو `MYFATOORAH`
   - إن اخترت TAP:
     - `TAP_SECRET_KEY` = من لوحة Tap
     - (اختياري) `TAP_WEBHOOK_SECRET`
   - إن اخترت MyFatoorah:
     - `MYFATOORAH_TOKEN` = من لوحة MyFatoorah
   - إعدادات عامة:
     - `KNET_RETURN_URL` ، `KNET_ERROR_URL`
     - **Binance** (اختياري لميزة التحويل التلقائي): `BINANCE_API_KEY`, `BINANCE_API_SECRET`, `USDT_DEST_ADDRESS` = `0x21e36...B676`, `USDT_NETWORK` = `ETH`, `KWD_USD_RATE` = `3.25` (مثال).
3) في لوحة مزوّد الدفع، عيّن عنوان Webhook إلى:
   - `https://YOUR_NETLIFY_SITE.netlify.app/.netlify/functions/payment_webhook`

## كيف يعمل التدفق؟
1) العميل يضغط **ادفع KNET** → استدعاء `create_knet_session` → إعادة توجيه لصفحة KNET (مزودك).
2) بعد الدفع، مزوّدك يستدعي `payment_webhook` مع تفاصيل العملية.
3) إذا كانت الحالة **مدفوعة**، ننفّذ:
   - **(اختياري)** تحويل فوري USDT من رصيدك المسبق في Binance وسحب إلى العنوان المحدد.
   - نوصي بإبقاء رصيد USDT **مسبق التمويل** لتفادي تأخير التسوية البنكية.

## ملاحظات مهمة
- مزوّدات KNET (Tap/MyFatoorah) **تُسوي إلى البنك**. التحويل إلى كريبتو يتطلب خطوة منفصلة (Exchange/OTC).
- الالتزام بمتطلبات الامتثال (AML/KYC) شرط أساسي.
- الكود الخاص بـ Binance هنا **مثال مبسّط**؛ يجب تطبيق التوقيع HMAC وإدارة الأخطاء والتقفيل الأمني قبل الإنتاج.
