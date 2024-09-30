---
layout: post
title: Steganography
subtitle: Làm thế nào để trích xuất thông tin bí mật từ file.
tags: [steganography, cybersecurity, forensics]
comments: true
mathjax: true
author: VuNguyen
---

Trong mật mã học, Steganography là một cách biểu diễn thông tin để khó bị phát hiện bởi con người.
Trong thực tế, phương pháp này thường được dùng để ẩn thông tin bí mật trong một file khi truyền tải. Thông tin bí mật có thể là một file khác hoặc đơn thuần là một file text có chưa nội dung bí mật. Đây có thể là một trong những cách để “đánh lạc hướng” trong các cuộc điều tra kỹ thuật số (Digital Forensics) vì “những cái thực sự thấy chỉ là phần nổi của tảng băng chìm".

### Forensics

*Để dễ hình dung, hãy đi vào xem xét một ví dụ sau:*

Giả sử một nhóm tội phạm mạng đang thực hiện buôn bán bất hợp pháp trên internet nhưng họ muốn bằng một cách nào đó đánh lạc hướng các điều tra viên nên đã “bẻ lái” câu chuyện thành một cuộc thảo luận về UFO. Trong câu chuyện có bình luận những hình ảnh liên quan đến những alien “rất cute” chẳng hạn như:

![image.png](/assets/img/image_steganography.png)


Trông thì có vẻ không có gì bất thường ở đây. Nhưng hãy download ảnh này về và tiến hành phân tích chúng.

Đầu tiên, bản chất của image file là một file có chứa tập dữ liệu được mã hóa. Vì vậy ta hoàn toàn có thể kiểm tra file đó bằng lệnh **cat :**

```bash
┌──(kali㉿kali)-[/tmp]
└─$ cat cutie.png          
�PNG
▒
IHD���PLTE�����������������������������������������������������������������������������������������������������������������������������a���*EB��:����ϲ30p�.(CA��b+FB��8">;&@B&A>9RO =:#<A;8$@=96.)%>A��b��:&AA��:��e��c��]��9��_�`P��b4-HC��Z5NK�#▒��W���!:@t�-�ӵ0KG�����������Ͱ���r�-u�"6SB��?/KB�����6��G��������L��C2OB��ب�Z���CZWn�,Rhc��T9X@>VRMc^�ٻ����ꖞ�P��`H_[z�;��W��9l�,���Wli������j}z��])&��c���^rmGk>t�<Nq;��d��;���cwt������>\G��Z�����Ց����:������Q|?Be>>^>��\!EC��������Ј��u����\Y|M��������QsK}��o�~��X�����vEeHm�<��ƌ��a�=l�*x����mg�=$ ������`�2m�R�è]�>���a�OW�>h�0��/����ôf�PWw5���t�S��������°����PLlJ����������󜥰�i{n�:�"�˼}�|z�)[oa�������줃����I��Vx�T�����Aq�q��������W3D?y�5���}�U�aPAA=�>0�'��x�UF�.%sPHz6/|�nVE?e82��ȅ�ZM�UJ��N��D�*tRNS��
�(���0Θ�?E�LU8��]��eԹsxoj������������IDATx����OSg▒�[
W4/ȠD��󞛾=9=���!�ihK�
        d)��i0�P!�Au��Ĉ��,
                          Y$r���;�[�2�v�s����=0�hO9��e�x���y���<���4��䤢�S��*���X�ɂ��d���%��t��t=�`0f��,/I�q�HI-(./19�ey�Z�8L��U���ƲSE:��Aک,؝���Ru▒     OQv:�+��4�FB�����b-J$0)����r⸖_&*G���s-H$&�F�?J
t▒      GR6�7��
               �Fb������N#�8}D���N#aH.փh1��i)g�'@_��R���c:�D�▒�Th�F"p
|2
`w�}��`�kէ�I����Ͼ���t����ʝ��]t���S���2�@��b��F�v;�0v;mu����>-�L\���������▒h��/�����B��z�;����(éAC{�E�$��zI:0L\��"�P�÷t
                                                                                                                      `(K�i���:W�`H<iw�hN�4�I��K����n��w��C#���H�;���
�uUcF�t▒G�<:Dhs�*���j�J���*ߍ/_���_�~��Y]C�qt�G�V��H�׋��jn��p��U��r�<|�\]%��
                                                                           �Yf�4���t ���K
TU�����r��-2_ݰX�1bU}�kC����;/�����&�X5�1;�z~�"E@�@��NC]���
...
```

Trông có vẻ không khả thi vì toàn bộ thông tin của file đã được mã hóa :D. Lúc này hãy thử lọc ra những ký tự mà chúng ta có thể đọc được bằng lệnh **strings**

```bash
┌──(kali㉿kali)-[/tmp]
└─$ cat cutie.png | strings
...
        ([b!
G9qE
(A}j
^MEjOk
L^ay
"P{J
d: ]
ZT%Q
p7a4u
^[=&
IEND
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
                                                                                                                                                     
┌──(kali㉿kali)-[/tmp]
└─$ 
```

Weu, chúng ta thấy gì này :D, có vè là một file gì đó có tên **To_agentR.txt** được nhúng trong hình ảnh này. File này có thể có một vài thông tin bí mật sẽ hữu ích trong cuộc điều tra của chúng ta.

Vì vậy, hãy thử trích xuất file này khỏi file ảnh ban đầu.

**Kiểm tra các file binaries khác có thể được nhúng trong file:**

```bash
┌──(kali㉿kali)-[/tmp]
└─$ binwalk cutie.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

```

Kết quả kiểm tra cho thấy có một tệp Zip được nhúng, và file được zip có tên **To_agentR.txt.** Vì vậy phán đoán ban đầu của chúng ta đã đúng.

**Thực hiện trích xuất toàn bộ dữ liệu của file**

```bash
──(kali㉿kali)-[/tmp]
└─$ binwalk -e cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

                                                                                                                                                     
┌──(kali㉿kali)-[/tmp]
└─$ ls -la _cutie.png.extracted 
total 316
drwxrwxr-x  2 kali kali    100 Sep 30 04:18 .
drwxrwxrwt 18 root root    460 Sep 30 04:18 ..
-rw-rw-r--  1 kali kali 279312 Sep 30 04:18 365
-rw-rw-r--  1 kali kali  33973 Sep 30 04:18 365.zlib
-rw-rw-r--  1 kali kali    280 Sep 30 04:18 8702.zip
                                                                                                                                                     
┌──(kali㉿kali)-[/tmp]
└─$ 
```

Một file zip có tên: **8702.zip** được trích xuất. Trong đây có thể chứa thông tin chúng ta đang cần. Hãy thử giải nén file đó ra.

**Giải nén file**

```bash
┌──(kali㉿kali)-[/tmp]
└─$ 7za e _cutie.png.extracted/8702.zip 

7-Zip (a) 24.07 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-06-19
 64-bit locale=en_US.UTF-8 Threads:32 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: _cutie.png.extracted/8702.zip
--
Path = _cutie.png.extracted/8702.zip
Type = zip
Physical Size = 280

    
Enter password (will not be echoed):

```

Có vẻ chúng ta đã gặp chút rắc rối vì file yêu cầu mật khẩu để có thể giải nén được. Các bước tiếp theo sẽ thực hiện crack zip file để truy xuất được thông tin cần tìm.

> Trong mật mã truyền tin, 2 bên thường sẽ có một khóa (thường gọi là secret key). Khóa này như là một password dùng “mở khóa” để lấy được những thông tin mà người gửi muốn truyền tải.  Cách này càng giúp cuộc điều tra trở nên khó khăn vì đã tăng thêm 1 lớp xác thực.
> 

**Crack zip password**

```bash
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ zip2john 8702.zip > dump.hash    
                                                                                                                                                     
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ ls    
365  365.zlib  8702.zip  dump.hash
                                                                                                                                                     
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ cat dump.hash          
8702.zip/To_agentR.txt:$zip2$*0*1*0*4673cae714579045*67aa*4e*61c4cf3af94e649f827e5964ce575c5f7a239c48fb992c8ea8cbffe51d03755e0ca861a5a3dcbabfa618784b85075f0ef476c6da8261805bd0a4309db38835ad32613e3dc5d7e87c0f91c0b5e64e*4969f382486cb6767ae6*$/zip2$:To_agentR.txt:8702.zip:8702.zip

```

Sau khi nhận được hash. Lúc này hãy thực hiện brute-force để tìm được mật khẩu

```bash
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ john dump.hash                                                         
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 AVX 4x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 8 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:01 DONE 2/3 (2024-09-30 04:36) 1.000g/s 45470p/s 45470c/s 45470C/s 123456..ferrises
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Rất may mắn khi mật khẩu được đặt không quá phức tạp nên việc crack hoàn thành rất nhanh chóng. Lúc này hãy sử dụng mật khẩu đã crack được là **alien** để unzip file

```bash
# Extract file
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ 7za e 8702.zip                     

7-Zip (a) 24.07 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-06-19
 64-bit locale=en_US.UTF-8 Threads:32 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: 8702.zip
--
Path = 8702.zip
Type = zip
Physical Size = 280

    
Enter password (will not be echoed):
Everything is Ok

Size:       86
Compressed: 280
                                                                                                                                       
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ ls
365  365.zlib  8702.zip  dump.hash  To_agentR.txt

# Trích xuất thông tin bí mật                                                                                                                                                     
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ cat To_agentR.txt 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
                                                                                                                                                     
┌──(kali㉿kali)-[/tmp/_cutie.png.extracted]
└─$ 

```

Done, sẽ khá là bất ngờ khi lần đầu tiên bạn khám phá được cách nhúng thông tin bí mật vào một file khác. Hầu hết các file đều có thể nhúng những thông tin bí mật khác vào chúng. Quan trọng là trong quá trình truyền tải, các dữ liệu đó có phải là những dữ liệu vi phạm tính chất về cộng đồng hay những nội dung độc hại được các mạng xã hội detect được và xóa nó đi hay không, còn việc nhúng thông tin bí mật vào file khá dễ dàng. Vì vậy, các tội phạm thường sẽ sử dụng những mạng xã hội lỏng lẻo trong tính bảo mật để thuận lợi trong việc trao đổi thông tin bất hợp pháp.