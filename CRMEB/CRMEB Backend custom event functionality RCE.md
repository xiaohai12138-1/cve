## Vulnerability Introduction
The custom event feature in the CRMEB backend contains a persistent server-side code execution vulnerability. An attacker with high-privilege backend access can create a custom event via `POST /adminapi/system/event/save` and inject attacker-controlled PHP code into the `customCode` field; subsequently, when the corresponding business event is triggered, this code is directly executed by the global event listener using `eval()`.

Source code download link：https://gitee.com/ZhongBangKeJi/CRMEB
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB/image.7p4akhk2iy.webp

## Vulnerability Analysis

To reproduce the vulnerability, modify the `.env` file by setting `APP_DEBUG = true`. In `crmeb/app/services/system/SystemEventServices.php` (lines 487–498), the `customCode` variable is JSON-encoded and then written to the database.
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.9gx9ffhrsd.webp

With the global event configuration enabled in lines 23–48 of `crmeb/app/event.php`, `CustomEventListener` is registered with the application's event system.
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.7i12p3ccyl.webp

When a matching event is dispatched, lines 18–25 of `crmeb/app/listener/CustomEventListener.php` query all enabled custom event records associated with the current event tag and execute `eval` on each record.
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.5c1o3bks0a.webp

## Vulnerability reproduction

Maintenance --> Custom Events --> Add Custom Event
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.7axutnqc95.webp
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.2h8zxj5pou.webp

After submitting, you need to log back into the backend to trigger the event; alternatively, you can select a different event type to trigger it. Once triggered, simply access it.
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.1e9amn9y3c.webp

Data package contents
https://github.com/xiaohai12138-1/picx-images-hosting/raw/master/CRMEB-2/image.5trprwmeke.webp
