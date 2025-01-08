---
{"dg-publish":true,"permalink":"/ctf/investigation/"}
---


![Pasted image 20250108190926.png](/img/user/Pasted%20image%2020250108190926.png)


Araştırma, kullanıcıların resim yüklemesine izin veren ve bu resimlerde [[Exiftool\|Exiftool]] çalıştıran bir web sitesiyle başlıyor. Kullanılan sürümde bir komut enjeksiyonu zafiyeti var. Bu zafiyeti inceleyecek ve ardından bir başlangıç erişimi elde etmek için istismar edeceğim. Daha sonra bir dizi **Windows olay günlüğü** bulup bunları analiz ederek bir parola çıkaracağım. Son olarak, **root** olarak çalışan bir kötü amaçlı yazılım bulup, bunu anlamaya çalışarak komut yürütme elde edeceğim.

## Box Info

|Name|[Investigation](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)[![Investigation](https://0xdf.gitlab.io/icons/box-investigation.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)|
|---|---|
|Release Date|[21 Jan 2023](https://twitter.com/hackthebox_eu/status/1616103239899914240)|
|Retire Date|22 Apr 2023|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Medium [30]|
|Rated Difficulty|![Rated difficulty for Investigation](https://0xdf.gitlab.io/img/investigation-diff.png)|
|Radar Graph|![Radar chart for Investigation](https://0xdf.gitlab.io/img/investigation-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:17:46[![jkr](https://www.hackthebox.com/badge/image/77141)](https://app.hackthebox.com/users/77141)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:23:49[![xct](https://www.hackthebox.com/badge/image/13569)](https://app.hackthebox.com/users/13569)|
|Creator|[![Derezzed](https://www.hackthebox.com/badge/image/15515)](https://app.hackthebox.com/users/15515)|

## Recon

### nmap


