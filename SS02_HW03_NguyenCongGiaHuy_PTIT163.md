# BÀI 3: Đọc hiểu & Dò lỗi qua Prompt  
## Phát hiện ảo giác AI - Hallucination

## 1. Giải thích nguyên lý sinh lỗi ảo giác của AI

Trong đề bài, người dùng yêu cầu AI viết hàm Java sử dụng:

```java
StringUtils.hasSpecialCharacters()
```

Vấn đề là trong thư viện **Apache Commons Lang**, lớp `StringUtils` không có phương thức tên là `hasSpecialCharacters()`.

Việc đưa một phương thức không có thật vào prompt dễ làm AI sinh ra lỗi ảo giác vì các lý do sau:

### Thứ nhất, prompt đã “mớm” cho AI một giả định sai

Khi người dùng viết rằng muốn dùng `StringUtils.hasSpecialCharacters()`, AI có thể hiểu nhầm rằng phương thức đó đã tồn tại. Thay vì kiểm chứng lại API, AI có thể tiếp tục viết code dựa trên giả định sai này.

Ví dụ AI có thể sinh ra đoạn code như sau:

```java
import org.apache.commons.lang3.StringUtils;

public class StringChecker {
    public static boolean hasSpecialCharacters(String input) {
        return StringUtils.hasSpecialCharacters(input);
    }
}
```

Đoạn code trên nhìn có vẻ hợp lý, nhưng sẽ **lỗi biên dịch** vì `hasSpecialCharacters()` không tồn tại trong `StringUtils`.

### Thứ hai, AI thường sinh câu trả lời theo xác suất ngôn ngữ

AI không phải lúc nào cũng kiểm tra hoặc biên dịch code thật. Nó thường dự đoán câu trả lời có vẻ phù hợp dựa trên dữ liệu đã học.

Tên phương thức `hasSpecialCharacters()` nghe rất tự nhiên, đúng kiểu đặt tên hàm Java, nên AI có thể bịa ra phương thức này nếu prompt dẫn dắt sai.

### Thứ ba, thiếu yêu cầu kiểm chứng tài liệu chính thức

Prompt ban đầu chỉ yêu cầu “hãy viết code”, không yêu cầu AI chứng minh phương thức có thật. Vì vậy AI có thể bỏ qua bước kiểm tra tài liệu.

Cách tốt hơn là yêu cầu AI:

- Chỉ dùng API chính thức.
- Không tự bịa tên phương thức.
- Giải thích API được dùng đến từ đâu.
- Nếu dùng thư viện ngoài, phải nêu rõ class và method có tồn tại trong tài liệu chính thức.
- Ưu tiên dùng thư viện chuẩn Java nếu bài toán đơn giản.

Trong bài này, cách an toàn là dùng **Regular Expression** của Java thông qua `java.util.regex.Pattern`. Đây là thư viện chuẩn của Java, thường được dùng để kiểm tra chuỗi theo mẫu biểu thức chính quy.

---

## 2. Prompt tối ưu mới

```text
Hãy viết một hàm Java để kiểm tra một chuỗi có chứa ký tự đặc biệt hay không.

Yêu cầu bắt buộc:
1. Không được sử dụng hoặc tự bịa ra các phương thức không có thật như StringUtils.hasSpecialCharacters().
2. Ưu tiên sử dụng API chính thức của Java, ví dụ java.util.regex.Pattern và Matcher.
3. Nếu sử dụng thư viện ngoài, phải chứng minh class và method đó có tồn tại trong tài liệu chính thức.
4. Trước khi đưa code, hãy giải thích ngắn gọn cơ sở API được dùng:
   - Tên package/class/method.
   - Lý do API đó tồn tại và phù hợp.
   - Nguồn tài liệu chính thống cần kiểm tra, ví dụ Oracle Java Documentation hoặc Apache Commons Lang Javadoc.
5. Hàm cần xử lý trường hợp input là null hoặc chuỗi rỗng.
6. Ký tự đặc biệt được hiểu là ký tự không phải chữ cái, không phải chữ số và không phải khoảng trắng.
7. Trả về true nếu chuỗi có ít nhất một ký tự đặc biệt, ngược lại trả về false.
8. Viết code Java hoàn chỉnh, có thể chạy thử bằng hàm main.
```

---

## 3. Đoạn mã nguồn Java do AI sinh ra sau khi chạy prompt mới

```java
import java.util.regex.Pattern;

/**
 * Lớp kiểm tra chuỗi có chứa ký tự đặc biệt hay không.
 *
 * Cơ sở API:
 * - Sử dụng java.util.regex.Pattern, đây là API chuẩn của Java.
 * - Pattern dùng để biên dịch biểu thức chính quy.
 * - Phương thức matcher(input).find() kiểm tra xem trong chuỗi có phần nào khớp với regex hay không.
 *
 * Quy ước trong bài:
 * - Ký tự đặc biệt là ký tự không phải chữ cái.
 * - Không phải chữ số.
 * - Không phải khoảng trắng.
 */
public class SpecialCharacterChecker {

    /*
     * Regex: [^a-zA-Z0-9\\s]
     *
     * Giải thích:
     * - a-zA-Z: chữ cái tiếng Anh
     * - 0-9: chữ số
     * - \\s: khoảng trắng, tab, xuống dòng
     * - ^ bên trong [] nghĩa là phủ định
     *
     * Vì vậy regex này sẽ tìm ký tự không thuộc nhóm chữ cái,
     * chữ số hoặc khoảng trắng.
     */
    private static final Pattern SPECIAL_CHARACTER_PATTERN =
            Pattern.compile("[^a-zA-Z0-9\\s]");

    /**
     * Kiểm tra chuỗi có ký tự đặc biệt hay không.
     *
     * @param input chuỗi cần kiểm tra
     * @return true nếu có ký tự đặc biệt, false nếu không có
     */
    public static boolean hasSpecialCharacters(String input) {
        if (input == null || input.isEmpty()) {
            return false;
        }

        return SPECIAL_CHARACTER_PATTERN.matcher(input).find();
    }

    public static void main(String[] args) {
        System.out.println(hasSpecialCharacters("Hello123"));        // false
        System.out.println(hasSpecialCharacters("Hello 123"));       // false
        System.out.println(hasSpecialCharacters("Hello@123"));       // true
        System.out.println(hasSpecialCharacters("Java#Code"));       // true
        System.out.println(hasSpecialCharacters("NoSpecialChar"));   // false
        System.out.println(hasSpecialCharacters(""));                // false
        System.out.println(hasSpecialCharacters(null));              // false
    }
}
```

---

## 4. Kết luận

Prompt mới tốt hơn prompt cũ vì không ép AI dùng một phương thức chưa được kiểm chứng.

Thay vào đó, prompt yêu cầu AI dùng API chuẩn Java là `java.util.regex.Pattern`, đồng thời yêu cầu giải thích cơ sở tài liệu. Cách này giúp giảm nguy cơ AI bịa ra phương thức không tồn tại và giúp lập trình viên dễ kiểm tra lại trước khi đưa code vào dự án.
