---
title: "[Node.js] 使用 NodeMailer 發送信件"
tags:
  - Node.js
categories:
  - Programming
date: 2022-12-18 19:52:00
slug: node-send-mail-by-nodemailer
---

## TL;DR

在本地端透過 nodemailer 寄送郵件時，發現預設的 25 port 會有 `connect ECONNREFUSED` 的錯誤無法傳送。

<!--more-->

## Solution

```js
import * as nodemailer from "nodemailer";
import { Options } from "nodemailer/lib/mailer";

async function mail() {
  const transporter = nodemailer.createTransport({
    host: "hqsmtp.abc.com",
    port: 587,
    secure: false,
    auth: {
      user: "test1@abc.com",
      pass: "!Passw0rd",
    },
    tls: {
      rejectUnauthorized: false,
    },
  });

  const options: Options = {
    from: "test1@abc.com",
    to: "test2@abc.com",
    subject: "test",
    text: "test",
  };
  await transporter.sendMail(options);
  transporter.close();
}
```

- secure &lt;boolean&gt; – If false (the default) then TLS is used if server supports the STARTTLS extension. For port 587 or 25 keep it false

- rejectUnauthorized &lt;boolean&gt; – If not false the server will reject any connection which is not authorized with the list of supplied CAs. This option only has an effect if requestCert is true. Default: true.

## Reference

- https://stackoverflow.com/questions/46742402/error-self-signed-certificate-in-certificate-chain-nodejs-nodemailer-express
