---
{"dg-publish":true,"permalink":"/ctf/forest-htb-oscp-like/"}
---


HTB'nin en güzel yanlarından biri, daha önce karşılaştığım hiçbir CTF'ye benzemeyen Windows konseptlerini ortaya çıkarması. Forest bunun harika bir örneği. RPC üzerinden kullanıcıları listelememe, AS-REP Roasting ile Kerberos'a saldırı yapmama ve bir shell almak için Win-RM kullanmama izin veren bir domain controller. Daha sonra DCSycn yeteneklerini elde etmek için bu kullanıcının izinlerinden ve erişimlerinden faydalanabilirim, böylece admin kullanıcısı için hash'leri dump edebilir ve admin olarak bir shell elde edebilirim. Beyond Root bölümünde, DCSync'in kablo üzerinde nasıl göründüğüne ve izinleri temizleyen otomatik göreve bakacağım. "Beyond Root" kısmında, DCSync'in ağ trafiğinde nasıl göründüğünü inceleyeceğim ve izinleri temizleyen otomatik görevi analiz edeceğim.

## Box Info

|Name|[Forest](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)[![Forest](https://0xdf.gitlab.io/icons/box-forest.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)|
|---|---|
|Release Date|[12 Oct 2019](https://twitter.com/hackthebox_eu/status/1215300155341266951)|
|Retire Date|21 Mar 2020|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Forest](https://0xdf.gitlab.io/img/forest-diff.png)|
|Radar Graph|![Radar chart for Forest](https://0xdf.gitlab.io/img/forest-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:20:45[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|01:23:31[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|Creators|[![egre55](https://www.hackthebox.com/badge/image/1190)](https://app.hackthebox.com/users/1190)  <br>[![mrb3n](https://www.hackthebox.com/badge/image/2984)](https://app.hackthebox.com/users/2984)|


