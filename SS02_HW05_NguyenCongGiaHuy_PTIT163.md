# BÀI 5: Sáng tạo  
## Thiết kế Quy trình & Prompt Kiểm chứng Đầu ra - AI Verifier

## 1. Bối cảnh và ý đồ thiết kế quy trình 2 bước

Trong hệ thống bán vé xe khách trực tuyến **GreenBus**, người dùng nhập mã vé để hệ thống kiểm tra tính hợp lệ trước khi tra cứu hoặc xác nhận vé.

Mã vé hợp lệ cần tuân theo các quy tắc:

- Bắt đầu bằng tiền tố `BUS-`.
- Tiếp theo là mã tỉnh/thành phố gồm đúng 2 ký tự chữ in hoa, ví dụ: `HN`, `SG`, `DN`.
- Tiếp theo là dấu gạch ngang `-`.
- Cuối cùng là 6 chữ số biểu diễn ngày đi xe theo định dạng `YYMMDD`.
- Ngày đi xe không được là ngày trong quá khứ.

Ví dụ:

```text
BUS-HN-260615
```

Nghĩa là vé xe từ Hà Nội, ngày đi là `15/06/2026`.

Ý đồ thiết kế quy trình 2 bước là không chỉ dùng AI để sinh code, mà còn dùng một lượt AI độc lập khác để **kiểm chứng, phản biện và phát hiện lỗi logic tiềm ẩn**. Cách làm này giúp người lập trình không phụ thuộc hoàn toàn vào kết quả AI sinh ra lần đầu.

Quy trình gồm:

1. **Bước 1 - Code Generation Prompt:**  
   Yêu cầu AI sinh lớp `TicketValidator` bằng Java theo đúng nghiệp vụ.

2. **Bước 2 - AI Auditing Prompt:**  
   Yêu cầu AI khác hoặc một phiên kiểm tra độc lập đóng vai kỹ sư kiểm thử/kiểm toán bảo mật để rà soát code, tìm lỗi logic và đề xuất sửa.

Cách làm này thể hiện tư duy kiểm chứng đầu ra AI, giảm nguy cơ nộp hoặc đưa vào dự án một đoạn code nhìn có vẻ đúng nhưng vẫn có lỗi biên.

---

## 2. Nội dung Prompt sinh mã nguồn - Bước 1

```text
Bạn là một lập trình viên Java đang phát triển hệ thống bán vé xe khách trực tuyến GreenBus.

Hãy viết một lớp Java tên là TicketValidator để kiểm tra tính hợp lệ của mã số vé do người dùng nhập vào.

Quy tắc nghiệp vụ của mã vé hợp lệ:

1. Mã vé bắt buộc phải bắt đầu bằng tiền tố "BUS-".
2. Sau tiền tố là mã tỉnh/thành phố gồm đúng 2 ký tự chữ in hoa, ví dụ: HN, SG, DN.
3. Sau mã tỉnh/thành phố là dấu gạch ngang "-".
4. Phần cuối là 6 chữ số biểu diễn ngày đi xe theo định dạng YYMMDD.
5. Ngày đi xe không được là ngày trong quá khứ, tức là phải lớn hơn hoặc bằng ngày hiện tại của hệ thống.

Ví dụ mã hợp lệ:
BUS-HN-260615

Yêu cầu kỹ thuật:

1. Viết class Java hoàn chỉnh tên là TicketValidator.
2. Có phương thức public boolean isValid(String ticketCode).
3. Phải xử lý các trường hợp mã vé null, chuỗi rỗng hoặc chỉ chứa khoảng trắng.
4. Phải kiểm tra đúng định dạng mã vé.
5. Phải phát hiện ngày tháng không hợp lệ, ví dụ ngày 32, tháng 13 hoặc ngày không tồn tại.
6. Phải kiểm tra ngày đi xe không nằm trong quá khứ.
7. Không được để xảy ra NullPointerException hoặc lỗi runtime không kiểm soát khi người dùng nhập sai.
8. Viết thêm hàm main để chạy thử một số ví dụ hợp lệ và không hợp lệ.
9. Code cần rõ ràng, dễ hiểu, có comment ngắn gọn cho sinh viên mới học Java.
```

---

## 3. Nội dung Prompt kiểm chứng độc lập - Bước 2

```text
Bạn là một kỹ sư kiểm thử và kiểm toán bảo mật độc lập.

Tôi sẽ cung cấp mã nguồn Java của lớp TicketValidator được AI sinh ra ở bước trước. Nhiệm vụ của bạn là không tin mặc định rằng code đúng, mà phải rà soát, phản biện và tìm lỗi logic tiềm ẩn.

Hãy kiểm tra các điểm sau:

1. Code có xử lý null, chuỗi rỗng và chuỗi chỉ chứa khoảng trắng an toàn không?
2. Code có kiểm tra đúng định dạng BUS-XX-YYMMDD không?
3. Code có đảm bảo XX là đúng 2 ký tự chữ in hoa không?
4. Code có phát hiện ngày không hợp lệ như tháng 13, ngày 32, hoặc ngày không tồn tại như 29/02 ở năm không nhuận không?
5. Cách phân tích YYMMDD có bị lỗi khi chuyển giao năm mới không?
6. Code có sử dụng API ngày tháng cũ như Date hoặc SimpleDateFormat không? Nếu có, hãy đánh giá rủi ro.
7. Code có xử lý múi giờ hệ thống rõ ràng không?
8. Code có nguy cơ NullPointerException, StringIndexOutOfBoundsException hoặc lỗi runtime khi input sai không?
9. Code có kiểm thử đủ các giá trị biên chưa?
10. Hãy đề xuất phiên bản tối ưu hơn, ưu tiên sử dụng java.time.LocalDate, Clock hoặc ZoneId nếu cần.

Yêu cầu đầu ra:

- Nhận xét tổng quan.
- Danh sách lỗi hoặc rủi ro cụ thể.
- Mức độ nghiêm trọng của từng lỗi.
- Đề xuất sửa.
- Mã nguồn Java cuối cùng đã được tối ưu.
```

---

## 4. Minh chứng thực tế dạng text log

### 4.1. Bước 1 - AI sinh mã nguồn lần đầu

**Người dùng gửi prompt:**

```text
Bạn là một lập trình viên Java đang phát triển hệ thống bán vé xe khách trực tuyến GreenBus.

Hãy viết một lớp Java tên là TicketValidator để kiểm tra tính hợp lệ của mã số vé do người dùng nhập vào.

Quy tắc nghiệp vụ của mã vé hợp lệ:

1. Mã vé bắt buộc phải bắt đầu bằng tiền tố "BUS-".
2. Sau tiền tố là mã tỉnh/thành phố gồm đúng 2 ký tự chữ in hoa, ví dụ: HN, SG, DN.
3. Sau mã tỉnh/thành phố là dấu gạch ngang "-".
4. Phần cuối là 6 chữ số biểu diễn ngày đi xe theo định dạng YYMMDD.
5. Ngày đi xe không được là ngày trong quá khứ.

Hãy viết class Java hoàn chỉnh tên là TicketValidator.
```

**AI sinh mã nguồn lần đầu:**

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class TicketValidator {

    public boolean isValid(String ticketCode) {
        if (ticketCode == null || ticketCode.trim().isEmpty()) {
            return false;
        }

        if (!ticketCode.matches("^BUS-[A-Z]{2}-\\d{6}$")) {
            return false;
        }

        String datePart = ticketCode.substring(7);

        SimpleDateFormat sdf = new SimpleDateFormat("yyMMdd");

        try {
            Date travelDate = sdf.parse(datePart);
            Date today = new Date();

            return !travelDate.before(today);

        } catch (ParseException e) {
            return false;
        }
    }

    public static void main(String[] args) {
        TicketValidator validator = new TicketValidator();

        System.out.println(validator.isValid("BUS-HN-260615"));
        System.out.println(validator.isValid("BUS-SG-991332"));
        System.out.println(validator.isValid(""));
        System.out.println(validator.isValid(null));
    }
}
```

---

### 4.2. Bước 2 - AI kiểm duyệt độc lập chỉ ra lỗi

**Người dùng gửi prompt kiểm chứng:**

```text
Bạn là một kỹ sư kiểm thử và kiểm toán bảo mật độc lập.

Hãy rà soát mã nguồn TicketValidator sau đây. Không được tin mặc định rằng code đúng. Hãy tìm lỗi logic, lỗi ngày tháng, lỗi múi giờ và lỗi runtime tiềm ẩn.

[Đưa mã nguồn TicketValidator ở Bước 1 vào đây]
```

**AI kiểm duyệt phản hồi:**

```text
Nhận xét tổng quan:

Code có kiểm tra null, chuỗi rỗng và regex cơ bản. Tuy nhiên, vẫn có nhiều rủi ro logic quan trọng liên quan đến ngày tháng.

Các lỗi và rủi ro phát hiện được:

1. Sử dụng SimpleDateFormat nhưng không tắt chế độ lenient
   - Mức độ: Cao
   - SimpleDateFormat mặc định có thể tự điều chỉnh ngày không hợp lệ.
   - Ví dụ ngày 32 tháng 13 có thể bị tự chuyển sang một ngày hợp lệ khác thay vì báo lỗi.
   - Điều này làm mã vé sai vẫn có thể được chấp nhận.

2. So sánh Date có cả giờ, phút, giây
   - Mức độ: Cao
   - Vé đi ngày hôm nay có thể bị coi là quá khứ nếu travelDate được parse về 00:00:00, còn today là thời điểm hiện tại trong ngày.
   - Theo nghiệp vụ, ngày đi bằng ngày hiện tại vẫn phải hợp lệ.

3. Sử dụng API ngày tháng cũ Date và SimpleDateFormat
   - Mức độ: Trung bình
   - API cũ dễ gây lỗi về parsing, timezone và lenient date.
   - Nên dùng java.time.LocalDate hiện đại hơn.

4. Chưa xử lý rõ ràng múi giờ hệ thống
   - Mức độ: Trung bình
   - Local date của hệ thống có thể khác nhau nếu server chạy ở múi giờ khác.
   - Nên dùng Clock hoặc ZoneId để dễ kiểm thử và kiểm soát ngày hiện tại.

5. substring(7) hiện tại tạm an toàn vì regex đã kiểm tra trước
   - Mức độ: Thấp
   - Tuy nhiên vẫn nên giữ thứ tự kiểm tra này cẩn thận để tránh StringIndexOutOfBoundsException.

Đề xuất sửa:

- Dùng java.time.LocalDate thay cho Date.
- Parse YYMMDD bằng cách tách year, month, day rõ ràng.
- Chuyển YY thành năm 2000 + YY, ví dụ 26 thành 2026.
- Dùng LocalDate.of(year, month, day) để bắt lỗi ngày không hợp lệ.
- So sánh bằng LocalDate, không so sánh Date có giờ phút giây.
- Dùng Clock để dễ kiểm thử ngày hiện tại.
```

---

## 5. Mã nguồn Java tối ưu cuối cùng sau kiểm duyệt

```java
import java.time.Clock;
import java.time.DateTimeException;
import java.time.LocalDate;
import java.util.regex.Pattern;

/**
 * TicketValidator dùng để kiểm tra mã vé của hệ thống GreenBus.
 *
 * Quy tắc mã vé hợp lệ:
 * - Bắt đầu bằng "BUS-"
 * - Tiếp theo là đúng 2 ký tự chữ in hoa
 * - Tiếp theo là dấu "-"
 * - Cuối cùng là 6 chữ số ngày đi theo định dạng YYMMDD
 * - Ngày đi không được là ngày trong quá khứ
 *
 * Ví dụ:
 * BUS-HN-260615
 */
public class TicketValidator {

    /*
     * Regex kiểm tra định dạng tổng quát:
     * ^BUS-       : bắt đầu bằng BUS-
     * [A-Z]{2}    : đúng 2 chữ cái in hoa
     * -           : dấu gạch ngang
     * \\d{6}      : đúng 6 chữ số ngày tháng
     * $           : kết thúc chuỗi
     */
    private static final Pattern TICKET_PATTERN =
            Pattern.compile("^BUS-[A-Z]{2}-\\d{6}$");

    private final Clock clock;

    /**
     * Constructor mặc định dùng ngày hiện tại theo hệ thống.
     */
    public TicketValidator() {
        this.clock = Clock.systemDefaultZone();
    }

    /**
     * Constructor nhận Clock để dễ kiểm thử.
     * Ví dụ khi test có thể cố định ngày hiện tại.
     */
    public TicketValidator(Clock clock) {
        if (clock == null) {
            throw new IllegalArgumentException("clock must not be null");
        }

        this.clock = clock;
    }

    /**
     * Kiểm tra mã vé có hợp lệ hay không.
     *
     * @param ticketCode mã vé người dùng nhập
     * @return true nếu hợp lệ, false nếu không hợp lệ
     */
    public boolean isValid(String ticketCode) {
        if (ticketCode == null) {
            return false;
        }

        String normalizedCode = ticketCode.trim();

        if (normalizedCode.isEmpty()) {
            return false;
        }

        if (!TICKET_PATTERN.matcher(normalizedCode).matches()) {
            return false;
        }

        String datePart = normalizedCode.substring(7);

        LocalDate travelDate = parseTravelDate(datePart);

        if (travelDate == null) {
            return false;
        }

        LocalDate today = LocalDate.now(clock);

        return !travelDate.isBefore(today);
    }

    /**
     * Chuyển chuỗi YYMMDD thành LocalDate.
     *
     * Ví dụ:
     * "260615" -> 2026-06-15
     *
     * Nếu ngày không hợp lệ, hàm trả về null.
     */
    private LocalDate parseTravelDate(String datePart) {
        try {
            int year = 2000 + Integer.parseInt(datePart.substring(0, 2));
            int month = Integer.parseInt(datePart.substring(2, 4));
            int day = Integer.parseInt(datePart.substring(4, 6));

            return LocalDate.of(year, month, day);

        } catch (NumberFormatException | DateTimeException e) {
            return null;
        }
    }

    public static void main(String[] args) {
        TicketValidator validator = new TicketValidator();

        System.out.println("=== GreenBus TicketValidator Test ===");

        test(validator, "BUS-HN-260615");     // Có thể true hoặc false tùy ngày hiện tại
        test(validator, "BUS-SG-991332");     // false, tháng/ngày không hợp lệ
        test(validator, "BUS-DN-260230");     // false, ngày 30/02 không tồn tại
        test(validator, "BUS-DN-240101");     // false nếu ngày hiện tại sau 01/01/2024
        test(validator, "BUS-hn-260615");     // false, mã tỉnh phải in hoa
        test(validator, "CAR-HN-260615");     // false, sai tiền tố
        test(validator, "BUS-HN-26061");      // false, thiếu chữ số
        test(validator, "BUS-HNN-260615");    // false, mã tỉnh quá 2 ký tự
        test(validator, "");                  // false
        test(validator, "   ");               // false
        test(validator, null);                // false
    }

    private static void test(TicketValidator validator, String ticketCode) {
        System.out.println(ticketCode + " -> " + validator.isValid(ticketCode));
    }
}
```

---

## 6. Giải thích vì sao mã nguồn cuối cùng tốt hơn

Mã nguồn cuối cùng tốt hơn phiên bản đầu vì:

1. **Dùng `java.time.LocalDate` hiện đại**  
   `LocalDate` phù hợp với bài toán chỉ cần so sánh ngày, không cần giờ phút giây.

2. **Không dùng `SimpleDateFormat`**  
   Tránh rủi ro parse ngày sai hoặc tự điều chỉnh ngày không hợp lệ.

3. **Bắt được ngày không hợp lệ**  
   `LocalDate.of(year, month, day)` sẽ ném lỗi nếu tháng hoặc ngày không hợp lệ, ví dụ tháng 13 hoặc ngày 30/02.

4. **Không coi ngày hôm nay là quá khứ**  
   Vì so sánh bằng `LocalDate`, nếu ngày đi bằng ngày hiện tại thì vẫn hợp lệ.

5. **Có kiểm tra null, rỗng và sai định dạng**  
   Tránh lỗi `NullPointerException` hoặc `StringIndexOutOfBoundsException`.

6. **Có dùng `Clock`**  
   Giúp kiểm thử dễ hơn vì có thể cố định ngày hiện tại khi viết unit test.

---

## 7. Kết luận

Quy trình 2 bước giúp người học không chỉ biết dùng AI để sinh code, mà còn biết dùng AI như một công cụ kiểm chứng độc lập.

Trong bài toán GreenBus, nếu chỉ dùng code sinh ra lần đầu, chương trình có thể sai ở các trường hợp ngày tháng và ngày hiện tại. Sau bước kiểm duyệt, mã nguồn được cải thiện bằng cách dùng `java.time.LocalDate`, kiểm tra ngày không hợp lệ và xử lý tốt hơn các trường hợp biên.

Vì vậy, quy trình này thể hiện đúng tư duy của một kỹ sư phần mềm: **AI hỗ trợ viết code, nhưng con người phải chịu trách nhiệm kiểm chứng chất lượng đầu ra**.