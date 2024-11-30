---
layout: post
title:  "Tôi làm gì khi tôi review code"
author: "Tuna"
comments: false
category: Coding
tags: review vietnamese mentoring
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

Bên cạnh coding, *review code* và *mentoring* là hai trong số những công việc mình thực hiện hằng ngày. Thông thường, trừ khi phải tập trung code cho một feature mới hay fix bug, *review code* là công việc chiếm nhiều thời gian nhất. <!--more--> Nhân đây, cũng muốn flex một chút là mình là thành viên của **Code Review Committee** – một committee định ra các quy chuẩn, coding style, và đôi khi là system design cho **LINE Chat client app**.

Sau đây, mình xin chia sẻ một số việc mình thường làm khi review một [**Pull Request** (PR)][1].

## 1.  Đặt mình vào hoàn cảnh của tác giả
Việc này giúp mình *“be human”* hơn khi viết comment, đồng thời hiểu tại sao tác giả lại code hoặc mắc lỗi như vậy.

Ví dụ gần đây, một đồng nghiệp của mình vừa mới có em bé. Bạn ấy phải dành nhiều thời gian và tâm trí chăm sóc gia đình, nên khó có thể hoàn toàn tập trung khi code. Hiểu được điều này, mình sẽ không quá khắt khe khi bạn ấy lặp lại một vài lỗi nhỏ, hoặc mình sẽ giải thích kỹ hơn, thậm chí viết sẵn mẫu code để bạn ấy có thể copy-paste dễ dàng.

Một ví dụ khác: Khi tác giả đang làm việc theo từng bước nhỏ (step-by-step adding code), một vài lỗi nhỏ có thể xảy ra, như:
- Format chưa đúng;
- Tên hàm/biến không còn phù hợp;
- Quên xóa những đoạn code tạm dùng để debug…

Hiểu được điều này, mình sẽ coi những lỗi này như những “tai nạn” khi code và nhắc nhở nhẹ nhàng kiểu: 

> “*I guess you forgot pressing Command + Shift + L to format the code*”

Nguyên nhân chính thường là do tác giả phải chuyển đổi ngữ cảnh (context switch) nhiều, dẫn đến quên xử lý một vài đoạn code nhỏ.

## 2. Tránh dùng "you"

Dù review code là quá trình giao tiếp giữa reviewer và tác giả, mục tiêu chính của việc review là **code**, không phải **người viết code**.

> “This code looks not correct”  

thay vì

> “You wrote this code incorrectly” 

Thay vì dùng “you,” mình thường dùng “we” để trao đổi với tác giả, chẳng hạn: *“I think we should change this to …”*

Ngoài ra, để làm nhẹ không khí, mình thường thêm các cụm từ như _“I think, I guess, I feel, IMO…”_ vào comment. Điều này giúp tránh cảm giác *“bị tấn công”* đối với tác giả. Không ai thích bị soi mói sau khi đã dành rất nhiều thời gian viết code, test, debug… Việc dùng “we” tạo cảm giác cả reviewer và tác giả cùng đứng trên một chiến tuyến (mà thực tế đúng là như vậy). Reviewer sau này cũng phải maintain và làm việc với code của tác giả, nên tất cả đều hướng tới mục tiêu cải thiện chất lượng code sau mỗi lần merge.

Tuy nhiên, không phải lúc nào cũng cần tránh hoàn toàn “you”. Có những trường hợp mình vẫn dùng “you”, chẳng hạn:
- _“Could you help me double-check this for memory leaks?”_
- _“Could you confirm this with the product owner?”_
- _“Could you discuss this with the designer?”_
- _“Thank you!”_
- _“The new code looks awesome, thank you for your great efforts!”_

Hoặc vừa mới hôm qua, mình viết thế này:
- _“If you want more fun and faster (in case the list is long), we can use binary search.”_

## 3. Không thoả hiệp với ambiguous code

**Ambiguous code** xảy ra khi (bao gồm nhưng không phải tất cả nguyên nhân):
- Tên biến/hàm không rõ nghĩa hoặc không đúng ngữ cảnh;
- Có thứ tự ngầm định giữa các dòng code;
- Các thuật ngữ chưa được định nghĩa rõ ràng;
- v.v...

Loại code này gây khó khăn rất lớn cho quá trình maintain sau này.

Cả tác giả và reviewer thường có **bias** rằng code hiện tại “có thể hiểu được” tại thời điểm review, vì họ còn nhớ rõ context. Nhưng chỉ sau khoảng một năm, gần như không ai nhớ context đó nữa.

**Cách hạn chế bias**
Khi code hoặc review code, luôn tự hỏi: *“năm sau nếu đọc lại thì mình có hiểu được đoạn này có nghĩa là gì không?”* để hạn chế việc thêm và cho phép ambiguous code xảy ra. Nếu được, có thể nhờ một người ngoài team review thử.

Khi review, mình tránh việc diễn giải đoạn code theo cách mà nó không thể hiện hoặc không làm rõ. Thường, mình đóng vai một **dummy**, tự đặt mình vào vị trí của người không biết gì về feature, dù đó là code do team mình maintain hay dựa trên phần mình từng viết. Mentor của mình thường tự nhắc đi nhắc lại: _“I’m not good enough to understand this.”_

**Cách giảm ambiguous**

Nếu context của đoạn code vượt ra khỏi khả năng giải thích của tên hàm, tên biến, hoặc cấu trúc code, **hãy viết comment giải thích trực tiếp trong code**. Những phần như commit log hay PR description tuy hữu ích nhưng không đủ, vì chúng không dễ truy cập khi cần đọc và hiểu đoạn code sau này. Gợi ý này chắc chắn sẽ không làm hài lòng những ai theo đuổi triết lý _Clean Code_, vì một trong những nguyên tắc của sách này là *“Code tự nói lên ý nghĩa của nó.”* . Một phản biện là "Comment thường không được maintain". Để giải quyết luận điểm này, hãy maintain cả comment lúc code và lúc review code.

Bên cạnh đó, mình cũng muốn nhấn mạnh rằng việc nhận biết một đoạn code có rõ ràng hay không là một kỹ năng khó. Nó khó bởi vì khi đọc code, bạn cần **hạ thấp độ tự tin** để nhận ra vấn đề, nhưng khi viết code, bạn lại cần **tăng độ tự tin** để giải quyết vấn đề.

Đối với các bạn junior, thử thách còn lớn hơn vì các bạn vừa phải nỗ lực diễn giải đoạn code để hiểu rõ vấn đề, học cái mới,... vừa phải chống lại xu hướng tự diễn giải để nhận biết **bad code**. Đây là một sự cân bằng không dễ đạt được nhưng rất cần thiết để trở thành một reviewer giỏi.

## 4. Thoả hiệp

Dù nói không với ambiguous code, không phải lúc nào cũng có thể tìm ra giải pháp tốt ngay tại thời điểm review. Đó là lý do tồn tại **tech debt**. Vì vậy, mình luôn sẵn sàng thoả hiệp với author trong những trường hợp cần thiết.

**Thoả hiệp như thế nào?**

Quy tắc của mình là: nếu author nhận thức đúng về hạn chế của đoạn code, mình sẽ đồng ý cho merge, với điều kiện có ghi chú rõ ràng (*TODO* hoặc comment). Ngược lại, mình sẽ yêu cầu author chứng minh, test, thử nghiệm, hoặc phân tích các kiểu để đảm bảo họ hiểu rõ limitation, side effects, và actual usage của đoạn code. Với những case phức tạp như state machine hay concurrency, đôi khi mình sẽ tự làm thử nghiệm để minh chứng cho tác giả.

**Khi tác giả không chịu thì sao?**

Mình sẽ nhẹ nhàng sử dụng *Request change*. Đây là công cụ mình rất ít khi dùng, vì nó dễ tạo cảm giác không thoải mái cho cả hai bên. Tuy nhiên, nó cần thiết để ngăn bad code bị merge vào project.

Một vấn đề mình thường gặp là nhiều thành viên của team mình hay dùng là chứng minh code bằng thực nghiệm thay vì dựa trên lý thuyết hoặc phân tích logic, kiểu *“chạy thấy ổn, không có lỗi gì.”* Một số trường hợp thì cách chứng mình này có thể chấp nhận được, nhưng tốt hơn là sử dụng **theory**, **diagram**, hoặc **data flow analysis** để giải thích, nhất là với các vấn đề liên quan đến concurrency.

## Lời kết

Review code không chỉ là một bước kiểm tra chất lượng mà còn là cơ hội để học hỏi, cải thiện kỹ năng, và xây dựng sự đồng cảm giữa các thành viên trong team. Là một reviewer, mình luôn cố gắng đặt mục tiêu chung lên hàng đầu: tạo ra code không chỉ đúng mà còn rõ ràng, dễ hiểu, và dễ maintain. Điều này đòi hỏi sự nghiêm khắc nhưng cũng cần linh hoạt và sẵn sàng thoả hiệp khi cần thiết.

Mình tin rằng một quy trình review tốt không chỉ cải thiện chất lượng sản phẩm mà còn giúp mọi người trong team phát triển, từ junior đến senior. Hy vọng bài viết này sẽ mang lại góc nhìn hữu ích cho bạn trong hành trình trở thành một code reviewer hiệu quả và có trách nhiệm.

Cảm ơn bạn đã đọc!

[1]: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests