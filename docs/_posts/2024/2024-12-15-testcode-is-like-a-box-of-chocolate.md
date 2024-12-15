---
layout: post
title:  "Test code is like a box of chocolates"
author: "Tuna"
comments: false
category: Coding
tags: testing
image: https://rare-gallery.com/thumbnail/72620-Tom-HanksForrest-Gump-HD-Wallpaper.jpg
excerpt: |
  Test code giống như hộp sô-cô-la, bạn sẽ không bao giờ biết mình sẽ nhận được gì… cho đến khi nó bị lỗi!
excerpt_separator: <!--more-->
lang: vi
sticky: false
hidden: false
draft: false
---

Hãy tưởng tượng một nhà toán học đang muốn kiểm chứng giả thuyết Collatz ([Collatz conjecture][collatz]), được mô tả như sau:

{: .box-note}
> **Collatz conjecture**: Bắt đầu với một số nguyên dương bất kỳ. Nếu số đó lẻ, nhân 3 và cộng 1. Nếu số đó chẵn, chia cho 2. Tiếp tục áp dụng quy luật này cho đến khi chuỗi số quay về 1.

Nhà toán học viết một đoạn code kiểm tra giả thuyết này:
```kotlin
fun calculate(n: Int): Int {
    if (n % 2 == 0) {
        return n / 2
    } else {
        return n * 3 + 1
    }
}
```

Đồng thời, họ viết một đoạn test code như sau:
```kotlin
@Test
fun testCalculate() {
    val numbers = listOf(1,2,3,4,5, 500, 5001,1000,1001)

    for (num in numbers) {
        if (num % 2 == 0) {
            assertEquals(num / 2, calculate(num))
        } else {
            assertEquals(num * 3 + 1, calculate(num))
        }
    }
}
```

Theo bạn thì có vấn đề gì với đoạn test code không?

<img src="/images/2024/12/thinking-meme.png" width=240 style="padding: 1.5rem">

**Vấn đề nằm ở đâu?**

Dù đoạn code trên có vẻ đơn giản và hoạt động tốt, nhưng nó tiềm ẩn một số vấn đề, đặc biệt khi bài kiểm thử bị lỗi.


## Refactor test code: Làm rõ ràng và dễ duy trì hơn

Thường thì **_test code không được viết để đọc, mà là để bị làm cho sai_** (Test failed). Cơ hội để test code được xem lại chỉ xảy ra khi test bị failed, và điều này có thể xảy ra vài năm sau khi chúng ta đã quên hết về context xung quanh đoạn test code.

Hãy thử refactor đoạn test code!

#### 1. Chia nhỏ bài test theo từng nhánh logic
Thay vì sử dụng cấu trúc if-else trong vòng lặp, chúng ta nên chia tập hợp đầu vào thành hai nhóm: số chẵn và số lẻ. Điều này giúp bạn tập trung kiểm tra từng nhánh logic riêng biệt mà không phụ thuộc vào điều kiện phân nhánh trong bài kiểm thử.

```kotlin
@Test
fun testCalculateWithOddNumbers() {
    val oddNumbers = listOf(1, 3, 5, 5001, 1001)
    for (num in oddNumbers) {
        assertEquals(num * 3 + 1, calculate(num))
    }
}

@Test
fun testCalculateWithEvenNumbers() {
    val evenNumbers = listOf(2, 4, 500, 1000)
    for (num in evenNumbers) {
        assertEquals(num / 2, calculate(num))
    }
}
```

#### 2. Loại bỏ vòng lặp để tăng tính rõ ràng
Việc sử dụng vòng lặp trong test code khiến việc nhận diện test case nào bị lỗi trở nên khó khăn. Bạn không cần tuân thủ nguyên tắc [DRY (Don’t Repeat Yourself)](dry) trong test code, bởi tính rõ ràng quan trọng hơn tính tối ưu.

```kotlin
fun testCalculateWithOddNumbers() {
    assertEquals(1 * 3 + 1, calculate(1))
    assertEquals(3 * 3 + 1, calculate(3))
    assertEquals(5 * 3 + 1, calculate(5))
    assertEquals(5001 * 3 + 1, calculate(5001))
    assertEquals(1001 * 3 + 1, calculate(1001))
}

fun testCalculateWithEvenNumbers() {
    assertEquals(2 / 2, calculate(2))
    assertEquals(4 / 2, calculate(4))
    assertEquals(500 / 2, calculate(500))
    assertEquals(1000 / 2, calculate(1000))
}
```

Việc loại bỏ vòng lặp giúp bạn nhanh chóng xác định test case nào đang thất bại mà không cần thêm bước debug.

{: .box-italic}
> Lưu ý: phương pháp này phù hợp khi thực hiện xác minh (assertion) giá trị kết quả, khác với cách tổ chức và chạy test như sử dụng `t.Run()` trong Go. (Trong Go, `t.Run()` hỗ trợ table-testing và cho phép dễ dàng truy xuất vị trí khai báo của đầu vào khi test bị failed, trong khi Java và Kotlin, theo hiểu biết hiện tại của mình, chưa có tính năng tương tự.)

### 3. Sử dụng giá trị tuyệt đối nếu có thể

Tham chiếu đến giá trị tuyệt đối hoặc giá trị cuối cùng của phép tính giúp chúng ta nhảy đến trường hợp kiểm thử bị lỗi và nhanh chóng hiểu được vấn đề thông qua thông báo lỗi.

Có hai cách chúng ta có thể áp dụng điều này vào ví dụ của chúng ta:

**Option 1**
```kotlin
fun testCalculateWithOddNumbers() {
    assertEquals(4, calculate(1)) // 1 * 3 + 1
    assertEquals(10, calculate(3)) // 3 * 3 + 1
    assertEquals(16, calculate(5)) // 5 * 3 + 1
    assertEquals(15004, calculate(5001)) // 5001 * 3 + 1
    assertEquals(3004, calculate(1001)) // 1001 * 3 + 1
}
```

**Option 2**
```kotlin
fun testCalculateWithOddNumbers() {
    assertEquals(1 * 3 + 1, calculate(1)) // 4
    assertEquals(3 * 3 + 1, calculate(3)) // 10
    assertEquals(5 * 3 + 1, calculate(5)) // 16
    assertEquals(5001 * 3 + 1, calculate(5001)) // 15004
    assertEquals(1001 * 3 + 1, calculate(1001)) // 3004
}
```
Cả hai cách đều giúp test code dễ đọc hơn và làm nổi bật giá trị mong đợi trong mỗi assertion.

## Kết luận
Test code là để đảm bảo tính đúng đắn và rõ ràng của test cases, không phải để tránh việc lặp lại code. Một test case dễ đọc và dễ hiểu sẽ giúp bạn nhanh chóng xác định vấn đề khi test failed.

**_Test code is like a box of chocolates, you never know what you're going to get (until it’s broken)_** - 
_"Test code giống như hộp sô-cô-la, bạn sẽ không bao giờ biết mình sẽ nhận được gì… cho đến khi nó bị lỗi!"_

(bắt chước câu "_**Life is like a box of chocolates** - Forrest Gump_")


![Forrest Gump](https://rare-gallery.com/thumbnail/72620-Tom-HanksForrest-Gump-HD-Wallpaper.jpg)
*Credit: [rare-gallery.com](https://rare-gallery.com/72620-forrest-gump-hd-wallpapertom-hanks.html)*

## Đọc thêm
- [Testing on the Toilet: Tests Too DRY? Make Them DAMP!](https://testing.googleblog.com/2019/12/testing-on-toilet-tests-too-dry-make.html)


[collatz]: https://en.wikipedia.org/wiki/Collatz_conjecture
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself