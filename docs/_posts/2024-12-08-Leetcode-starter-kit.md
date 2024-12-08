---
layout: post
title:  "Leetcode starter kit"
author: "Tuna"
comments: false
category: Coding
tags: leetcode vietnamese
custom_excerpt: |
  Hướng dẫn bắt đầu với LeetCode, bao gồm các mẹo và chiến lược để tối ưu hóa quá trình luyện tập thuật toán. Bài viết này sẽ giúp bạn vượt qua những khó khăn ban đầu và cải thiện kỹ năng giải thuật toán của mình.
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

![My LeetCode stats](/images/2024/12/my-leetcode-stats.png)

## TL;DR:
- Bắt đầu LeetCode càng sớm càng tốt.
- Dùng Python để code, nếu không bắt buộc dùng một ngôn ngữ nhất định.
- Code bằng IDE nhưng tắt auto-complete.
- Đo thời gian giải bài.
- Note lại các thông tin về bài toán: cách tiếp cận, các lưu ý, time/space complexity, các edge cases, v.v.
- Giải theo chủ đề.
- Làm bài daily để duy trì thói quen và động lực.

Nếu bạn nóng lòng xem danh sách về các tài liệu học Cấu trúc dữ liệu và giải thuật, hãy chuyển tới [Các nguồn tham khảo để học về DS&A](#các-nguồn-tham-khảo-để-học-về-dsa)

## Giới thiệu

Mình không giỏi về thuật toán và cũng không phải dân chơi CP (competitive programming), nên khi bắt đầu với LeetCode, mình đã bị ngợp và không biết bắt đầu từ đâu, mất rất nhiều thời gian để giải những bài đầu tiên. Mình đoán nhiều bạn cũng có cảm giác tương tự khi bắt đầu với LeetCode. Bài viết này tổng hợp một vài thứ có thể hữu ích cho những bạn giống mình.

> Trong bài này, thuật ngữ "**_Thuật toán_**" hoặc "**_Algorithm_**" được dùng để chỉ các thuật toán theo sách giáo khoa hoặc cần một vài bước suy luận và các cách tiếp cận cụ thể, khác với nghĩa của từ "thuật toán" nói chung, vốn có thể dùng cho tất cả logic trong code.

## 0. Tại sao lại là [LeetCode][leetcode]?

Hiện nay có khá nhiều nền tảng để luyện tập thuật toán như [TopCoder][topcoder], [Codeforces][codeforces], [CodeChef][codechef], [Kattis][kattis], v.v. LeetCode hay được nhắc đến như một mặc định khi nói về luyện tập cho coding interview. Mặc dù mỗi người có lý do riêng để chọn nền tảng phù hợp, dưới đây là những lý do mình chọn LeetCode:


- Không yêu cầu parse input/output, giúp bạn tập trung vào logic giải quyết vấn đề.
- Lời giải tương tự như viết một hàm thông thường, không phải bận tâm về cách input/output được format như thế nào.
- Mô tả ngắn gọn và straight-forward, giúp bạn hiểu được yêu cầu của bài toán ngay khi đọc đề thay vì phải đọc và suy nghĩ nhiều.

By the way, lựa chọn nền tảng nào cũng được, điều quan trọng là chốt lại một nền tảng và **bắt đầu**.

[leetcode]: https://leetcode.com/
[topcoder]: https://www.topcoder.com/
[codeforces]: https://codeforces.com/
[codechef]: https://www.codechef.com/
[kattis]: https://open.kattis.com/

## 1. Lúc nào thì nên bắt đầu Leetcode?
Câu trả lời ngắn gọn là **bây giờ**.

Tuy nhiên, **_nếu bạn chưa từng học về DS&A, hãy học một khoá về DS&A trước khi bắt đầu làm LeetCode_**. Điều này sẽ giúp bạn có cái nhìn tổng quan về các thuật toán và cấu trúc dữ liệu, giúp bạn dễ dàng hơn khi tiếp cận các bài toán phức tạp hơn cũng như có thể đọc hiểu được các lời giải từ các bài viết trên mạng.

Nếu bạn ít khi phải giải quyết vấn đề bằng một thuật toán nào đó, khả năng bạn sẽ mất cảm giác về việc sử dụng thuật toán, các cấu trúc dữ liệu phức tạp ngoài `ArrayList` hoặc `HashMap`. Bắt đầu sớm sẽ giúp bạn lấy lại cảm giác và xây dựng được một **mental model** về các thuật toán và cấu trúc dữ liệu cần thiết để giải các bài khó hơn hoặc áp dụng được vào công việc hằng ngày.

Không cần phải phân vân liệu bạn có cần học lại các khóa về **Data Structures & Algorithms (DS&A)** trước hay không. Chỉ cần **bắt đầu**, và khi gặp phải một bài toán cần một giải thuật cụ thể, lúc đó bạn có thể tìm hiểu tài liệu liên quan. Phương pháp tiếp cận tuần tự sẽ giúp bạn học nhanh hơn và thực hành hiệu quả hơn. Công thức của mình là:
1. Làm LeetCode.
2. Gặp câu không giải được.
3. Xem lời giải.
4. Tìm hiểu các giải thuật và cấu trúc dữ liệu liên quan.
5. Giải các câu liên quan để củng cố.

Tuy nhiên, để tránh nản lòng, bạn **không nên chọn một câu ngẫu nhiên để làm bài đầu tiên.** Nếu câu LeetCode đầu tiên của bạn yêu cầu đến những khái niệm khó như **Dynamic Programming**, **LinkedList**, hay **Binary Tree**, bạn sẽ dễ cảm thấy bế tắc. Vì vậy, hãy bắt đầu với những câu cơ bản, dễ hiểu để xây dựng nền tảng.

**Một vài bài LeetCode đơn giản giúp bạn khởi động:**
1. **[Two Sum][two-sum]** (Easy - Array): Một bài cơ bản giúp làm quen với cách xử lý mảng và sử dụng HashMap.
2. **[Reverse String][reverse-string]** (Easy - String): Một bài luyện tập đơn giản liên quan đến thao tác chuỗi.
3. **[Merge Two Sorted Lists][merge-lists]** (Easy - LinkedList): Giúp bạn làm quen với cách thao tác trên danh sách liên kết.
4. **[Best Time to Buy and Sell Stock][best-time]** (Easy - Array): Một bài cơ bản để hiểu cách tìm giá trị tối ưu với một vòng lặp đơn.
5. **[Valid Parentheses][valid-parent]** (Easy - Stack): Một bài tập phổ biến để làm quen với cấu trúc dữ liệu Stack.

  [two-sum]: https://leetcode.com/problems/two-sum/
  [reverse-string]: https://leetcode.com/problems/reverse-string/
  [merge-lists]: https://leetcode.com/problems/merge-two-sorted-lists/
  [best-time]: https://leetcode.com/problems/best-time-to-buy-and-sell-stock/
  [valid-parent]: https://leetcode.com/problems/valid-parentheses/

Bắt đầu với những bài như thế này không chỉ giúp bạn làm quen với nền tảng LeetCode mà còn mang lại cảm giác thành công, từ đó tăng thêm động lực để tiếp tục hành trình.


## 2. Dùng ngôn ngữ lập trình gì cho LeetCode?
Có ba ngôn ngữ bạn nên cân nhắc lựa chọn khi làm LeetCode:

**_1.  Ngôn ngữ mà team hoặc công ty bạn nhắm đến đang sử dụng._**
Nếu bạn nhắm đến ứng tuyển vào một công ty hoặc team yêu cầu sử dụng ngôn ngữ cụ thể, hãy ưu tiên học và thực hành bằng ngôn ngữ đó.

**_2. Ngôn ngữ bạn đang sử dụng và quen thuộc._**
Sử dụng ngôn ngữ mà bạn đã nắm vững sẽ giảm bớt áp lực khi vừa phải học thuật toán vừa phải làm quen với cú pháp mới. Điều này giúp bạn tập trung nhiều hơn vào tư duy giải quyết vấn đề và cải thiện logic lập trình.

**_3. Python._**
Với mình, Python là lựa chọn lý tưởng sau **pseudo code** để mô phỏng thuật toán. Ngôn ngữ này rất dễ học, cú pháp đơn giản và không yêu cầu nhiều bước chuẩn bị. Ví dụ:
  - Để tạo một **array list**, bạn chỉ cần viết `array = []`.
  -  Với **hash map**, chỉ cần `m = {}`.

Điều này rất tiện lợi vì bạn không cần bận tâm về việc import các thư viện như ArrayList trong Java hay khai báo các template như C++.

**Lợi ích của Python khi làm LeetCode:**

**_1. Cú pháp ngắn gọn:_**

Python cho phép bạn viết mã ngắn gọn và tập trung hơn vào logic. Như ví dụ ở trên, để định nghĩa một list hoặc map, bạn chỉ cần

```python
array = []
a_map = {}
```

Trong khi ở C++ hoặc Java, bạn cần khai báo kiểu dữ liệu của biến, import thư viện liên quan, làm code trở nên dài hơn và phải nhớ đường dẫn của thư viện đó.

**_2. Thư viện tích hợp sẵn:_**

Python có số lượng thư viện tích hợp lớn, hỗ trợ nhiều cấu trúc dữ liệu và thuật toán. Một vài ví dụ:

- `heapq`: Hiện thực Priority Queue.
- `deque` từ collections: Hỗ trợ Double-ended Queue.
- `math`: Hỗ trợ các hàm toán học cơ bản như `gcd, factorial, sqrt, v.v`.

**_3. Code block by indent_**
Một điều ít khi được để ý việc dùng indent (4-space) để định nghĩa các code block thay vì các cặp `{...}` hay `begin...end` trong Pascal thực sự hữu ích, nhất là trong phỏng vấn khi dùng bảng vì nó giúp code ngắn gọn và tiết kiệm không gian rất nhiều.

Ví dụ, khi phải viết 2 vòng lặp lồng nhau:

- Với Java hoặc C++:
```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        // ...
    }
}
```

- Với Python:
```python
for i in range(n):
    for j in range(m):
        # ...
```


## 3. Hãy đo thời gian giải bài
Khi bạn bắt đầu giải bài, hãy đo thời gian mà bạn đã dành cho mỗi bài toán. Điều này giúp bạn đánh giá được kĩ năng của mình đã tiến bộ như thế nào, có dạng bài nào mà bạn mất nhiều thời gian hơn, hay cần cải thiện thêm.

Ngoài ra, đặt ngưỡng thời gian cho mỗi level của bài toán (Easy, Medium, Hard) để không rơi vào việc suy nghĩ quá lâu và thường dẫn đến việc dùng trick hay các hacky solution để được accept.

Ngưỡng thời gian mình đặt cho mỗi level _(bạn có thể điều chỉnh ngưỡng thời gian cho phù hợp với mình)_:

- **Easy**: Dưới 15 phút.
- **Medium**: Dưới 30 phút.
- **Hard**: Dưới 60 phút.

Nếu vượt quá thời gian mà vẫn chưa tìm ra lời giải tối ưu, mình sẽ xem hướng dẫn hoặc lời giải để học hỏi cách tiếp cận. Sau đó, mình không vội code và submit ngay, mà để dành bài đó cho ngày hôm sau, nhằm kiểm tra xem mình đã thực sự hiểu và áp dụng được phương pháp hay chưa.

## 4. Hãy note lại
Khi giải một bài toán, hãy note lại các thông tin sau:
- **Intuition**: Từ mô tả bài toán, các giải thuật và cấu trúc dữ liệu có thể dùng để giải bài toán. Các edge cases cần lưu ý.
- **Time and Space Complexity**: Đánh giá big-0 của lời giải của bạn.
- Các ghi chú khác, ví dụ, đã sai ở đâu, cần bao nhiêu lần chạy thử, thời gian cần để bạn hoàn thành bài toán, v.v.

Các note này giúp bạn luyện tập thêm việc trao đổi trong quá trình phỏng vấn cũng như xem mình đã mắc lỗi ở đâu và tiến bộ như thế nào qua mỗi bài toán. Dưới đây là form note mình dùng:

```
[Hard] 2577. Minimum Time to Visit a Cell In a Grid
https://leetcode.com/problems/minimum-time-to-visit-a-cell-in-a-grid/
Created: 2024-12-08 08:25 (Sunday)
Done   : - mins
Attempt: -
---------------------NOTE---------------------
Time: O(-)
Space: O(-)

Intuition:
-
```

## 5. Giải theo chủ đề

Sau giai đoạn kickoff, khi bạn đã quen với cách tiếp cận bài toán và làm quen với cú pháp lập trình, đây là lúc chuyển sang giải theo chủ đề. Phương pháp này giúp bạn xây dựng sự tự tin và thành thạo trong việc áp dụng thuật toán cũng như cấu trúc dữ liệu liên quan đến từng loại bài toán cụ thể.

#### Tại sao nên giải theo chủ đề?

**_1.	Xây dựng kiến thức có hệ thống_**

Việc tập trung vào từng chủ đề giúp bạn hiểu sâu hơn về thuật toán và cấu trúc dữ liệu liên quan, thay vì chỉ giải rải rác và không có định hướng.

**_2. Cải thiện khả năng nhận diện bài toán_**

Nhiều bài toán phức tạp thường thuộc các chủ đề quen thuộc. Khi bạn đã thành thạo một chủ đề, bạn sẽ dễ dàng nhận ra giải pháp tiềm năng khi gặp các bài toán tương tự.

**_3.	Tăng hiệu quả ôn tập_**

Khi bạn luyện tập theo nhóm bài liên quan, bạn sẽ dễ nhớ và nắm bắt được mối liên hệ giữa các dạng bài trong cùng một chủ đề, từ đó tối ưu hóa thời gian học tập.

#### Bắt đầu từ đâu?
Chủ đề đầu tiên mình khuyên bạn luyện tập là [Binary Search][binary-search].

- Đây là một thuật toán cơ bản nhưng cực kỳ quan trọng. Binary Search không chỉ xuất hiện trong các bài toán trực tiếp mà còn là nền tảng cho nhiều dạng bài phức tạp hơn như tìm kiếm trên không gian câu trả lời (searching on answer space).
- Một khi bạn hiểu rõ cách hoạt động của Binary Search, bạn sẽ dễ dàng áp dụng nó vào các bài toán như tìm kiếm trên mảng, tối ưu hóa giá trị, hoặc các bài toán liên quan đến đồ thị.

Tiếp theo, bạn có thể chọn chủ đề luyện tập dựa trên sở thích cá nhân hoặc các dạng bài toán bạn thường gặp. Đừng quên tận dụng [Study Plan][study-plan] của LeetCode để xây dựng lộ trình phù hợp.

Ngoài ra, bạn cũng có thể tham khảo các curated list có sẵn như [LeetCode 75][lc75] hoặc [Grokking the Code Interview similar list][grokking-list] để luyện tập hiệu quả hơn.


[binary-search]: https://leetcode.com/studyplan/binary-search/
[study-plan]: https://leetcode.com/studyplan/
[lc75]: https://leetcode.com/studyplan/leetcode-75/
[grokking-list]: https://gist.github.com/tykurtz/3548a31f673588c05c89f9ca42067bc4


## 6. Một số toolkit khác
### Làm bài daily
<img src="/images/2024/12/my-daily-complete.png" width=200/>

Bài LeetCode daily problem đôi khi khó đôi khi dễ, và có cảm giác hơi random, tuy nhiên, làm bài daily có 2 tác dụng là:
- Giúp bản thân có động lực duy trì. Một khi đã làm được tầm 30 ngày liên tiếp, bạn sẽ không muốn mình bị mất cái streak này.
- Phát hiện ra những dạng bài mới hoặc những Data Structure hay Algorithm mới. Chính sự hơi random sẽ bổ sung vào kho DS&A của bạn mỗi tuần, phần bù cho việc chỉ giải theo chủ đề như đề cập ở trên.

### Hãy dùng IDE khi mới bắt đầu
Hầu hết chúng ta đều phụ thuộc khá nhiều vào các công cụ hỗ trợ như auto-complete, auto-import, hay thậm chí gần đây là AI để viết code. Chính vì vậy, viết code thẳng lên LeetCode editor thời gian đầu sẽ gặp rất nhiều khó khăn. Bạn sẽ gặp kha khá lỗi sai về cú pháp, thiếu import,... Để tránh các lỗi này, hãy dùng IDE thời gian đầu với một số lượng hạn chế các automation tools, ví dụ, chỉ cho auto-import hoạt động, auto-complete và AI sẽ dc tắt. Mình dùng **[VS Code - Insiders][vscode-insiders]** (phiên bản early release của VS Code) cho LeetCode và cấu hình mặc định tắt hết tất cả các tool automation đi.

Ngoài ra, việc debug trên local vẫn tiện hơn so với việc dùng LeetCode run, nhất là khi code của bạn rơi vào vòng lặp vô hạn.

Bạn có thể tham khảo thêm bài viết [Let's code with Leetcode](letscode) về việc tạo ra một trình chạy test cho các bài giải LeetCode của mình.

[vscode-insiders]: https://code.visualstudio.com/insiders/
[letcode]: https://iamtuna.org/2019-01-05/lets-code-with-leetcode

### Hướng tới việc 1 hit 1 ~~kill~~ accept
Điều này hàm ý bạn có một giai đoạn phân tích và test các phân tích của mình trước khi viết code. Đừng vội vàng viết code ngay khi đọc xong đề, hãy đảm bảo bạn đã hiểu rõ yêu cầu và cách tiếp cận bài toán. Một khi bạn đã viết code, hãy test kỹ trước khi submit.
Đây là cách hiệu quả để luyện tập cho việc viết code trong môi trường phỏng vấn.

## Các nguồn tham khảo để học về DS&A

#### Sách
- **[Algorithm Design Manual](http://www.algorist.com/)**: Một cuốn sách khác về thuật toán và cấu trúc dữ liệu, được viết bởi Steven S. Skiena. Theo đánh giá của mình thì cuốn này phù hợp hơn với người đã đi làm và muốn cải thiện kỹ năng thuật toán.
- **[Algorithms (4th Edition)][algorithms-book]**: Một trong những cuốn sách nền tảng về thuật toán và cấu trúc dữ liệu.
- **[Introduction to Algorithms (3rd Edition)][clrs-book]**: Một cuốn sách khác về thuật toán và cấu trúc dữ liệu, thường được dùng trong các khóa học về DS&A.
- **[Giải thuật và lập trình][vietnamese-books]**: Một cuốn sách tiếng Việt về thuật toán và lập trình của thầy Lê Minh Hoàng.
- **[Các tài liệu tiếng Việt khác][vietnamese-books]**: Một số tài liệu tiếng Việt khác về thuật toán và lập trình được liệt kê và tổng hợp trên VNOI Wiki.

[algorithms-book]: https://algs4.cs.princeton.edu/home/
[clrs-book]: https://mitpress.mit.edu/books/introduction-algorithms-third-edition
[vietnamese-books]: https://wiki.vnoi.info/algo/basic/Tai-Lieu-Thuat-Toan

#### Online resources (không bao gồm các course)
Dưới đây là một số nguồn tham khảo online mà mình thấy hữu ích:

**Text-based**:
- [The Algorithms](https://the-algorithms.com/)
- [William Fiset's Algorithms Repository](https://github.com/williamfiset/Algorithms)
- [Liu Zheng Lai's Algorithm GitBook](https://liuzhenglaichn.gitbook.io/algorithm)
- [Codeforces Blog](https://codeforces.com/blog/entry/13529)
- [VNOI Wiki (tiếng việt)](https://wiki.vnoi.info/)
- [CP Algorithms](https://cp-algorithms.com/)

**YouTube channels**:
- [William Fiset](https://www.youtube.com/@WilliamFiset-videos)
- [Stable Sort](https://www.youtube.com/@stablesort)
