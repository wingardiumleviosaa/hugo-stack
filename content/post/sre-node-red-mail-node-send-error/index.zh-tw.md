---
title: "[Node-RED] mail node 534-5.7.14 Error"
tags:
  - Node-RED
categories:
  - SRE
date: 2021-10-26 09:39:00
slug: nodered-mail-node-error
---

當 google 帳號設置了啟用 **`允許安全性較低的應用程式`**，但 node-red 的 mail node 還是出現 `534-5.7.14 Please log in via your web browser and then try again.` 的錯誤。

<!--more-->

可能是因為 Gmail 自動阻擋了可疑的登入。

1. 此時到用來發信的 Gmail 信箱，可發現一封 `系統已阻止可疑的登入`的系統通知信。
2. 或到[異常活動](https://security.google.com/settings/security/activity)頁面  ，也可看到 `應用程式/裝置登入嘗試遭拒` 的記錄。
3. 如果通知信、異常活動記錄中的 IP 與時間是正常由我們的程式發出的，則可到 `授權存取您的 Google 帳戶` 頁面  [https://accounts.google.com/DisplayUnlockCaptcha](https://accounts.google.com/DisplayUnlockCaptcha)，按 `繼續`。
4. 然到到原本的程式，執行登入/寄信，應該就可以認證了。

其他：

- 如果原程式換了 IP，可能會再次被 Gmail 自動阻擋，則須在進行一次以上的的動作。
- 有時候顯示密碼錯誤，有可能密碼是對的，但被 Google 當成可疑的登入阻擋了。

## Source

- [https://xyz.cinc.biz/2014/09/gmail-password-not-accepted-from-server.html](https://xyz.cinc.biz/2014/09/gmail-password-not-accepted-from-server.html)
- [https://stackoverflow.com/questions/20337040/getting-error-while-sending-email-through-gmail-smtp-please-log-in-via-your-w](https://stackoverflow.com/questions/20337040/getting-error-while-sending-email-through-gmail-smtp-please-log-in-via-your-w)

---

另外補充一下 email node 的使用

![](https://imgur.com/N6OrFDA.png)

function node

```js
msg.to = "ulahsieh@domain.com";
msg.topic = "信件主旨";
msg.payload = "信件內容";
return msg;
```

在 function node 寫上寄件者後，mail node 的 `To` 欄位即可省略。

![](https://imgur.com/GOh6yba.png)
