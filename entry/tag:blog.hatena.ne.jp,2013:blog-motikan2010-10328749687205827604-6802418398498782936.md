<div style="text-align:center;">[f:id:motikan2010:20250703231551p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　**2025年も折り返しを迎えたということで、「2025年上半期 人気脆弱性」の発表です！！** 🎉

今回の順位を見ながら

- 「Next.js」や「Vite」のフロントエンドの技術の脆弱性
- 「生成AIが発見した」 Linux SMB のゼロデイの脆弱性

がランクインしており、時代の節目を見ている感（小並感）

## 2025年 上半期 人気脆弱性 TOP 10 in GitHub

### 1位 CVE-2025-29927 - Next.js の認可バイパス

- <span><a href="https://zenn.dev/t3tra/articles/c293410c7daf63" target="_blank">Next.jsの脆弱性CVE-2025-29927まとめ</a></span>

### 2位 CVE-2025-24813 - Apache Tomcat partial PUT におけるリモートコード実行、情報漏えいや改ざん

- <span><a href="https://www.akamai.com/ja/blog/security-research/march-apache-tomcat-path-equivalence-traffic-detections-mitigations" target="_blank">Apache Tomcat の脆弱性 CVE-2025-24813 の検知と緩和 | Akamai</a></span>
- <span><a href="https://socprime.com/ja/blog/detect-cve-2025-24813-exploitation/" target="_blank">CVE-2025-24813 Detection: Apache Tomcat RCE Vulnerability Actively Exploited in the Wild | SOC Prime</a></span>

### 3位 CVE-2025-24071 - Microsoft Windows ファイルエクスプローラーのスプーフィング

- <span><a href="https://msrc.microsoft.com/update-guide/vulnerability/CVE-2025-24071" target="_blank">CVE-2025-24071 - セキュリティ更新プログラム ガイド - Microsoft - Microsoft Windows ファイル エクスプローラーのスプーフィングの脆弱性</a></span>
- <span><a href="https://iototsecnews.jp/2025/05/29/windows-11-file-explorer-vulnerability-enables-ntlm-hash-theft/" target="_blank">Windows 11 File Explorer の脆弱性 CVE-2025-24071：NTLM Hash 窃取での悪用と PoC のリリース – IoT OT Security News</a></span>

### 4位 CVE-2025-30208 - Vite の任意のファイル読み取り

- <span><a href="https://iototsecnews.jp/2025/03/27/millions-at-risk-poc-exploit-releases-for-vite-arbitrary-file-read-flaw-cve-2025-30208/" target="_blank">Vite の脆弱性 CVE-2025-30208 が FIX：任意のファイル読み取りの PoC も登場 – IoT OT Security News</a></span>

### 5位 CVE-2025-33073 - Windows SMB の特権昇格

- <span><a href="https://socprime.com/ja/blog/cve-2025-33073-zero-day-vulnerability/" target="_blank">CVE-2025-33073: Windows SMBクライアントのゼロデイ脆弱性により攻撃者がSYSTEM権限を取得可能 | SOC Prime</a></span>

### 6位 CVE-2025-1974 - Kubernetes 用の Ingress NGINX Controller の リモートコード実行

- <span><a href="https://kubernetes.io/ja/blog/2025/03/24/ingress-nginx-cve-2025-1974/" target="_blank">Ingress-nginxの脆弱性CVE-2025-1974: 知っておくべきこと | Kubernetes</a></span>
- <span><a href="https://sysdig.jp/blog/detecting-and-mitigating-ingressnightmare-cve-2025-1974/" target="_blank">IngressNightmare（CVE-2025-1974）の検知と緩和 – Sysdig</a></span>

### 7位 CVE-2025-32433 - Erlang/OTP 内の SSH サーバのリモートコード実行

- <span><a href="https://www.cybereason.co.jp/blog/threat-analysis-report/13152/" target="_blank">【脅威分析レポート】CVE-2025-32433 〜Erlang/OTPのSSH実装における脆弱性、悪用されると認証なしでリモートコード実行（RCE）が可能に〜 | BLOG | サイバーリーズン | EDR（次世代エンドポイントセキュリティ）</a></span>
- <span><a href="https://www.security-next.com/169684" target="_blank">【セキュリティ ニュース】「Erlang/OTP」に深刻なRCE脆弱性 - 概念実証コードも公開済み（1ページ目 / 全1ページ）：Security NEXT</a></span>

### 8位 CVE-2025-37899 - 生成AIが発見した Linux SMB のゼロデイ

- <span><a href="https://gigazine.net/news/20250527-o3-find-linux-vulnerability/" target="_blank">OpenAIのo3モデルでLinuxカーネルのゼロデイ脆弱性を発見した方法とは - GIGAZINE</a></span>
- <span><a href="https://iototsecnews.jp/2025/05/22/linux-kernel-zero-day-smb-vulnerability-discovered-via-chatgpt/" target="_blank">Linux SMB のゼロデイ脆弱性 CVE-2025-37899：特定したのは ChatGPT を操る研究者 – IoT OT Security News</a></span>

### 9位 CVE-2025-24203 - iOS カーネルのファイルシステムの保護された部分への改変を許す脆弱性

- <span><a href="https://iototsecnews.jp/2025/05/19/poc-released-ios-kernel-flaw-allows-file-system-modification-without-jailbreak/" target="_blank">iOS カーネルの脆弱性 CVE-2025-24203 の PoC 公開：システムをカスタマイズ without ジェイルブレイク – IoT OT Security News</a></span>

### 10位 CVE-2025-0282 - Ivanti Connect Secure のリモードコード実行

- <span><a href="https://www.jpcert.or.jp/at/2025/at250001.html" target="_blank">Ivanti Connect Secureなどにおける脆弱性（CVE-2025-0282）に関する注意喚起</a></span>

## まとめ

 2025年上半期に期待。

個人的には IIJ に影響を与えた Active! mail の脆弱性が印象に残っています。  
- <span><a href="https://www.iij.ad.jp/news/pressrelease/2025/0422-2.html" target="_blank">IIJセキュアMXサービスにおけるお客様情報の漏えいについてのお詫びとご報告 | IIJについて | IIJ</a></span>
- <span><a href="https://www.jpcert.or.jp/at/2025/at250010.html" target="_blank">Active! mailにおけるスタックベースのバッファオーバーフローの脆弱性に関する注意喚起</a></span>
