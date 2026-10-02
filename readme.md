# file archive 辅助工具

通过 GitHub Actions 下载文件并保存到 [Releases](https://github.com/jcleng/filearchive/releases)，同时记录 `sha256sum`。

## DaVinci Resolve 下载原理

`RESOLVE_DLID`（在 Blackmagic 官方 API 中称为 `downloadId`）**不是人工推算的，而是由 Blackmagic Design 官方 API 返回的字段**。分析 [night199uk/resolve-flatpak](https://github.com/night199uk/resolve-flatpak) 的 `installer/api.py` 得到来源链路：

1. **下载页 refer_id**：固定值 `77ef91f67a9e411bbbe299e595b4cfcc`，对应 URL `.../support/download/77ef91f67a9e411bbbe299e595b4cfcc/Linux`，仅用于构造 `Referer` 请求头，不是下载 ID 本身。

2. **获取 downloadId 的两个 API**：

   - 取最新稳定版：

     ```
     GET https://www.blackmagicdesign.com/api/support/latest-stable-version/{app_tag}/linux
     # 返回 JSON 中的 linux.downloadId 即 RESOLVE_DLID
     # app_tag: davinci-resolve (免费版) / davinci-resolve-studio (Studio 版)
     ```

   - 列出所有可用下载（可拿到任意版本）：

     ```
     GET https://www.blackmagicdesign.com/api/support/en/downloads.json
     # 返回列表中每个条目的 urls.Linux[].downloadId 即 RESOLVE_DLID
     # 同时 downloadTitle 可解析出版本号，如 "DaVinci Resolve 19.1.4"
     ```

3. **downloadId → 真实下载 URL**：拿到 `downloadId` 后，POST 到

   ```
   POST https://www.blackmagicdesign.com/api/register/us/download/{downloadId}
   ```

   并附带注册表单（姓名/邮箱等），返回带签名、有时效的 `SIGNED_URL`，再用该 URL 下载安装包。

> 当前工作流（`.github/workflows/DaVinci_Resolve.yml`）的 `RESOLVE_DLID` 为手动输入参数，默认 `ee1da4f13df74d72b6da783ead2ed875`（对应 DaVinci Resolve 19.0.3）。如需全自动，可在 workflow 中先请求 `downloads.json`，用 `jq` 按版本号解析出 `downloadId`，则只需填 `PKGVER`。

## 工作流说明

- 文件超过 **2GB** 时自动用 `split -b 1610612736`（1.5GB）拆分为分片上传；未超过则原文件上传。
- 下载后合并分片命令：

  ```bash
  cat DaVinci_Resolve_*.zip.part* > DaVinci_Resolve_Linux.zip
  ```

## Blackmagic Design 全部 Linux 可用下载（含 downloadId）

> 数据来源：`https://www.blackmagicdesign.com/api/support/en/downloads.json`（抓取于 2026-10-02）。`downloadId` 即工作流所需的 `RESOLVE_DLID`。

| 产品 / 版本 | product | downloadId | 日期 |
| --- | --- | --- | --- |
| DaVinci Resolve 21.1.1 | davinci-resolve | bc1eb63d0e51443892a43033cb039201 | Today |
| DaVinci Resolve Studio 21.1.1 | davinci-resolve-studio | 708182cd91f14d638def48d0d33c08c9 | Today |
| Fusion Studio 21.1.1 | fusion-studio | cc838d60ea704c029a6c16fc6c528de2 | Today |
| Blackmagic RAW 4.3.1 | braw-sdk | ec9fc1d7d79f44a6833f0002587a411e | 31 Oct 2024 |
| DaVinci Resolve 16.1.1 | davinci-resolve | 5566b13b3db24d72ae4f2b0af2da2534 | 31 Oct 2019 |
| DaVinci Resolve Studio 16.1.1 | davinci-resolve-studio | e86c5ea2c77f4e07bb401c9ef8c684fa | 31 Oct 2019 |
| DaVinci Resolve 11.1.1 | davinci-resolve-studio | 1901b56b8fc346b68d8dbfdf91fc1d7d | 31 Oct 2014 |
| DaVinci Resolve 17.1.1 | davinci-resolve | 6c68f757d62342e2a9fd295d854a8c44 | 31 Mar 2021 |
| DaVinci Resolve Studio 17.1.1 | davinci-resolve-studio | bf8c4f6eac894d4c84cfe9ba8518d388 | 31 Mar 2021 |
| Fusion Studio 17.1.1 | fusion-studio | a8522e2b3eec49f28a62c0a23d3bade1 | 31 Mar 2021 |
| DaVinci Resolve 16.2.5 | davinci-resolve | 702dc3eb8e7641fe8617c2cb99cd89d6 | 31 Jul 2020 |
| DaVinci Resolve Studio 16.2.5 | davinci-resolve-studio | 7fc7eab939054db586cab85cb1d6ee0f | 31 Jul 2020 |
| DaVinci Resolve 18.0.4 | davinci-resolve | 5c482c557a2940138ceb320653715d66 | 30 Sep 2022 |
| DaVinci Resolve Studio 18.0.4 | davinci-resolve-studio | 12bfbf375af042e5b862519616278f76 | 30 Sep 2022 |
| Fusion Studio 18.0.4 | fusion-studio | 22eb3c7927f3445e91cc4d15a2483295 | 30 Sep 2022 |
| Blackmagic RAW 2.7 | braw-sdk | 6e930a8a85b84597a3737331728b3062 | 30 Sep 2022 |
| Desktop Video 15.2 | desktop-video | aa20aa867f12463ba7cac0879784c465 | 30 Oct 2025 |
| Desktop Video 15.2 SDK | desktop-video-sdk | 5497ce8fc2c8400fb88cc633af73dfbe | 30 Oct 2025 |
| Blackmagic RAW 1.5.2 | braw-sdk | 47f7842761654d5c81146e99c95a475f | 30 Oct 2019 |
| DaVinci Resolve 20.3 | davinci-resolve | 5dd107f0a88e4433a6c20c73ef0f0055 | 30 Nov 2025 |
| DaVinci Resolve Studio 20.3 | davinci-resolve-studio | 87078fd0cd7f4436a34cf9a421ff1006 | 30 Nov 2025 |
| Fusion Studio 20.3 | fusion-studio | 22846d1eb31e4a9f8b2e3a95d5dda5f3 | 30 Nov 2025 |
| Blackmagic RAW 3.5 | braw-sdk | e6cfa4c2e25c433382cfe7af27a14940 | 30 Nov 2023 |
| Desktop Video 11.2 | desktop-video | cae6708b1743402693c6ae208f5e9fb8 | 30 May 2019 |
| Desktop Video 11.2 SDK | desktop-video-sdk | c63b4b939005460b880d5cf8e7885e43 | 30 May 2019 |
| DaVinci Resolve 11.3 | davinci-resolve-studio | 67a6bedd16d34dd78b19ffb37666826c | 30 Mar 2015 |
| DaVinci Resolve 20.0.1 | davinci-resolve | 9167b249b93c4d0ab8928bfa64a90794 | 30 Jun 2025 |
| DaVinci Resolve Studio 20.0.1 | davinci-resolve-studio | ef5d9061181b4de4970d2c8f3bfd237e | 30 Jun 2025 |
| Fusion Studio 20.0.1 | fusion-studio | f459f9cdc8e248e2a4ccf5b7c7cee93e | 30 Jun 2025 |
| Blackmagic RAW 2.6 | braw-sdk | b890ec839a404bf882a7dfdf7dff11e7 | 30 Jun 2022 |
| Blackmagic Fusion 9.0.2 Studio | fusion-studio | e01fe8e134c14d308384fe15f0dbb5fc | 30 Jul 2018 |
| Blackmagic Fusion 9.0.2 | fusion | 8e1149d13d6f4910b15f523f9f43ff48 | 30 Jul 2018 |
| DaVinci Resolve 17.4.1 | davinci-resolve | 46b47253f7974ba0a0e5ba7bc66786f2 | 29 Oct 2021 |
| DaVinci Resolve Studio 17.4.1 | davinci-resolve-studio | 7b115c3db2424e3ca307ee58b916021e | 29 Oct 2021 |
| Fusion Studio 17.4.1 | fusion-studio | f5b968176b544122a90d171af51cf2fb | 29 Oct 2021 |
| DaVinci Resolve 15.2.1 Studio | davinci-resolve-studio | 79650b52b38d4e0c8cce493e4178e801 | 29 Nov 2018 |
| DaVinci Resolve 15.2.1 | davinci-resolve | ecddd8d199644ef1a7b90bf153b9a2e7 | 29 Nov 2018 |
| Desktop Video 10.9.3 | desktop-video | 593cebbfb81f43e3a24ca19a8679df4c | 29 May 2017 |
| Desktop Video 10.9.3 SDK | desktop-video-sdk | 07f27b20a4934e98a104e87cbdce746a | 29 May 2017 |
| DaVinci Resolve 17.4.6 | davinci-resolve | 0e34f66357634628b918e96b6680c137 | 29 Mar 2022 |
| DaVinci Resolve Studio 17.4.6 | davinci-resolve-studio | 34023b93d9f64d03aaf7654e2cbbb727 | 29 Mar 2022 |
| Fusion Studio 17.4.6 | fusion-studio | 7ca357d237724b58bf4896994480c9fa | 29 Mar 2022 |
| Desktop Video 12.5 | desktop-video | fecacc0f9b2f4c2e8bf2863e9e26c8e1 | 29 Jun 2023 |
| Desktop Video 12.5 SDK | desktop-video-sdk | 359e4b2f20df4547bc443ad90b556968 | 29 Jun 2023 |
| Desktop Video 12.8 | desktop-video | 904793398795438db9a070ab1151d92c | 29 Jan 2024 |
| Desktop Video 12.8 SDK | desktop-video-sdk | 82671d9009c448d7b4ac61fdfacf3d6d | 29 Jan 2024 |
| DaVinci Resolve 12.5.3 Studio | davinci-resolve-studio | a5ac55dc67f64adbb8a0dc380e4a9853 | 28 Oct 2016 |
| DaVinci Resolve 20.0 | davinci-resolve | 9cf153267dbb4d9487ac8c22ff14d543 | 28 May 2025 |
| DaVinci Resolve Studio 20.0 | davinci-resolve-studio | 407110f9045e410996bb9ff3ad6956d5 | 28 May 2025 |
| Fusion Studio 20.0 | fusion-studio | 256b55c8e6144b0d9b4120388d2e0038 | 28 May 2025 |
| Desktop Video 16.2 | desktop-video | 606db7a66c534e818c253c6df87a3972 | 28 Jul 2026 |
| Desktop Video 14.4.1 | desktop-video | bc31044728f146859c6d9e0ccef868d8 | 28 Jan 2025 |
| Blackmagic RAW 4.4 | braw-sdk | 598fd609746249a6a400ebad2d7d4bd9 | 28 Jan 2025 |
| DaVinci Resolve 15.1.1 Studio | davinci-resolve-studio | 3c251921709c49ef88a9cc8616ffa2b8 | 27 Sep 2018 |
| DaVinci Resolve 15.1.1 | davinci-resolve | f43b89d94a0c4620a9cdc7acfece2848 | 27 Sep 2018 |
| Desktop Video 10.8.2 | desktop-video | a001da6b18954d31adaa43a367a0c719 | 27 Oct 2016 |
| Blackmagic RAW 1.6 | braw-sdk | 4e6fa8183fab48d0a942460350583127 | 27 Nov 2019 |
| Desktop Video 9.7.3 | desktop-video | 42eafb4b852c43d48af5382199cccbec | 27 May 2013 |
| Blackmagic RAW 4.1 | braw-sdk | 727197935d0a4b8cb14c1685d37c27c0 | 27 Jun 2024 |
| Desktop Video 10.8.5 | desktop-video | 55714dd0990b40e386325a0487955082 | 27 Feb 2017 |
| Desktop Video 10.8.5 SDK | desktop-video-sdk | 07a7d4cc1327424b9c931148161dc5a0 | 27 Feb 2017 |
| DaVinci Resolve 16.2.6 | davinci-resolve | 97dd8ef364874c3fb470c4e4cb8d1c54 | 27 Aug 2020 |
| DaVinci Resolve Studio 16.2.6 | davinci-resolve-studio | 45b1f98304804d659e0e2e1bdeb55c1c | 27 Aug 2020 |
| Blackmagic RAW 1.8.1 | braw-sdk | b92e2fb4caa24e8a9d2adb7e85523c6e | 27 Aug 2020 |
| Desktop Video 12.7 | desktop-video | b94e51eb5c6c45c48a04b09b72989448 | 26 Sep 2023 |
| Desktop Video 12.7 SDK | desktop-video-sdk | 295127ef439940e798dde165c0ae2ef9 | 26 Sep 2023 |
| Desktop Video 12.2 | desktop-video | f3106b481e2b4cd5934a650c869be25f | 26 Oct 2021 |
| Desktop Video 12.2 SDK | desktop-video-sdk | a87d33e6b5f5414aa2dabcde93774b87 | 26 Oct 2021 |
| Desktop Video 14.3 | desktop-video | 357a1702a28c4901b37e44beb29efd6c | 26 Nov 2024 |
| Desktop Video 14.3 SDK | desktop-video-sdk | 56e90be48aaf4c7b8480b79092d837c5 | 26 Nov 2024 |
| Desktop Video 10.5.2 | desktop-video | a846a8e1b2bb48e089ba5dca7d44070c | 26 Nov 2015 |
| Desktop Video 10.5.2 SDK | desktop-video-sdk | d499445832344a8d951b4e7dfa73dca8 | 26 Nov 2015 |
| Desktop Video 10.1.2 | desktop-video | c59f8c3ef602460b973b52c51fb8900a | 26 Jun 2014 |
| DaVinci Resolve 18.0.1 | davinci-resolve | 3355657703a0429a8a197030e84cdf76 | 26 Jul 2022 |
| DaVinci Resolve Studio 18.0.1 | davinci-resolve-studio | 78b4f5edaf6e47b99a489c7f084888a3 | 26 Jul 2022 |
| Fusion Studio 18.0.1 | fusion-studio | c723102afaec4a1d98f36e79fe6b4e77 | 26 Jul 2022 |
| DaVinci Resolve 20.1.1 | davinci-resolve | 81779a00206a4508a3eff8bb4bb03c02 | 26 Aug 2025 |
| DaVinci Resolve Studio 20.1.1 | davinci-resolve-studio | 6b3cc20ee7134881a5d2079fc30f924f | 26 Aug 2025 |
| Fusion Studio 20.1.1 | fusion-studio | 6f43e5573f09473882c6fb405b7c8f5e | 26 Aug 2025 |
| DaVinci Resolve 16.2.8 | davinci-resolve | 3dc96708d3e841c4a19f9ca17aca2e6b | 25 Nov 2020 |
| DaVinci Resolve Studio 16.2.8 | davinci-resolve-studio | 2931a6b5a2ab4a1e8a2690b807005685 | 25 Nov 2020 |
| Blackmagic RAW 1.8.2 | braw-sdk | 946593167c5d44bb8fc6dd49d8ac8247 | 25 Nov 2020 |
| Blackmagic RAW 2.0 | braw-sdk | b13f819fac2a485bb8f0b0dc0e8ec12d | 25 Mar 2021 |
| Desktop Video 9.5.3 | desktop-video | 4e773ad3c50d4faa80042a2c7638be00 | 25 Jun 2012 |
| DaVinci Resolve 17.0 | davinci-resolve | 6ed62557482c400b8066824c4d270103 | 25 Feb 2021 |
| DaVinci Resolve Studio 17.0 | davinci-resolve-studio | 75e93671c322457183eb5642d96605ee | 25 Feb 2021 |
| Fusion Studio 17.0 | fusion-studio | f7091064e9ee4ee491aabf102dc89aee | 25 Feb 2021 |
| DaVinci Resolve 12.0.1 Studio | davinci-resolve-studio | daa453ad433b481d9147d3a6f123f3ed | 24 Sep 2015 |
| Desktop Video 15.3 | desktop-video | 0d37a837bbfd457cacb29001473f9cb0 | 24 Nov 2025 |
| Desktop Video 15.3 SDK | desktop-video-sdk | 39a6e8089f0b4ec7b54bc0306ead069c | 24 Nov 2025 |
| Blackmagic Cintel 5.0 | cintel | 19667e3870254d0f8973b2ba601d2b18 | 24 Nov 2022 |
| Blackmagic Cintel 5.0 SDK | cintel-sdk | fe519d3301714f5a8673dab2ada7c43e | 24 Nov 2022 |
| DaVinci Resolve 21.0.1 | davinci-resolve | 0d8163c312d14e9489776bd5c382c0f8 | 24 Jun 2026 |
| DaVinci Resolve Studio 21.0.1 | davinci-resolve-studio | a95b1e7f4f8846bdb2eb4500f1793229 | 24 Jun 2026 |
| Fusion Studio 21.0.1 | fusion-studio | 3625e72ff855410cb0a2ea7ee5414a2b | 24 Jun 2026 |
| Blackmagic RAW 2.1 | braw-sdk | 97379c669e174b1b98ef4c4a4ff84320 | 24 Jun 2021 |
| Blackmagic RAW 4.2 | braw-sdk | 7161141a704445a8861f97a642ee31f0 | 24 Jul 2024 |
| DaVinci Resolve 15.2.3 Studio | davinci-resolve-studio | 4f8fe0d5e283447d8be0209ee0f2a4c1 | 24 Jan 2019 |
| DaVinci Resolve 15.2.3 | davinci-resolve | 89633cfa41e142c9bdc1724fbb5ff873 | 24 Jan 2019 |
| Desktop Video 10.3.4 | desktop-video | f14252fda5ca4b90b1f151b97c7270f9 | 24 Dec 2014 |
| Desktop Video 16.4 | desktop-video | ff43241f07c64883a88a75fe5da51b56 | 24 Aug 2026 |
| Desktop Video 12.3 | desktop-video | b69591b5747f4522b97925a53b01a712 | 24 Apr 2022 |
| Desktop Video 12.3 SDK | desktop-video-sdk | e820e0d2575a45c49225d4b77bc40550 | 24 Apr 2022 |
| Desktop Video 10.9 | desktop-video | 46d65f46d6434b16bd69482b0ca7dba3 | 24 Apr 2017 |
| Desktop Video 10.9 SDK | desktop-video-sdk | 70acb24d87294f33993ed055fa6a9a93 | 24 Apr 2017 |
| DaVinci Resolve 20.2.1 | davinci-resolve | c7fe3923d6c043bca9ffd4de1a879625 | 23 Sep 2025 |
| DaVinci Resolve Studio 20.2.1 | davinci-resolve-studio | 38ceb164b36e49d587cc771de75e7483 | 23 Sep 2025 |
| Fusion Studio 20.2.1 | fusion-studio | befe2f940a034317be10017fa915e8af | 23 Sep 2025 |
| Desktop Video 12.7.1 | desktop-video | 873e0f71e6bb4f9992ee372b1aaac8dc | 23 Oct 2023 |
| Desktop Video 10.11.4 | desktop-video | f9a1f5fda76447838a8d0e5fb363dcd8 | 23 Oct 2018 |
| Desktop Video 10.11.4 SDK | desktop-video-sdk | 24a482e1e9154ef793aa71e5d3b3a391 | 23 Oct 2018 |
| DaVinci Resolve 18.1.1 | davinci-resolve | e09749f2de1d4c20a2b707c405d243fd | 23 Nov 2022 |
| DaVinci Resolve Studio 18.1.1 | davinci-resolve-studio | f4d7a8482d244ab885e503d291155269 | 23 Nov 2022 |
| Fusion Studio 18.1.1 | fusion-studio | deb315199a1e49c49c33d1748f104e27 | 23 Nov 2022 |
| DaVinci Resolve 14.1.1 Studio | davinci-resolve-studio | 56ea0ea1346a46f98da4dd50dd5fda2d | 23 Nov 2017 |
| DaVinci Resolve 14.1.1 | davinci-resolve | c69083eb2df541e2a1dc696f208afe2a | 23 Nov 2017 |
| Blackmagic RAW 2.5 | braw-sdk | 7ddd2336383745f2b34dd74c96a70795 | 23 Jun 2022 |
| DaVinci Resolve 14.2.1 Studio | davinci-resolve-studio | c482ded7f6884cf88feeaca62bd15d20 | 23 Jan 2018 |
| DaVinci Resolve 14.2.1 | davinci-resolve | 7229349439bc4b94870a1a2ce3e79071 | 23 Jan 2018 |
| Desktop Video 10.3.5 | desktop-video | 444a55643b814918815b2c484205167b | 23 Jan 2015 |
| Desktop Video 10.6 | desktop-video | aeb7a0cef29747d595efe5ab1cdd47dc | 23 Feb 2016 |
| Desktop Video 10.6 SDK | desktop-video-sdk | 82184e2faaa64852b7efbe79f3dbc21e | 23 Feb 2016 |
| DaVinci Resolve 12.2 Studio | davinci-resolve-studio | 749c91b7f77c45cfa154ad2301fcb728 | 23 Dec 2015 |
| Desktop Video 10.5.3 | desktop-video | c76441a8e9fd49ba847a373e8854d024 | 23 Dec 2015 |
| Desktop Video 15.1 | desktop-video | b257d20e85a54327844cfa54ec07ceeb | 22 Sep 2025 |
| Blackmagic Cintel 4.0 | cintel | a3c2299c6e454a92b1f0115dc6f4e24b | 22 Sep 2020 |
| Blackmagic Cintel 4.0 SDK | cintel-sdk | f353a4b2be4a4e999f298c21c45371ee | 22 Sep 2020 |
| DaVinci Resolve 17.4 | davinci-resolve | fc2fa4f752634191a7374cf73ad6cdb9 | 22 Oct 2021 |
| DaVinci Resolve Studio 17.4 | davinci-resolve-studio | 4398ede551a4413db8038a38bdefffcb | 22 Oct 2021 |
| Fusion Studio 17.4 | fusion-studio | 8dcce528010246569f9c608be39c9625 | 22 Oct 2021 |
| Desktop Video 10.2.3 | desktop-video | 2b24bbfaeaef48dc83631a4ddd2ea846 | 22 Oct 2014 |
| Desktop Video 14.0 | desktop-video | 7862bff091734e11b7dbcc626b056895 | 22 May 2024 |
| Desktop Video 14.0 SDK | desktop-video-sdk | 3b7958a069be4705abed586abf06b4b4 | 22 May 2024 |
| Blackmagic RAW 3.1 | braw-sdk | 04114befa5c046138bd7c36e0a333214 | 22 May 2023 |
| DaVinci Resolve 21.0.3 | davinci-resolve | a77754710e824036a6d77cd344df1be1 | 22 Jul 2026 |
| DaVinci Resolve Studio 21.0.3 | davinci-resolve-studio | 60c57e20c37d488882dfea5b8d15355a | 22 Jul 2026 |
| Fusion Studio 21.0.3 | fusion-studio | 87ea6e8b53a4441d8f882e76b5b19c55 | 22 Jul 2026 |
| Desktop Video 14.1 | desktop-video | 0f544a89ce204df6818079a2f18c76a7 | 22 Jul 2024 |
| Desktop Video 14.1 SDK | desktop-video-sdk | 0456797562de450f843aa86f633e1efc | 22 Jul 2024 |
| DaVinci Resolve 18.1.2 | davinci-resolve | 4755b7bd2d924c0db1980824edb84a20 | 22 Dec 2022 |
| DaVinci Resolve Studio 18.1.2 | davinci-resolve-studio | 1ea37eaee99740fe8a063783f4afe636 | 22 Dec 2022 |
| Fusion Studio 18.1.2 | fusion-studio | 931eab2387ea48ed8cdec3bea3d923ba | 22 Dec 2022 |
| DaVinci Resolve 19.0 | davinci-resolve | eeae77279b39447b85d0735c1e09ee39 | 22 Aug 2024 |
| DaVinci Resolve Studio 19.0 | davinci-resolve-studio | 83b6ec6ee61049fab84fe60670d23659 | 22 Aug 2024 |
| Fusion Studio 19.0 | fusion-studio | e745d19b18114f19944d6e9258e33b87 | 22 Aug 2024 |
| DaVinci Resolve 19.1.4 | davinci-resolve | fb0b4bd2f1494207a5452a3c705639e7 | 21 Mar 2025 |
| DaVinci Resolve Studio 19.1.4 | davinci-resolve-studio | 571804036ff14ab18c6a036c41f9bb50 | 21 Mar 2025 |
| Fusion Studio 19.1.4 | fusion-studio | ebffc54a212444b2a6ba17a17253da8d | 21 Mar 2025 |
| Blackmagic RAW 4.5 | braw-sdk | 63750ca5fefc4a49ab16e2b2ffbc822d | 21 Mar 2025 |
| DaVinci Resolve 18.5 | davinci-resolve | 6d977a8a9f384a3a9b3f28f6ca1efedd | 21 Jul 2023 |
| DaVinci Resolve Studio 18.5 | davinci-resolve-studio | 16aad9a497e24871bf3c740fd1ccc1c5 | 21 Jul 2023 |
| Fusion Studio 18.5 | fusion-studio | 36c8e296ccc94f2b97de57a59491a24a | 21 Jul 2023 |
| Desktop Video 12.0 | desktop-video | 9e696d8929c44646a381855bf5c24d32 | 21 Jan 2021 |
| Desktop Video 12.0 SDK | desktop-video-sdk | f1506f10ac1d494aa9dc17acc07586c4 | 21 Jan 2021 |
| DaVinci Resolve 17.4.3 | davinci-resolve | 5efad1a052e8471989f662338d5247f1 | 21 Dec 2021 |
| DaVinci Resolve Studio 17.4.3 | davinci-resolve-studio | 9f63a591c6db46bd8dc8dd41aad1daf9 | 21 Dec 2021 |
| Fusion Studio 17.4.3 | fusion-studio | b8a5a93f991f4f9c8325103b3e223ae5 | 21 Dec 2021 |
| DaVinci Resolve 16.2.1 | davinci-resolve | e324ef94398f4808b3b9d8bf1e59ab01 | 21 Apr 2020 |
| DaVinci Resolve Studio 16.2.1 | davinci-resolve-studio | 3e7183e12fee46939e4bddf58e6034a0 | 21 Apr 2020 |
| Fusion Studio 16.2.1 | fusion-studio | a1d619f3a88d41f385cec133dd249f4b | 21 Apr 2020 |
| Desktop Video 10.3.1 | desktop-video | 39ed6e7bd48e4458aba6430629f2e057 | 20 Nov 2014 |
| Desktop Video 10.3.1 SDK | desktop-video-sdk | 03dcca658e454de9bb8608240a73ec3d | 20 Nov 2014 |
| Desktop Video 12.1 | desktop-video | 114f976c4d3642168d24344d5f5b2afc | 20 May 2021 |
| Desktop Video 12.1 SDK | desktop-video-sdk | a5f3e4d7c4324bad9845c8ffb8d15e3a | 20 May 2021 |
| DaVinci Resolve 18.6.6 | davinci-resolve | dfd43085ef224766b06b579ce8a6d097 | 20 Mar 2024 |
| DaVinci Resolve Studio 18.6.6 | davinci-resolve-studio | 0978e9d6e191491da9f4e6eeeb722351 | 20 Mar 2024 |
| Fusion Studio 18.6.6 | fusion-studio | 86f8812c33e84298907c3d7c9ec5b7d8 | 20 Mar 2024 |
| DaVinci Resolve 18.0 | davinci-resolve | d363098ad3fb48e2b9dc6649d833d15d | 20 Jul 2022 |
| DaVinci Resolve Studio 18.0 | davinci-resolve-studio | c9cd2e8f92cb439594b0e783980c78c9 | 20 Jul 2022 |
| Fusion Studio 18 | fusion-studio | 49a36d6100904cc883d81906e6260406 | 20 Jul 2022 |
| Desktop Video 10.9.5 | desktop-video | aa49f59ba50648f4a315264583624780 | 20 Jul 2017 |
| Desktop Video 10.9.5 SDK | desktop-video-sdk | 627a3b1bcfdb4996a2bcc0d70af9ee3b | 20 Jul 2017 |
| Desktop Video 10.9.4 | desktop-video | 78b43d226c2446cd8e6708a9317b9f0d | 20 Jul 2017 |
| DaVinci Resolve 19.1.3 | davinci-resolve | b0751707709343a587cc762cfe17e4eb | 20 Jan 2025 |
| DaVinci Resolve Studio 19.1.3 | davinci-resolve-studio | 3d8a26e0f5034ae4b353bcb30d5984c3 | 20 Jan 2025 |
| Fusion Studio 19.1.3 | fusion-studio | 21a5a01960fd4358b96cb4226c67b06f | 20 Jan 2025 |
| DaVinci Resolve 17.3 | davinci-resolve | 2fa2973a5599444583a6ec12febcb92f | 20 Aug 2021 |
| DaVinci Resolve Studio 17.3 | davinci-resolve-studio | dc9601bdba7040f1b4c526f4a904141c | 20 Aug 2021 |
| Fusion Studio 17.3 | fusion-studio | 07e79166ecb142289a5b069f1d4025cd | 20 Aug 2021 |
| DaVinci Resolve 16.2.2 | davinci-resolve | ab771c07b120447a9eb05d4c41b91f8c | 19 May 2020 |
| DaVinci Resolve Studio 16.2.2 | davinci-resolve-studio | a0a03fc81876438185af8ed010a0c216 | 19 May 2020 |
| Desktop Video 10.6.6 | desktop-video | d295b0efe1b04fdcadc29234922d0254 | 19 May 2016 |
| Desktop Video 10.6.6 SDK | desktop-video-sdk | ba2e6172603844e1a79500605ed50774 | 19 May 2016 |
| Desktop Video 14.4 | desktop-video | 192f4a67df694dc2a8be7845eceee695 | 19 Dec 2024 |
| Desktop Video 14.4 SDK | desktop-video-sdk | fe7c7ca6891a495f831d8bdefc2d7112 | 19 Dec 2024 |
| Blackmagic RAW 3.6.1 | braw-sdk | c8fc4f0842b441b793bdbc313b68de32 | 19 Dec 2023 |
| Desktop Video 14.2.1 | desktop-video | de70ac89064045368ae353935078e818 | 18 Sep 2024 |
| DaVinci Resolve 15.1 Studio | davinci-resolve-studio | 48954caccd5448dab50e6ffa3d8662f1 | 18 Sep 2018 |
| DaVinci Resolve 15.1 | davinci-resolve | 600f8cd7f97d47789cffb3a6ddf62e04 | 18 Sep 2018 |
| Blackmagic Fusion 9.0.1 Studio | fusion-studio | eae009d57a454b9c9d84dd9e1501d730 | 18 Sep 2017 |
| Blackmagic Fusion 9.0.1 | fusion | b9834531b4554e4684e0a7810a1df6c1 | 18 Sep 2017 |
| Blackmagic RAW 1.5.1 | braw-sdk | cb9fb0c6c42545f1a6d7ab8c39cac6a0 | 18 Oct 2019 |
| DaVinci Resolve 16.1 | davinci-resolve | bc13b7714b9b4b16bc134d8bcfecc894 | 18 Oct 2019 |
| DaVinci Resolve Studio 16.1 | davinci-resolve-studio | ab713d8b44e24b468f4c51c5c62a0f5c | 18 Oct 2019 |
| Fusion Studio 16.1 | fusion-studio | 3b24fc7b28cd4efdb6297bd1e4922502 | 18 Oct 2019 |
| Desktop Video 10.5.1 SDK | desktop-video-sdk | dba8ed8ef8b84862b18e5fe8f3f1b7db | 18 Nov 2015 |
| Blackmagic RAW 2.4 | braw-sdk | 0b1cc7664eff4b5f9c4ec9e245bd7543 | 18 May 2022 |
| Blackmagic RAW SDK 1.2 | braw-sdk | 25186433d6f64d3c9932a397e90b9313 | 18 Mar 2019 |
| DaVinci Resolve 12.3.2 Studio | davinci-resolve-studio | 9507174768844621b9de090bae4ec3ec | 18 Mar 2016 |
| DaVinci Resolve 16.2.3 | davinci-resolve | 6d0dfe07ce934056b6f471b217dbe4ba | 18 Jun 2020 |
| DaVinci Resolve Studio 16.2.3 | davinci-resolve-studio | fd660344018a48ecb2a2079615a6141d | 18 Jun 2020 |
| Fusion Studio 16.2.3 | fusion-studio | 049d8bf0af9e4383ba9baa75c3447e99 | 18 Jun 2020 |
| DaVinci Resolve 19.1.2 | davinci-resolve | fa542e92839b45d0a6672e5bf1c09edb | 18 Dec 2024 |
| DaVinci Resolve Studio 19.1.2 | davinci-resolve-studio | 08788205ebaa4dbe9ed486cba4a01ea4 | 18 Dec 2024 |
| Fusion Studio 19.1.2 | fusion-studio | 3643b0cb34254970adfa1427d50016ae | 18 Dec 2024 |
| Desktop Video 10.6.4 | desktop-video | 8a5205e903b44a39bf7d251a85438532 | 18 Apr 2016 |
| Desktop Video 10.6.4 SDK | desktop-video-sdk | a1978a57b7a847a0998bceed8665d457 | 18 Apr 2016 |
| DaVinci Resolve 16.2.7 | davinci-resolve | 9b15a70f5ce1418686be3479612f1134 | 17 Sep 2020 |
| DaVinci Resolve Studio 16.2.7 | davinci-resolve-studio | 1795da3bdb894d58b875156049110415 | 17 Sep 2020 |
| DaVinci Resolve 19.0.3 | davinci-resolve | ee1da4f13df74d72b6da783ead2ed875 | 17 Oct 2024 |
| DaVinci Resolve Studio 19.0.3 | davinci-resolve-studio | 86463718c6d1491d8d95f8b49f75c4db | 17 Oct 2024 |
| Fusion Studio 19.0.3 | fusion-studio | 181b8df3677441d78e3adff67778c67d | 17 Oct 2024 |
| Blackmagic RAW 2.8 | braw-sdk | 339fd4f74eaa4a45a3c996b7c4fd5552 | 17 Nov 2022 |
| Desktop Video 12.4.1 | desktop-video | 14db0fdb95d54778928cec6711ec543e | 17 Nov 2022 |
| Desktop Video 12.4.1 SDK | desktop-video-sdk | 5cfbdc62bf5a46b7881b7d7042db6c36 | 17 Nov 2022 |
| DaVinci Resolve 17.4.2 | davinci-resolve | 05e306bd033143f4980d9cddff68065d | 17 Nov 2021 |
| DaVinci Resolve Studio 17.4.2 | davinci-resolve-studio | 47c7d014c2a74c1a85ebc85369a60d81 | 17 Nov 2021 |
| Fusion Studio 17.4.2 | fusion-studio | b97f59bfbd8f490a8190a6dafef16b0d | 17 Nov 2021 |
| Desktop Video 11.6 | desktop-video | 6b9e675965fc4c3b9ece9e040dff5358 | 17 Jul 2020 |
| Desktop Video 11.6 SDK | desktop-video-sdk | 761a49d851a24c20bb54f6ab010a8909 | 17 Jul 2020 |
| Blackmagic Cintel 6.1 | cintel | b1be349b53394540b9788626e2ba2233 | 17 Feb 2025 |
| Blackmagic Cintel 6.1 SDK | cintel-sdk | c27c142c580c46aa96ae5c5adc9ce6e4 | 17 Feb 2025 |
| Desktop Video 11.5 | desktop-video | 9a205dd8b075460b8a021c519258d6cd | 17 Feb 2020 |
| Desktop Video 11.5 SDK | desktop-video-sdk | daf30cc6513c4f4a94af71204b2b6778 | 17 Feb 2020 |
| DaVinci Resolve 20.3.1 | davinci-resolve | 646adfc281734638a1d664c9329880c7 | 17 Dec 2025 |
| DaVinci Resolve Studio 20.3.1 | davinci-resolve-studio | ce83cc9a62e94ea780c0b556fed27abe | 17 Dec 2025 |
| Fusion Studio 20.3.1 | fusion-studio | 8ed7518ad2474fd09aac89d1314b8f2c | 17 Dec 2025 |
| DaVinci Resolve 16.1.2 | davinci-resolve | cedbb0f8eb57449fb1d0096426836e63 | 17 Dec 2019 |
| DaVinci Resolve Studio 16.1.2 | davinci-resolve-studio | 6751f0720408486b8499e9b5695534b2 | 17 Dec 2019 |
| Blackmagic RAW 3.0 | braw-sdk | 553241db47fd46d0a9c19665337eefae | 17 Apr 2023 |
| DaVinci Resolve 15.1.2 Studio | davinci-resolve-studio | 4960b81a5f174ab69df39717cd3f3a8d | 16 Oct 2018 |
| DaVinci Resolve 15.1.2 | davinci-resolve | 3e92dc9239a943cdb33cd8e2ece48976 | 16 Oct 2018 |
| Blackmagic RAW 2.2.1 | braw-sdk | 86e644d5443a4d719b5950b94346a94d | 16 Nov 2021 |
| Desktop Video 10.8.3 | desktop-video | d14103ddd967408297e55d2721afab2f | 16 Nov 2016 |
| Desktop Video 10.8.3 SDK | desktop-video-sdk | be774c1dc7394d77ba1c9156f698628b | 16 Nov 2016 |
| Desktop Video 10.10 | desktop-video | 046b297aa3a844fa8fc46d6c32241dbd | 16 May 2018 |
| Desktop Video 10.10 SDK | desktop-video-sdk | aee99f987e694869aff00fbd268fd069 | 16 May 2018 |
| DaVinci Resolve 12.5.6 Studio | davinci-resolve-studio | e48b40aeaddf414fbf556b8cce8f3d6c | 16 Jun 2017 |
| DaVinci Resolve 12.5.6 | davinci-resolve | 9d76f58a0a254b91b457b237fcd21ceb | 16 Jun 2017 |
| Blackmagic Cintel 2.2 | cintel | 17123d22f0d64c65ae2b195312cc6aa6 | 16 Jul 2018 |
| Desktop Video 10.11.1 | desktop-video | 667ef3f6d7564c308f7e2049c1836632 | 16 Jul 2018 |
| Desktop Video 10.11.1 SDK | desktop-video-sdk | d05a346c5fd34a629a38c9eef7c832ef | 16 Jul 2018 |
| Desktop Video 12.8.1 | desktop-video | 0636d85539fd4446a24f5952223cc1ec | 16 Feb 2024 |
| DaVinci Resolve 17.4.4 | davinci-resolve | 8bf9dbe8bfd94b088a86647fb2888384 | 16 Feb 2022 |
| DaVinci Resolve Studio 17.4.4 | davinci-resolve-studio | 0b5a9068349e415ebefb19e29d369149 | 16 Feb 2022 |
| Fusion Studio 17.4.4 | fusion-studio | 06faaa2e472144c88e37f16f0438f887 | 16 Feb 2022 |
| Blackmagic RAW 2.3 | braw-sdk | f2e306aca5fa4a4588e30f03cc29c326 | 16 Feb 2022 |
| Desktop Video 10.3.7 | desktop-video | 0f3296ef0e904de381abedd71879b6a8 | 16 Feb 2015 |
| Blackmagic RAW 1.6.1 | braw-sdk | 421b73d5433a49d2bdbbcc6206d80962 | 16 Dec 2019 |
| DaVinci Resolve 18.6 | davinci-resolve | cebf954f05a74eaeae6b6b14bcca7971 | 15 Sep 2023 |
| DaVinci Resolve Studio 18.6 | davinci-resolve-studio | 2cdeb3d6ccfb4e65add749acb36e659b | 15 Sep 2023 |
| Fusion Studio 18.6 | fusion-studio | a3967899e2ef49d5ab1367ae6c0e8bdd | 15 Sep 2023 |
| Blackmagic RAW 3.4 | braw-sdk | 43b60f54db324712a898333b1799b2d0 | 15 Sep 2023 |
| DaVinci Resolve 18.0.3 | davinci-resolve | 3573fb7e028a4103a823b54c784efb4e | 15 Sep 2022 |
| DaVinci Resolve Studio 18.0.3 | davinci-resolve-studio | 23d948bf9418457693bf61b58fa8a3c7 | 15 Sep 2022 |
| DaVinci Resolve 20.2.2 | davinci-resolve | 6e7fd786285b42aa87853343e8d22fba | 15 Oct 2025 |
| DaVinci Resolve Studio 20.2.2 | davinci-resolve-studio | 8bb647adca65489fa74b841e74f9ddb9 | 15 Oct 2025 |
| Fusion Studio 20.2.2 | fusion-studio | b33183d7b88946569416ec271b36b175 | 15 Oct 2025 |
| Blackmagic RAW 4.3 | braw-sdk | 625c2f523f1b4872881087ef8b0eefb1 | 15 Oct 2024 |
| DaVinci Resolve 11.3.1 | davinci-resolve-studio | a74d7b94688c442784f48c7bd609e03c | 15 May 2015 |
| Blackmagic RAW 3.2 | braw-sdk | 8161588750c94ee4a0de5b1f6202ae6b | 15 Jun 2023 |
| DaVinci Resolve 16.2.4 | davinci-resolve | 4eb7235fb8be4bcea0b18199f1513833 | 15 Jul 2020 |
| DaVinci Resolve Studio 16.2.4 | davinci-resolve-studio | deabd0693c7b44d2a829cb0229a50c61 | 15 Jul 2020 |
| Fusion Studio 16.2.4 | fusion-studio | e3b1a7ccaa42471eb58489265d746b9c | 15 Jul 2020 |
| Blackmagic RAW 1.8 | braw-sdk | 053787b5db7341969906f2d6a5e9a622 | 15 Jul 2020 |
| Desktop Video 10.9.10 | desktop-video | 86f9afa3ef88483eb14c922bd012b1aa | 15 Jan 2018 |
| Desktop Video 10.9.10 SDK | desktop-video-sdk | 9db40f9eb8734f03aa9406e1c178f67d | 15 Jan 2018 |
| Desktop Video 10.9.11 | desktop-video | dd00be4e62b64bd68567d33b3e5c6606 | 15 Feb 2018 |
| Desktop Video 10.9.11 SDK | desktop-video-sdk | 0f513cf7525e4e9f895a19fd62828c0f | 15 Feb 2018 |
| Desktop Video 15.3.1 | desktop-video | d7f4da1c87bb4b2dac8b8c708e84ed42 | 15 Dec 2025 |
| DaVinci Resolve 14.2 Studio | davinci-resolve-studio | ba69aa6b99ab4d51a0408f0b28ee2bf1 | 15 Dec 2017 |
| DaVinci Resolve 14.2 | davinci-resolve | 735097eaff5a48918aee10a450b96bf0 | 15 Dec 2017 |
| Blackmagic Fusion 8.2.1 Studio | fusion-studio | 2b0532e8338046d3b7a92e35b199351c | 15 Dec 2016 |
| Blackmagic Fusion 8.2.1 | fusion | 2fba8c5b63a5449f8b0a5bef6c2fa5d1 | 15 Dec 2016 |
| Blackmagic RAW 6.0 | braw-sdk | e18bcd1a00e049958c162782128f50ff | 14 Sep 2026 |
| Desktop Video 12.6 | desktop-video | 7ee9401cf3104ccdab43b4ccb64e922d | 14 Sep 2023 |
| Desktop Video 12.6 SDK | desktop-video-sdk | de32b6d254784eb78f40b8d083bcb601 | 14 Sep 2023 |
| DaVinci Resolve 18.6.3 | davinci-resolve | 5e61e3f70f7f4d11870586669cdf4d0f | 14 Nov 2023 |
| DaVinci Resolve Studio 18.6.3 | davinci-resolve-studio | f7c543c2f3824a3fb862b76fc7eaa977 | 14 Nov 2023 |
| Fusion Studio 18.6.3 | fusion-studio | ce4e985489b14cdf8decf0829878abee | 14 Nov 2023 |
| DaVinci Resolve 15.2 Studio | davinci-resolve-studio | ca48226e8d584e1d8b491541069f04c5 | 14 Nov 2018 |
| DaVinci Resolve 15.2 | davinci-resolve | 0379b1d486514f0c88da3d779d962719 | 14 Nov 2018 |
| Blackmagic RAW SDK 1.1 | braw-sdk | 3dc847b548c2411da8435f42075be879 | 14 Nov 2018 |
| Desktop Video 10.11 | desktop-video | 4ccd43d59fb54378b65385cba8abbc30 | 14 Jun 2018 |
| Desktop Video 10.11 SDK | desktop-video-sdk | a9e081e6b3df4cf1834027c7e3b8f0df | 14 Jun 2018 |
| Desktop Video 12.4 | desktop-video | bc560d799d5a4ecb87e88c4c0add9ca7 | 14 Jul 2022 |
| Desktop Video 12.4 SDK | desktop-video-sdk | 5d8eaacc274b409dbbee4b9d7724d073 | 14 Jul 2022 |
| Desktop Video 10.4.2 | desktop-video | c5175aa437104765835a1f8ee40487b1 | 14 Jul 2015 |
| DaVinci Resolve 12.5.4 Studio | davinci-resolve-studio | 896f0a98dc5d4cb9be09378e2f3ce8ea | 14 Dec 2016 |
| DaVinci Resolve 18.5.1 | davinci-resolve | defc1c6789b7475b9ee4a42daf9ba61d | 14 Aug 2023 |
| DaVinci Resolve Studio 18.5.1 | davinci-resolve-studio | afb1acaebabb434a9779b4fa8e69399e | 14 Aug 2023 |
| Fusion Studio 18.5.1 | fusion-studio | c1ab151c1811488b9fb6568337d3f57a | 14 Aug 2023 |
| Desktop Video 10.1.4 | desktop-video | 8242b8d9a10e434a8506411eb2feb9c4 | 14 Aug 2014 |
| Desktop Video 10.1.4 SDK | desktop-video-sdk | 22c000795d7e49b4b69942b4c14cf411 | 14 Aug 2014 |
| Blackmagic RAW 1.5 | braw-sdk | 95b653c0e61f449a8332388c40dc25e9 | 13 Sep 2019 |
| Blackmagic Fusion 8.2 Studio | fusion-studio | dc6305fe9f714619ba801560eabbdf42 | 13 Sep 2016 |
| Blackmagic Fusion 8.2 | fusion | 726db1b6b1b14f69aca6e1f7ec95ef92 | 13 Sep 2016 |
| Desktop Video 10.2.2 | desktop-video | b0285c2c62ba46b59f27d4b2762b92de | 13 Oct 2014 |
| Desktop Video 11.7 | desktop-video | a129758034e44f678b1fd630d40e0770 | 13 Nov 2020 |
| Desktop Video 11.7 SDK | desktop-video-sdk | f7ac397e58db442d9f502f9bd49e1162 | 13 Nov 2020 |
| DaVinci Resolve 14.1 Studio | davinci-resolve-studio | f43c0f2f426543d9bdb91b0a10de1b63 | 13 Nov 2017 |
| DaVinci Resolve 14.1 | davinci-resolve | a63dbf8477b744fab80792b7c62e53b5 | 13 Nov 2017 |
| Desktop Video 10.3 | desktop-video | a21a8fb2845c4ff79e17ad38e99d4821 | 13 Nov 2014 |
| Desktop Video 10.3 SDK | desktop-video-sdk | 2d36f8affd4a4966b2f3e07ffeaba38e | 13 Nov 2014 |
| DaVinci Resolve 20.3.3 | davinci-resolve | 01cef5cca71c43a38ec7743085f44d1e | 13 May 2026 |
| DaVinci Resolve Studio 20.3.3 | davinci-resolve-studio | ad27248499354f5aa01ad1e940bfbf46 | 13 May 2026 |
| DaVinci Resolve 15.2.2 Studio | davinci-resolve-studio | 4dab00caeb3e469b8830012a61ccb5ba | 13 Dec 2018 |
| DaVinci Resolve 15.2.2 | davinci-resolve | fc66f58a2af64b079443bf401acac89e | 13 Dec 2018 |
| Blackmagic Cintel 3.0 | cintel | c806599e603443e1ac9b3cb2142d4c50 | 13 Dec 2018 |
| Blackmagic Cintel 3.0 SDK | cintel-sdk | 3190d641f03245d7bcb10123fed66479 | 13 Dec 2018 |
| Desktop Video 9.6.9 | desktop-video | b24b56270c79405fad995f6917424b45 | 13 Dec 2012 |
| DaVinci Resolve 15 Studio | davinci-resolve-studio | 2ae7bf479a3743d3918d82878f92c039 | 13 Aug 2018 |
| DaVinci Resolve 15 | davinci-resolve | e5404c7c8790463a9a2b3d8871d80fdb | 13 Aug 2018 |
| Desktop Video 10.8.6 | desktop-video | 16a91518cfe84cc0a00b3ef1a5719ef3 | 13 Apr 2017 |
| Desktop Video 10.8.6 SDK | desktop-video-sdk | 0eba1b0f7ffc437a8fdbab6b5e32cd4a | 13 Apr 2017 |
| Desktop Video 10.9.7 | desktop-video | c4549243d0e544669323c31de099a45f | 12 Sep 2017 |
| DaVinci Resolve 19.1 | davinci-resolve | e47ee00ccb314e8e9f561b807290be35 | 12 Nov 2024 |
| DaVinci Resolve Studio 19.1 | davinci-resolve-studio | a6e75a45a8fb4abdace3509705007484 | 12 Nov 2024 |
| Fusion Studio 19.1 | fusion-studio | a660ad434106466d9aa9e9a8235ee536 | 12 Nov 2024 |
| DaVinci Resolve 12.1 Studio | davinci-resolve-studio | c0083878311849729db5b2bf574a087a | 12 Nov 2015 |
| DaVinci Resolve 17.2 | davinci-resolve | 7cb792771ce34cf798ac4d5940cba080 | 12 May 2021 |
| DaVinci Resolve Studio 17.2 | davinci-resolve-studio | 5542c36be87249909843d0df7ca14e06 | 12 May 2021 |
| Fusion Studio 17.2 | fusion-studio | 81ada1d7e9314107a76f94e26888dc62 | 12 May 2021 |
| DaVinci Resolve 20.3.2 | davinci-resolve | 91ea14e59bb840339b8fb2e4f919c78b | 12 Feb 2026 |
| DaVinci Resolve Studio 20.3.2 | davinci-resolve-studio | 19d7b1f6daf94f7f8203c68f17237b30 | 12 Feb 2026 |
| Fusion Studio 20.3.2 | fusion-studio | 7a2d1e9849bb4a32a15ae3633cbd14ea | 12 Feb 2026 |
| DaVinci Resolve 15.2.4 Studio | davinci-resolve-studio | f1a216058e6644009536febb73271253 | 12 Feb 2019 |
| DaVinci Resolve 15.2.4 | davinci-resolve | ee6bdc42cba44806b1ab37bcdd1e2e64 | 12 Feb 2019 |
| DaVinci Resolve 11.2.1 | davinci-resolve-studio | 48a61463f4c74604919d17f742fd589d | 12 Feb 2015 |
| Desktop Video 16.3 | desktop-video | efd6b229947d43828a891ab267da9c3b | 12 Aug 2026 |
| Desktop Video 12.9 | desktop-video | 495ebc707969447598c2f1cf0ff8d7d8 | 12 Apr 2024 |
| Desktop Video 12.9 SDK | desktop-video-sdk | cfc892228821453d880022c576fae5fb | 12 Apr 2024 |
| Blackmagic RAW 4.0 | braw-sdk | 6e494d7247a649e587d767cbdd2f8d72 | 12 Apr 2024 |
| DaVinci Resolve 12.0 Studio | davinci-resolve-studio | 1afd118e7ce747288f958257dbcd26f9 | 11 Sep 2015 |
| DaVinci Resolve 18.1 | davinci-resolve | 9c9bbba7b1ee46688489d8f36a04b723 | 11 Nov 2022 |
| DaVinci Resolve Studio 18.1 | davinci-resolve-studio | 197ebc9010bc41b59b9b77dc6dacdd97 | 11 Nov 2022 |
| Fusion Studio 18.1 | fusion-studio | 3871c2d117cf49949a9eb903ffdd69be | 11 Nov 2022 |
| Blackmagic RAW 4.6 | braw-sdk | b623e6fd49294bd0b9cfd854ff809c11 | 11 Jun 2025 |
| Blackmagic RAW 3.3 | braw-sdk | 58a15c91299c440898ae659b982ff134 | 11 Jul 2023 |
| Desktop Video 12.3.1 | desktop-video | 2554875dbf0740c286098fd7a97e1541 | 11 Jul 2022 |
| Desktop Video 12.3.1 SDK | desktop-video-sdk | 5bba45caefdf4e2abb86bde980a1921d | 11 Jul 2022 |
| DaVinci Resolve 12.5.1 Studio | davinci-resolve-studio | b209859ccdc94365ab1a400bcd939835 | 11 Aug 2016 |
| Desktop Video 10.4.3 | desktop-video | c868220859d643d39212a7e78b0120dc | 11 Aug 2015 |
| Desktop Video 10.4.3 SDK | desktop-video-sdk | 890af0dc0fec45a29c2f1e150cafe320 | 11 Aug 2015 |
| DaVinci Resolve 20.2 | davinci-resolve | 2a6285afa62a422ab00e0a2117ebdf3c | 10 Sep 2025 |
| DaVinci Resolve Studio 20.2 | davinci-resolve-studio | 8f5be27631414e56a09e4a6c3e0172d6 | 10 Sep 2025 |
| Fusion Studio 20.2 | fusion-studio | 027117706431493ca5aef2e14341e89f | 10 Sep 2025 |
| Desktop Video 11.4 | desktop-video | 14b0524fb0dc4115b89847530192f9f8 | 10 Sep 2019 |
| Desktop Video 11.4 SDK | desktop-video-sdk | a0340cfa110a4a9ebe2f0730ff30abcc | 10 Sep 2019 |
| DaVinci Resolve 18.6.2 | davinci-resolve | 05f2ae9b4ff34914b23542498ed70de7 | 10 Oct 2023 |
| DaVinci Resolve Studio 18.6.2 | davinci-resolve-studio | f25cb64ded254e8682b287ed1135c557 | 10 Oct 2023 |
| DaVinci Resolve 17.1 | davinci-resolve | e6fb8c64b09a41e0ae4d96e5d88efb8c | 10 Mar 2021 |
| DaVinci Resolve Studio 17.1 | davinci-resolve-studio | 7f222c5ab73542648b2932ea4248141b | 10 Mar 2021 |
| Fusion Studio 17.1 | fusion-studio | 3ce2ff9452154a2da7ba9125865b0a84 | 10 Mar 2021 |
| Desktop Video 10.6.8 | desktop-video | 60c0e3faeb434316b90e5a63918cb8b7 | 10 Jun 2016 |
| Desktop Video 10.4.1 | desktop-video | bf09c832a92649e6b21d68a5d2e65460 | 10 Jun 2015 |
| Desktop Video 10.4.1 SDK | desktop-video-sdk | 78b9d0af24ab44ebb2ab392a56f1f096 | 10 Jun 2015 |
| Blackmagic RAW 4.6.1 | braw-sdk | 5fb8c2c77ee64958ac5a08dda274e9fd | 10 Jul 2025 |
| DaVinci Resolve 12.3.1 Studio | davinci-resolve-studio | 5df3743cddcf4ed98c8d1622b5181db5 | 10 Feb 2016 |
| Desktop Video 12.2.2 | desktop-video | d58607f0c29d4e17a6813f5d06ab2959 | 10 Dec 2021 |
| Desktop Video 12.2.2 SDK | desktop-video-sdk | b17880a8bd654989a1641b0179c2db2f | 10 Dec 2021 |
| DaVinci Resolve 11.1.3 | davinci-resolve-studio | d83b786d56724e8993693869ce080e88 | 10 Dec 2014 |
| Desktop Video 10.8 | desktop-video | 73a8a96378d2459182251738aa516d63 | 09 Sep 2016 |
| Desktop Video 10.8 SDK | desktop-video-sdk | 144240922af449e8904bba31a4dc868b | 09 Sep 2016 |
| DaVinci Resolve 12.5.2 Studio | davinci-resolve-studio | 3d9ad39afac4408fbb64de3b4f179805 | 09 Sep 2016 |
| Desktop Video 9.6.4 | desktop-video | 4a2789a7091f441296a553203ce92589 | 09 Sep 2012 |
| DaVinci Resolve 10.1.5 | davinci-resolve-studio | 667fa16f925a41699b3dff66e83f2013 | 09 May 2014 |
| DaVinci Resolve 16 | davinci-resolve | 2a60296a8e994c5c8deefdd3cb8a6045 | 09 Aug 2019 |
| DaVinci Resolve Studio 16 | davinci-resolve-studio | db6d5e915f3a4b8aa200831bb283d9dc | 09 Aug 2019 |
| Fusion Studio 16 | fusion-studio | d12056c461714eebbec15ac2fa580fc8 | 09 Aug 2019 |
| Blackmagic RAW SDK 1.4 | braw-sdk | e05abbd527dd4179a4d75d859edd1cc8 | 09 Aug 2019 |
| Blackmagic RAW Player 1.4 | braw-player | c96639f1e3314dc28b97ab4f84892d82 | 09 Aug 2019 |
| Desktop Video 11.3 | desktop-video | 2fdc19fd59ba44cc85c4a55f2b124ce6 | 09 Aug 2019 |
| Desktop Video 11.3 SDK | desktop-video-sdk | 9bfd3eb624fc47b38a985893fcc1c8ac | 09 Aug 2019 |
| Desktop Video 10.9.12 | desktop-video | 2c55849057bb4f80850d8f83d48cc7e0 | 09 Apr 2018 |
| Desktop Video 10.9.12 SDK | desktop-video-sdk | 1a1e9345f88b41069bc9715f612c791b | 09 Apr 2018 |
| DaVinci Resolve 21.1 | davinci-resolve | 9ecf221a3a1f47cca35a4e4c34133865 | 08 Sep 2026 |
| DaVinci Resolve Studio 21.1 | davinci-resolve-studio | b9955d7104aa405bb491d924fd734a14 | 08 Sep 2026 |
| Fusion Studio 21.1 | fusion-studio | c2ad9534bf5746cd8a9db063f1cad9b1 | 08 Sep 2026 |
| DaVinci Resolve 14 Studio | davinci-resolve-studio | c92f6b2262cb4c378f8929e39e51e4ab | 08 Sep 2017 |
| DaVinci Resolve 14 | davinci-resolve | f7604745559f43e79fd146bcf9c341cc | 08 Sep 2017 |
| DaVinci Resolve 17.3.2 | davinci-resolve | e08a3e389a57432b93d7fb1892cd6536 | 08 Oct 2021 |
| DaVinci Resolve Studio 17.3.2 | davinci-resolve-studio | 209c833027434256969d167043ed0a6e | 08 Oct 2021 |
| Fusion Studio 17.3.2 | fusion-studio | 6257004d116e495a9614cf018da72c87 | 08 Oct 2021 |
| Blackmagic RAW 2.2 | braw-sdk | 85a24ac4df5f4424a54cf8a13121c79a | 08 Oct 2021 |
| Desktop Video 10.6.2 | desktop-video | 3f0844bdcfc24fd1a631aeca0d2cefe6 | 08 Mar 2016 |
| DaVinci Resolve 12.5 Studio | davinci-resolve-studio | eacf10ce5d1f48de971d7e57513c8778 | 08 Jun 2016 |
| Desktop Video 10.6.7 | desktop-video | 76fe80ebd9e9490abbde3923c42ea95f | 08 Jun 2016 |
| Desktop Video 16.1 | desktop-video | c5e05b53633c43acadd67deea648f4b3 | 08 Jul 2026 |
| Desktop Video 12.4.2 SDK | desktop-video-sdk | d6403d75bff141d2823d0f68bab96745 | 08 Dec 2022 |
| Desktop Video 15.0 | desktop-video | fb93016534354f8cb9c1da89bf384dde | 08 Aug 2025 |
| Desktop Video 15.0 SDK | desktop-video-sdk | d3a03e1609584ce6bee2f73813cc8a89 | 08 Aug 2025 |
| Desktop Video 16.0 | desktop-video | 2ea094777cde4f38bb9e682c23b89c16 | 08 Apr 2026 |
| Desktop Video 16.0 SDK | desktop-video-sdk | 47789a55dd7f4e848ec5352a4903d4e8 | 08 Apr 2026 |
| DaVinci Resolve 18.0.2 | davinci-resolve | 1ff0bded5ac64aa2b1ac85b94f7cc2cc | 07 Sep 2022 |
| DaVinci Resolve Studio 18.0.2 | davinci-resolve-studio | 9fd907f561904fee9f256a92d7616e62 | 07 Sep 2022 |
| Fusion Studio 18.0.2 | fusion-studio | f49a763c039d4170a2af5f3eb6825bf4 | 07 Sep 2022 |
| Desktop Video 10.8.1 | desktop-video | 0247fca5518644dfb196bbe962d84f65 | 07 Oct 2016 |
| DaVinci Resolve 17.4.5 | davinci-resolve | 6e8a71806e4e4816895777b966a873c5 | 07 Mar 2022 |
| DaVinci Resolve Studio 17.4.5 | davinci-resolve-studio | 4f8bb4869d314eb897e3b711f3522ffa | 07 Mar 2022 |
| Fusion Studio 17.4.5 | fusion-studio | 410b759cff1843fea7bf6441ea3aa380 | 07 Mar 2022 |
| DaVinci Resolve 15.3 Studio | davinci-resolve-studio | f1a397b8f81049cf929ff4a7a92da082 | 07 Mar 2019 |
| DaVinci Resolve 15.3 | davinci-resolve | 554c23bbaf404b8592a9b456df7d8073 | 07 Mar 2019 |
| DaVinci Resolve 18.6.5 | davinci-resolve | 0098df10e28e404682c4945687fcdf8c | 07 Feb 2024 |
| DaVinci Resolve Studio 18.6.5 | davinci-resolve-studio | 9a7b8de9c3e346b2925397cfe2faff2d | 07 Feb 2024 |
| Fusion Studio 18.6.5 | fusion-studio | 328ea33e567a4c7098039e619de7fa85 | 07 Feb 2024 |
| Blackmagic Fairlight Sound Library 1.0 | blackmagic-fairlight-sound-library | 9b98f6489e1b4432891918594e1105e7 | 07 Dec 2020 |
| DaVinci Resolve 20.1 | davinci-resolve | d39098aa3b174c9c91bb91a65f43b3b9 | 07 Aug 2025 |
| DaVinci Resolve Studio 20.1 | davinci-resolve-studio | 2addc9e5b2ab4627a1be8cfbf19d9f2c | 07 Aug 2025 |
| Fusion Studio 20.1 | fusion-studio | 0eca13928874416c9eac6d63e3e79608 | 07 Aug 2025 |
| Blackmagic RAW 5.0 | braw-sdk | d87a8b1fed3a4eba92692724f3d62294 | 07 Aug 2025 |
| Blackmagic Cintel 7.0 | cintel | 47a8313709194261a992668c54692b0e | 07 Aug 2025 |
| Blackmagic Cintel 7.0 SDK | cintel-sdk | 7906f56e7a87449e950c4e6d6f09cf07 | 07 Aug 2025 |
| Desktop Video 14.2 | desktop-video | 552546307a7c4de29ea6d09a6ca08c90 | 07 Aug 2024 |
| Desktop Video 14.2 SDK | desktop-video-sdk | f874ba6ae24e436fba84e3be19d2caff | 07 Aug 2024 |
| DaVinci Resolve 18.6.1 | davinci-resolve | f43e2a99efdf4afd8ce89f2c321ff857 | 06 Oct 2023 |
| DaVinci Resolve Studio 18.6.1 | davinci-resolve-studio | 29fcf4dece3b43669c4737d448f85da8 | 06 Oct 2023 |
| Fusion Studio 18.6.1 | fusion-studio | 54130e3b7f1447b0840e216bc6f45b1a | 06 Oct 2023 |
| DaVinci Resolve 14.0.1 Studio | davinci-resolve-studio | 1c864fa29afe45a390d601217667c67f | 06 Oct 2017 |
| DaVinci Resolve 14.0.1 | davinci-resolve | e865d55e35f64549ba03e507fba578e7 | 06 Oct 2017 |
| Desktop Video 9.8 | desktop-video | a215e189f61747069aae2de426d48a78 | 06 Oct 2013 |
| Desktop Video 16.0.1 | desktop-video | a2bf721e15534d50a936941e5ebe6a7b | 06 May 2026 |
| DaVinci Resolve 18.1.4 | davinci-resolve | 6449dc76e0b845bcb7399964b00a3ec4 | 06 Mar 2023 |
| DaVinci Resolve Studio 18.1.4 | davinci-resolve-studio | bb935a0d0e814961b7a75a2ded811b01 | 06 Mar 2023 |
| DaVinci Resolve 16.2 | davinci-resolve | 864344801a4844cfab341fc73fb4775c | 06 Mar 2020 |
| DaVinci Resolve Studio 16.2 | davinci-resolve-studio | 8c0ce4708141497e9ef2f21feaef000b | 06 Mar 2020 |
| Fusion Studio 16.2 | fusion-studio | 017633fcaceb4909a1ecda1286dd66b2 | 06 Mar 2020 |
| Blackmagic RAW 1.7 | braw-sdk | 5c2a10fb259340a6bc08c889108e9e76 | 06 Mar 2020 |
| Desktop Video 10.1.1 SDK | desktop-video-sdk | 2418fdc39bf54f18845fca6d5d3563a9 | 06 Jun 2014 |
| Desktop Video 10.1.1 | desktop-video | f503965cc5c84f04afafc09facb0547f | 06 Jun 2014 |
| DaVinci Resolve 14.3.1 Studio | davinci-resolve-studio | 9594abeef88c42f59a7550a7176cd51a | 06 Jul 2018 |
| DaVinci Resolve 14.3.1 | davinci-resolve | a9f06b5e94964c5aaa2a184ea401ccbb | 06 Jul 2018 |
| DaVinci Resolve 18.1.3 | davinci-resolve | 44be7e694b4e440db5d2f70ad732d3b2 | 06 Feb 2023 |
| DaVinci Resolve Studio 18.1.3 | davinci-resolve-studio | 748bf78ca717474cbc4fb792ac896e4c | 06 Feb 2023 |
| Fusion Studio 18.1.3 | fusion-studio | 1ed4da77afcd4143bd2df9076a913dc3 | 06 Feb 2023 |
| Desktop Video 10.11.2 | desktop-video | 7682360f6279408f8706c3f3418828b4 | 06 Aug 2018 |
| Desktop Video 10.11.2 SDK | desktop-video-sdk | cabdbaac37bd4d1aabd8d7f9c9a40922 | 06 Aug 2018 |
| Desktop Video 10.2.1 | desktop-video | 41777d25110b40dcb99aa7783a99fee8 | 05 Sep 2014 |
| Desktop Video 11.5.1 | desktop-video | 5f9af2c067674ed98d54bf67dcf7a9b6 | 05 May 2020 |
| Desktop Video 11.5.1 SDK | desktop-video-sdk | b7ea0c6787dc447bbf4544b584027724 | 05 May 2020 |
| Desktop Video 11.0 | desktop-video | 305f424c236d40cf976fec631471ee82 | 05 Mar 2019 |
| Desktop Video 11.0 SDK | desktop-video-sdk | 5d0e22b172f34912861344f3bf6f0e6a | 05 Mar 2019 |
| Desktop Video 12.5.1 SDK | desktop-video-sdk | 16b195b1b9c54c0089aaa3ef0757a457 | 05 Jul 2023 |
| Desktop Video 10.5.4 | desktop-video | 6b81b340601d41f784833db2faa789fd | 05 Jan 2016 |
| Desktop Video 10.5.4 SDK | desktop-video-sdk | ae8beb1441754ff48635b40bd4002cd3 | 05 Jan 2016 |
| DaVinci Resolve 18.6.4 | davinci-resolve | f38fc270c4c040bea4782182d0fbb367 | 05 Dec 2023 |
| DaVinci Resolve Studio 18.6.4 | davinci-resolve-studio | ad4b2831fdea407f91ba6cb6c5d85640 | 05 Dec 2023 |
| Fusion Studio 18.6.4 | fusion-studio | 917600b56f344c238968ff6ae7175140 | 05 Dec 2023 |
| Blackmagic RAW 3.6 | braw-sdk | 3fa4c85761b14c8fabee5089fb47c91a | 05 Dec 2023 |
| DaVinci Resolve 21.0.4 | davinci-resolve | 651bbe286f4c4544b4ded9b343638f60 | 05 Aug 2026 |
| DaVinci Resolve Studio 21.0.4 | davinci-resolve-studio | f6af677f3e3741f59a014b54445bd39e | 05 Aug 2026 |
| Fusion Studio 21.0.4 | fusion-studio | 43fae58795f749e18a64376fe0215e90 | 05 Aug 2026 |
| DaVinci Resolve 11 | davinci-resolve-studio | 93d360c385b442499182cdf9ae57ff4e | 05 Aug 2014 |
| DaVinci Resolve Studio 15.3.1 | davinci-resolve-studio | 6a5d7c97791746c3973975557a38e87e | 05 Apr 2019 |
| DaVinci Resolve 15.3.1 | davinci-resolve | a04ad145058f469b98d4977e5f13b853 | 05 Apr 2019 |
| DaVinci Resolve 19.0.1 | davinci-resolve | 98a118a7bac24d2b919878bb5d3ae045 | 04 Sep 2024 |
| DaVinci Resolve Studio 19.0.1 | davinci-resolve-studio | 38262ec602634b7eb09d4ae9ac72526f | 04 Sep 2024 |
| Fusion Studio 19.0.1 | fusion-studio | fc66e223901a4ee9b8933cf2556eea25 | 04 Sep 2024 |
| Blackmagic RAW 5.1 | braw-sdk | 65b23bbf937c4b7183bbaf37c8922dc1 | 04 Nov 2025 |
| DaVinci Resolve 20.2.3 | davinci-resolve | ae9d4b7046a441f081888c81c7e68068 | 04 Nov 2025 |
| DaVinci Resolve Studio 20.2.3 | davinci-resolve-studio | 23364c8d0a59451294d237bc7f02bdd0 | 04 Nov 2025 |
| Fusion Studio 20.2.3 | fusion-studio | cfdef8bb07c741119be469954b921bbe | 04 Nov 2025 |
| DeckLink 7.9.4 | desktop-video | 1c896748578b4664889f37f2e6ba73ee | 04 Jan 2011 |
| DaVinci Resolve 11.2 | davinci-resolve-studio | 360c3008052e43879802eda8a7c14fa3 | 04 Feb 2015 |
| DaVinci Resolve 11.1.2 | davinci-resolve-studio | 3f54aadfc03b4a74a73c058bde889a7a | 04 Dec 2014 |
| Desktop Video 10.3.2 | desktop-video | f6cbb04ac3c8444a9fb70c0e5c0697ef | 04 Dec 2014 |
| DaVinci Resolve 15.0.1 Studio | davinci-resolve-studio | 934d2a8a2a55474ea6ab2e9ac8e73e68 | 03 Sep 2018 |
| DaVinci Resolve 15.0.1 | davinci-resolve | 05318553bd714ffab5daddec9a7c01db | 03 Sep 2018 |
| Blackmagic Cintel 6.0 | cintel | e28bb81a0b364052bf6db3f37234e957 | 03 May 2024 |
| Blackmagic Cintel 6.0 SDK | cintel-sdk | 7d98892d27dc41fda8dccd4574684749 | 03 May 2024 |
| Desktop Video 10.6.5 | desktop-video | 9c4e28d745ef489caad4daca0f1e6435 | 03 May 2016 |
| Desktop Video 10.6.5 SDK | desktop-video-sdk | 2b4316adcffc49ab8c81348f04517f83 | 03 May 2016 |
| DaVinci Resolve 21.0 | davinci-resolve | a8fba3280b864128bf1d47949bfa3e3c | 03 Jun 2026 |
| DaVinci Resolve Studio 21.0 | davinci-resolve-studio | 6dfcd748c3494be3b3e8650af3ec6d36 | 03 Jun 2026 |
| Fusion Studio 21.0 | fusion-studio | 35f6261e5ed04db383498fecce53043a | 03 Jun 2026 |
| Desktop Video 14.0.1 | desktop-video | d73a63095b6a4a189916fb2baa5a0ef3 | 03 Jun 2024 |
| DaVinci Resolve 17.3.1 | davinci-resolve | 5803af60aa4e4d6b9f94c847724cd5da | 02 Sep 2021 |
| DaVinci Resolve Studio 17.3.1 | davinci-resolve-studio | 4b350d4044474e41b46b8b2869a6e14b | 02 Sep 2021 |
| Fusion Studio 17.3.1 | fusion-studio | 9832ea2b8828403b8dd3134a0e290af9 | 02 Sep 2021 |
| Desktop Video 10.5 | desktop-video | 71c010fd975c4e588702de36809cda21 | 02 Sep 2015 |
| Desktop Video 10.5 SDK | desktop-video-sdk | c81976760e274505ab4b631566323aae | 02 Sep 2015 |
| DaVinci Resolve 19.0.2 | davinci-resolve | b18a01b5460b4c6f867bb007e9861543 | 02 Oct 2024 |
| DaVinci Resolve Studio 19.0.2 | davinci-resolve-studio | cbccd0fb44684be3a2dace78a0ad7b20 | 02 Oct 2024 |
| Fusion Studio 19.0.2 | fusion-studio | 4aca47594e8746388053d8e52847384d | 02 Oct 2024 |
| DaVinci Resolve 11.1 | davinci-resolve-studio | 9a7988f37eb540759abcbb944dc14210 | 02 Oct 2014 |
| DaVinci Resolve 12.5.5 Studio | davinci-resolve-studio | 7b03b007846c4131951ba8b0981c99d3 | 02 Mar 2017 |
| DaVinci Resolve 12.5.5 | davinci-resolve | 2e116149deb9464b95e869c3146d5887 | 02 Mar 2017 |
| Desktop Video 10.6.1 | desktop-video | 5f6576b0cdc14a6fb75cb18383413669 | 02 Mar 2016 |
| Desktop Video 10.6.1 SDK | desktop-video-sdk | 6d76ba8da6174e51a3c9e0412e99e37c | 02 Mar 2016 |
| DaVinci Resolve 17.2.1 | davinci-resolve | 3256e3339ca7410bae00f875716512b5 | 02 Jun 2021 |
| DaVinci Resolve Studio 17.2.1 | davinci-resolve-studio | a92bd396fb6f4a23a773a051aeaa4502 | 02 Jun 2021 |
| Fusion Studio 17.2.1 | fusion-studio | 1d10694a8a634394809e66bd48c314e2 | 02 Jun 2021 |
| DaVinci Resolve 21.0.2 | davinci-resolve | 81eb9de9e7f14eca9da1f92376a39d70 | 02 Jul 2026 |
| DaVinci Resolve Studio 21.0.2 | davinci-resolve-studio | ec36996cf2694986b514285fffa37e46 | 02 Jul 2026 |
| Fusion Studio 21.0.2 | fusion-studio | 821c953d6914429f8b4d435d9fb094be | 02 Jul 2026 |
| DaVinci Resolve 14.3 Studio | davinci-resolve-studio | 7b47c2e602994a4bb1e366544b9a774b | 02 Feb 2018 |
| DaVinci Resolve 14.3 | davinci-resolve | 1c3f4026f97349aca0876b809f802142 | 02 Feb 2018 |
| DaVinci Resolve 19.1.1 | davinci-resolve | e451e04cc94d4380ac2a91982e9718e9 | 02 Dec 2024 |
| DaVinci Resolve Studio 19.1.1 | davinci-resolve-studio | 7a7cba986c49429ea53fdc2e690ffc8f | 02 Dec 2024 |
| Fusion Studio 19.1.1 | fusion-studio | 9d2445d8ccf84db798d272f0a0d8c9ba | 02 Dec 2024 |
| Desktop Video 10.4 | desktop-video | 90e843c7b4334875954282f7f3fe7b53 | 02 Apr 2015 |
| Desktop Video 10.4 SDK | desktop-video-sdk | e3d78a1476c747b4aab6fadb77cf5bf2 | 02 Apr 2015 |
| Desktop Video 10.2 | desktop-video | 06bfea91c25345108d035ffd7d6c8204 | 01 Sep 2014 |
| Fusion Studio 16.1.1 | fusion-studio | 6320766eac62440180c5455adce90398 | 01 Nov 2019 |
| DaVinci Resolve 17.2.2 | davinci-resolve | 9cd52af58d774a56b1a751ce0ecf9207 | 01 Jul 2021 |
| DaVinci Resolve Studio 17.2.2 | davinci-resolve-studio | 4490b8e355ad43f783ab8133863b44fd | 01 Jul 2021 |
| Fusion Studio 17.2.2 | fusion-studio | ca3fc605f1b44631a27b59eaac63e059 | 01 Jul 2021 |
| Desktop Video 10.7 | desktop-video | b86bbfa00925443abe6590e9e425421a | 01 Jul 2016 |
| Desktop Video 10.7 SDK | desktop-video-sdk | c863a9eed742485ab727469a8035f023 | 01 Jul 2016 |
| Blackmagic Fusion 9.0 Studio | fusion-studio | ed2770f51c374f75bf3aa10878cce7c6 | 01 Aug 2017 |
| Blackmagic Fusion 9.0 | fusion | aa1baf34b294458a833ace39ce00a850 | 01 Aug 2017 |
| Desktop Video 10.7.1 | desktop-video | 28f2ae1816604300a8c3aaf77abea7cb | 01 Aug 2016 |
| Blackmagic Cintel 4.1 | cintel | 98602bd6ec0946bfb36d450426e312df | 01 Apr 2021 |
| Blackmagic Cintel 4.1 SDK | cintel-sdk | 0d03de572f1048d6a7bde54f342cf9ea | 01 Apr 2021 |
| Desktop Video 11.1 | desktop-video | bdb0c72cc1af4bf2ac790957c8e5db12 | 01 Apr 2019 |
| Desktop Video 11.1 SDK | desktop-video-sdk | e062dd1abd5a4bf6af557a310c70a196 | 01 Apr 2019 |
| Blackmagic RAW SDK 1.3 | braw-sdk | 96a1f4663c9b478bbc6dfdd5b54058a3 | 01 Apr 2019 |
