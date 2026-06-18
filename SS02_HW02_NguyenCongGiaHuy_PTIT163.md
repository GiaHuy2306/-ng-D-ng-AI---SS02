# BÀI 2: Tối ưu Prompt - Bảo mật thông tin nhạy cảm

## 1. Nội dung Prompt sau khi tối ưu

Hãy viết cho tôi một class Java tên là `VNPayPaymentService` để xử lý thanh toán trực tuyến qua VNPay trong môi trường sandbox.

Yêu cầu bắt buộc về bảo mật:

* Tuyệt đối không hardcode API Key, HASH_SECRET, token, mật khẩu hoặc bất kỳ thông tin nhạy cảm nào trực tiếp trong mã nguồn.
* Không sử dụng giá trị khóa bí mật thật trong ví dụ.
* Khóa bí mật dùng để tạo chữ ký phải được lấy từ biến môi trường bằng `System.getenv("VNPAY_HASH_SECRET")`.
* URL thanh toán VNPay có thể được cấu hình trong hằng số hoặc biến môi trường, nhưng khóa bí mật bắt buộc phải lấy từ biến môi trường.
* Nếu biến môi trường `VNPAY_HASH_SECRET` bị thiếu, bằng `null`, hoặc là chuỗi rỗng sau khi `trim()`, chương trình phải ném ra ngoại lệ `IllegalStateException` với thông báo rõ ràng.
* Class cần có logic tạo URL thanh toán VNPay, tạo chữ ký HMAC SHA512 và mã hóa tham số theo chuẩn URL.
* Mã nguồn cần rõ ràng, dễ hiểu, có comment ngắn gọn cho người mới học Java.
* Kết quả trả về là một file/class Java hoàn chỉnh, an toàn hơn về mặt bảo mật.

Vui lòng sinh mã nguồn Java hoàn chỉnh cho class `VNPayPaymentService`.

## 2. Đoạn mã Java hoàn chỉnh và an toàn

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.*;

/**
 * VNPayPaymentService
 *
 * Class này dùng để tạo URL thanh toán VNPay.
 * Điểm quan trọng về bảo mật:
 * - Không hardcode HASH_SECRET trong mã nguồn.
 * - HASH_SECRET được lấy từ biến môi trường VNPAY_HASH_SECRET.
 * - Nếu thiếu biến môi trường thì chương trình sẽ ném lỗi để tránh chạy sai hoặc mất an toàn.
 */
public class VNPayPaymentService {

    private static final String VNP_URL = "https://sandbox.vnpayment.vn/paymentv2/vpcpay.html";

    private static final String VNP_TMN_CODE = "YOUR_TMN_CODE_HERE";

    private static final String RETURN_URL = "https://your-domain.com/payment/vnpay-return";

    private static final String HASH_SECRET_ENV_NAME = "VNPAY_HASH_SECRET";

    /**
     * Lấy khóa bí mật từ biến môi trường.
     * Không được hardcode khóa bí mật trong source code.
     */
    private String getHashSecret() {
        String hashSecret = System.getenv(HASH_SECRET_ENV_NAME);

        if (hashSecret == null || hashSecret.trim().isEmpty()) {
            throw new IllegalStateException(
                    "Missing required environment variable: " + HASH_SECRET_ENV_NAME
            );
        }

        return hashSecret.trim();
    }

    /**
     * Tạo URL thanh toán VNPay.
     *
     * @param orderId Mã đơn hàng
     * @param amount  Số tiền thanh toán, đơn vị VND
     * @param ipAddr  Địa chỉ IP của người dùng
     * @return URL thanh toán VNPay
     */
    public String createPaymentUrl(String orderId, long amount, String ipAddr) {
        if (orderId == null || orderId.trim().isEmpty()) {
            throw new IllegalArgumentException("orderId must not be null or empty");
        }

        if (amount <= 0) {
            throw new IllegalArgumentException("amount must be greater than 0");
        }

        if (ipAddr == null || ipAddr.trim().isEmpty()) {
            throw new IllegalArgumentException("ipAddr must not be null or empty");
        }

        String hashSecret = getHashSecret();

        Map<String, String> vnpParams = new HashMap<>();

        vnpParams.put("vnp_Version", "2.1.0");
        vnpParams.put("vnp_Command", "pay");
        vnpParams.put("vnp_TmnCode", VNP_TMN_CODE);

        // Theo VNPay, số tiền cần nhân 100
        vnpParams.put("vnp_Amount", String.valueOf(amount * 100));

        vnpParams.put("vnp_CurrCode", "VND");
        vnpParams.put("vnp_TxnRef", orderId);
        vnpParams.put("vnp_OrderInfo", "Thanh toan don hang: " + orderId);
        vnpParams.put("vnp_OrderType", "other");
        vnpParams.put("vnp_Locale", "vn");
        vnpParams.put("vnp_ReturnUrl", RETURN_URL);
        vnpParams.put("vnp_IpAddr", ipAddr);

        Calendar calendar = Calendar.getInstance(TimeZone.getTimeZone("Etc/GMT+7"));
        SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddHHmmss");

        String createDate = formatter.format(calendar.getTime());
        vnpParams.put("vnp_CreateDate", createDate);

        calendar.add(Calendar.MINUTE, 15);
        String expireDate = formatter.format(calendar.getTime());
        vnpParams.put("vnp_ExpireDate", expireDate);

        List<String> fieldNames = new ArrayList<>(vnpParams.keySet());
        Collections.sort(fieldNames);

        StringBuilder hashData = new StringBuilder();
        StringBuilder query = new StringBuilder();

        for (int i = 0; i < fieldNames.size(); i++) {
            String fieldName = fieldNames.get(i);
            String fieldValue = vnpParams.get(fieldName);

            if (fieldValue != null && !fieldValue.isEmpty()) {
                String encodedFieldName = urlEncode(fieldName);
                String encodedFieldValue = urlEncode(fieldValue);

                hashData.append(encodedFieldName)
                        .append("=")
                        .append(encodedFieldValue);

                query.append(encodedFieldName)
                        .append("=")
                        .append(encodedFieldValue);

                if (i < fieldNames.size() - 1) {
                    hashData.append("&");
                    query.append("&");
                }
            }
        }

        String secureHash = hmacSHA512(hashSecret, hashData.toString());

        query.append("&vnp_SecureHash=").append(secureHash);

        return VNP_URL + "?" + query;
    }

    /**
     * Mã hóa chuỗi theo chuẩn URL.
     */
    private String urlEncode(String value) {
        return URLEncoder.encode(value, StandardCharsets.UTF_8);
    }

    /**
     * Tạo chữ ký HMAC SHA512.
     *
     * @param key  Khóa bí mật lấy từ biến môi trường
     * @param data Dữ liệu cần ký
     * @return Chuỗi chữ ký dạng hexadecimal
     */
    private String hmacSHA512(String key, String data) {
        try {
            Mac hmac512 = Mac.getInstance("HmacSHA512");
            SecretKeySpec secretKey = new SecretKeySpec(
                    key.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA512"
            );

            hmac512.init(secretKey);

            byte[] bytes = hmac512.doFinal(data.getBytes(StandardCharsets.UTF_8));

            StringBuilder hash = new StringBuilder();
            for (byte b : bytes) {
                hash.append(String.format("%02x", b));
            }

            return hash.toString();

        } catch (Exception e) {
            throw new RuntimeException("Error while generating HMAC SHA512 signature", e);
        }
    }

    /**
     * Ví dụ chạy thử.
     * Trước khi chạy cần cấu hình biến môi trường:
     *
     * Windows PowerShell:
     * $env:VNPAY_HASH_SECRET="your_secret_here"
     *
     * Linux/macOS:
     * export VNPAY_HASH_SECRET="your_secret_here"
     */
    public static void main(String[] args) {
        VNPayPaymentService service = new VNPayPaymentService();

        String paymentUrl = service.createPaymentUrl(
                "ORDER_001",
                100000,
                "127.0.0.1"
        );

        System.out.println(paymentUrl);
    }
}
```

## 3. Giải thích ngắn gọn

Prompt ban đầu chưa an toàn vì đưa trực tiếp `HASH_SECRET` thật vào nội dung gửi cho AI. Đây là thông tin nhạy cảm, nếu bị lộ có thể ảnh hưởng đến hệ thống thanh toán.

Prompt tối ưu đã sửa vấn đề này bằng cách yêu cầu AI:

* Không hardcode khóa bí mật.
* Không dùng API Key thật trong ví dụ.
* Lấy khóa từ biến môi trường `VNPAY_HASH_SECRET`.
* Kiểm tra biến môi trường trước khi sử dụng.
* Ném `IllegalStateException` nếu cấu hình bị thiếu.

Cách làm này an toàn hơn vì mã nguồn không chứa trực tiếp thông tin bí mật. Khi triển khai lên môi trường thật, khóa bí mật sẽ được cấu hình bên ngoài chương trình.
