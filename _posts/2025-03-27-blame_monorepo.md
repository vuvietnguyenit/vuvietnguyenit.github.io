---
layout: post
title: Kick Your Ass, Monorepo Mindset!
subtitle: Nay đọc được một bài viết làm bản thân thấy phí tiền mạng internet vì ước gì nay mình không đi làm để được đọc nó. Thực sự thì việc viết và show-off một cái gì đấy luôn luôn dễ hơn nhiều việc đem đến một kiến thức chính xác đến tất cả mọi người.  
tags: [sre, system, design-structure]
author: VuNguyen
---

Nếu dùng để quản lý cấu hình tâp trung thì ok, chúng ta có thể tracking tất cả những thay đổi trên toàn bộ hệ thống của chúng ta trên một repo tập trung, điều này rõ ràng là tiện dụng vì không cần quản lý quá nhiều cấu hình lẻ tẻ ở những nơi khác nhau.

**Tuy nhiên, đối với source code. Hãy thử nghĩ về việc làm điều tương tự như với configuation management. Well, we should not do it.**

Bởi vì source code thì chúng ta cần build, packaging, release it. Sẽ thật vô nghĩa nếu release không có version. Sẽ thật khốn kiếp nếu có điều gì đó xảy ra và chúng ta cần revert chúng về version trước đấy. Khi bạn cho tất cả source code vào a mono-repo.

- Hoặc là bạn sẽ manual tagging version nó theo folder mà bạn chỉ định.
- Hoặc là bạn sẽ cần tagging và build lại toàn bộ repo khổng lồ khi bạn thay đổi một phần rất nhỏ.
- Bạn muốn quản lý a build script phức tạp thay vì nhiều scripts build khác ở nhiều nơi ? Điều gì sẽ xảy ra khi có một thành phần mới xuất hiện trong project của bạn, và bạn sẽ phải cập nhật lại build script tương tương ứng với sự thay đổi đó ? Sẽ không có tính tái sử dụng nào ở đây cả!
- Bạn muốn bỏ lỡ tất cả những tính năng tuyệt vời sẵn có mà Source Version Control đem lại không. Hay bạn muốn **build lại một cái bánh xe to hơn nhưng thực sự cái bánh xe to đó nó không giúp xe của bạn đi nhanh hơn**!!!!

Nghe có vẻ hợp lý và uy tín khi trong blog này đề cập đến việc Google sử dụng Monorepo:

[Why does Google use Monorepo?](https://blog.bytebytego.com/p/ep62-why-does-google-use-monorepo "Why does Google use Monorepo?") by Alex Xu

Thật sự thì, Google họ sử dụng nhiều mô hình khác nhau, không chỉ riêng Monorepo. Bài viết được đăng tải vào năm 2023 bởi một blogger tương đối nổi tiếng. **For fuck's sake, I honestly can't call this a complete article :D**

Việc Google sử dụng mô hình monorepo xảy ra từ trước khi sự phát triển mạnh mẽ của Source Version Control và CI/CD khi codebase của họ tại thời điểm đó bản chất được quản lý trên một legacy centralized version control system (có thể hiểu đơn giản là self-host).
Chưa kể, câu chuyện ở đây là đối với một repo lưu trữ C programing language code. Về bản chất họ muốn việc quản lý các dependencies phức tạp trở nên thuận tiện dẫn đến việc tạo ra sân chơi cho monorepo để tất cả mọi thứ đều có thể tái sử dụng và gọi lẫn nhau.

[Why Google Stores Billions of Lines of Code in a Single Repository](https://cacm.acm.org/research/why-google-stores-billions-of-lines-of-code-in-a-single-repository/ "Why Google Stores Billions of Lines of Code in a Single Repository")

Sau câu chuyện này, họ đã mất rất nhiều tiền và công sức để thực hiện migrate nó. Đây là bài học được rút ra.
Ngoài ra, kể cả Kubernetes moved to GitHub from a monorepo.

[GitHub Issue #24343](https://github.com/kubernetes/kubernetes/issues/24343)

[](https://kccnceu2023.sched.com/event/1HycF/mission-accomplished-kubernetes-is-not-a-monorepo-now-our-work-begins-justin-santa-barbara-google-ciprian-hacman-microsoft)

Tuy nhiên, bài đăn của blogger nổi tiếng kia được đăng tải vào năm 2023. Làm ít nhiều người đọc có những suy nghĩ rằng: "Google đã làm được, vậy thì chúng ta hãy học theo và áp dụng nó như best practice như Google đã làm" mà không thực sự hiểu được ngữ cảnh tại sao nó lại được sử dụng như vậy ? Quy mô của mình có đang giống với quy mô của Google không ?

Tóm lại, monorepo không còn phù hợp ở thời điểm hiện tại để lưu trữ source code, công nghệ đang dần thay đổi và chúng ta phải thích nghi với nó. Đừng cố **Reinventing the wheel** và phức tạp hóa mọi thứ. Một người kỹ sư khôn ngoan luôn biết cách sử dụng những thứ sẵn có và làm một công việc bằng cách đơn giản nhất có thể.

![image.png](/assets/img/photo_2025-03-27_21-11-03.jpg)

Cheers.
