# Chinese Payment / Commerce Apps

The following Chinese-market payment and commerce apps are pre-installed on the YodaOS device in the `product/app` partition. These are region-specific apps for the Chinese market and are not documented further.

| App Name | Package                         | Partition    | Description                                        |
|----------|----------------------------------|--------------|----------------------------------------------------|
| Alipay   | com.eg.android.AlipayGGlasses   | product/app  | Alipay client for AR glasses (Ant Group)           |
| AntPay   | com.iap.mobile.ar_pay           | product/app  | Ant Financial AR payment service                   |
| JdPay    | com.jd.jr.joyaibuy              | product/app  | JD.com AR shopping / payment client (JD Finance)   |

These apps are integrated with the Rokid Sprite Assist Server (`com.rokid.os.sprite.assistserver`), which exposes a `PaymentService` and queries both `com.iap.mobile.ar_pay` (AntPay) and `com.tencent.glasswxpay.glassapp` (WeChat Pay, not found pre-installed) for payment functionality.

The JD client SDK service (`com.jd.jr.joygoclient.SDKService`) is also bundled inside the Sprite Assist Server APK.
