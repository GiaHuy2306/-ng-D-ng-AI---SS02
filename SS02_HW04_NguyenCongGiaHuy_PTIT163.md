# BÀI 4: Phân tích & Lựa chọn  
## Liêm chính học thuật và Kiểm chứng đầu ra

## 1. Đáp án lựa chọn

**Chọn phương án B.**

Phương án B là phương án đúng nhất vì người học không sao chép máy móc kết quả do AI sinh ra, mà biết kiểm tra lại rủi ro kỹ thuật, tra cứu tài liệu chính thức và tự viết thêm kiểm thử để xác minh kết quả.

---

## 2. Phân tích lý do chọn phương án B

Trong bài toán tính tiền lãi tiết kiệm, dữ liệu đang xử lý là **tiền tệ**. Với tiền tệ, yêu cầu quan trọng là độ chính xác khi tính toán. Nếu dùng sai kiểu dữ liệu, kết quả có thể bị lệch dù chương trình vẫn chạy được.

Đoạn code ban đầu dùng kiểu `double`:

```java
public class SavingsAccount {

    public double calculateBalance(double initialAmount, double interestRate, int months) {

        double total = initialAmount;

        for (int i = 0; i < months; i++) {

            total += total * (interestRate / 12);

        }

        return total;

    }

}
```

Về mặt cú pháp, đoạn code này không sai. Tuy nhiên, về mặt kỹ thuật thực tế, cách làm này chưa phù hợp với bài toán tài chính.

Trong Java, `double` và `float` là kiểu số thực dấu phẩy động nhị phân. Chúng không biểu diễn chính xác được nhiều số thập phân như `0.1`, `0.2`, `0.01`. Vì vậy, phép tính tiền tệ có thể sinh ra sai số nhỏ. Nếu tính lãi qua nhiều tháng, sai số đó có thể bị tích lũy.

Ví dụ:

```java
System.out.println(0.1 + 0.2);
```

Kết quả có thể là:

```text
0.30000000000000004
```

Đối với bài toán thông thường, sai số nhỏ này có thể chấp nhận được. Nhưng đối với bài toán tài chính, sai số tiền tệ là vấn đề nghiêm trọng.

Phương án B đúng vì sinh viên đã làm các việc sau:

1. **Không tin tuyệt đối vào AI**  
   AI có thể sinh code chạy được nhưng chưa chắc đúng về mặt tiêu chuẩn kỹ thuật thực tế.

2. **Nhận ra rủi ro của kiểu `double`**  
   Người học hiểu rằng tính toán tiền tệ cần độ chính xác cao, không nên dùng `double` hoặc `float`.

3. **Tra cứu tài liệu Java chính thức**  
   Trong Java, lớp `BigDecimal` thuộc package `java.math` được dùng cho các phép toán số thập phân có độ chính xác cao.

4. **Sử dụng `RoundingMode` rõ ràng**  
   Khi xử lý tiền, cần quy định cách làm tròn. `RoundingMode` giúp kiểm soát cách làm tròn trong các phép tính.

5. **Tự viết hàm `main` để kiểm thử**  
   Việc kiểm thử với số tiền lớn, lãi suất nhỏ và số tháng dài giúp phát hiện sai số hoặc lỗi logic mà các test đơn giản có thể bỏ qua.

Vì vậy, phương án B thể hiện đúng vai trò của người học như một **kiểm duyệt viên đầu ra của AI**. AI chỉ là công cụ hỗ trợ, còn người học vẫn phải chịu trách nhiệm kiểm chứng, hiểu bản chất và đảm bảo bài làm đúng.

---

## 3. Gợi ý cách viết lại bằng `BigDecimal`

Ví dụ có thể cải tiến hàm tính lãi như sau:

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class SavingsAccount {

    public BigDecimal calculateBalance(BigDecimal initialAmount, BigDecimal annualInterestRate, int months) {
        if (initialAmount == null || annualInterestRate == null) {
            throw new IllegalArgumentException("Initial amount and interest rate must not be null");
        }

        if (initialAmount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Initial amount must not be negative");
        }

        if (annualInterestRate.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Interest rate must not be negative");
        }

        if (months < 0) {
            throw new IllegalArgumentException("Months must not be negative");
        }

        BigDecimal total = initialAmount;

        BigDecimal monthlyInterestRate = annualInterestRate
                .divide(BigDecimal.valueOf(12), 10, RoundingMode.HALF_UP);

        for (int i = 0; i < months; i++) {
            BigDecimal interest = total.multiply(monthlyInterestRate);
            total = total.add(interest);
        }

        return total.setScale(2, RoundingMode.HALF_UP);
    }

    public static void main(String[] args) {
        SavingsAccount account = new SavingsAccount();

        BigDecimal initialAmount = new BigDecimal("1000000000.00");
        BigDecimal annualInterestRate = new BigDecimal("0.0001");
        int months = 360;

        BigDecimal result = account.calculateBalance(initialAmount, annualInterestRate, months);

        System.out.println("Final balance: " + result);
    }
}
```

Ở đây:

- `BigDecimal` được dùng thay cho `double`.
- `new BigDecimal("1000000000.00")` được dùng thay vì `new BigDecimal(1000000000.00)` để tránh đưa sai số từ `double` vào `BigDecimal`.
- `RoundingMode.HALF_UP` được dùng để làm tròn rõ ràng.
- Kết quả cuối cùng được làm tròn về 2 chữ số thập phân, phù hợp với tiền tệ.

---

## 4. Nhược điểm hoặc hành vi vi phạm ở các phương án còn lại

## Phương án A

**Phương án A:** Sao chép nguyên văn code AI sinh ra vì test đơn giản vẫn đúng và AI khẳng định code hoạt động chính xác.

Phương án này không phù hợp vì sinh viên phụ thuộc hoàn toàn vào AI mà không kiểm chứng lại.

Các vấn đề chính:

- Chỉ vì chương trình vượt qua test đơn giản không có nghĩa là chương trình đúng trong mọi trường hợp.
- AI có thể đưa ra câu trả lời tự tin nhưng vẫn sai.
- Dùng `double` trong tính toán tài chính có nguy cơ sai số.
- Người học không thể hiện trách nhiệm học thuật vì không hiểu và không kiểm tra chất lượng mã nguồn trước khi nộp.

Đây là hành vi không tốt trong học tập vì người học chỉ sao chép, không đánh giá lại đầu ra của AI.

---

## Phương án C

**Phương án C:** Sao chép code AI và chỉ đổi `double` thành `float` để tiết kiệm bộ nhớ.

Phương án này còn tệ hơn phương án A.

Lý do là `float` có độ chính xác thấp hơn `double`. Nếu `double` đã không phù hợp cho tính toán tiền tệ, thì `float` càng không phù hợp hơn.

Các vấn đề chính:

- `float` dễ sai số hơn `double`.
- Tiết kiệm bộ nhớ không phải ưu tiên chính trong bài toán tài chính.
- Không tra cứu tài liệu.
- Không kiểm thử với giá trị biên.
- Không hiểu bản chất lỗi làm tròn trong kiểu số thực dấu phẩy động.

Cách làm này thể hiện tư duy xử lý đối phó, không đảm bảo chất lượng kỹ thuật và không phù hợp với trách nhiệm học thuật.

---

## 5. Kết luận

Em chọn **phương án B** vì đây là phương án đúng nhất cả về mặt học thuật lẫn kỹ thuật.

Người học cần hiểu rằng AI chỉ là công cụ hỗ trợ. Trước khi nộp bài, sinh viên phải kiểm tra lại mã nguồn, đối chiếu tài liệu chính thức và tự chạy thử với các trường hợp quan trọng. Với bài toán tài chính, việc sử dụng `BigDecimal` cùng `RoundingMode` là lựa chọn phù hợp hơn so với `double` hoặc `float`.
