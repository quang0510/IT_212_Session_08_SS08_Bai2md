1.  Nội dung Prompt gỡ lỗi do bạn thiết kế.
    Bạn là một Chuyên gia Gỡ lỗi Java (Professional Java Debugger) và Chuyên gia tối ưu hóa mã nguồn trong hệ thống Tài chính - Ngân hàng.

        Hệ thống SecureBank của chúng tôi vừa gặp sự cố đột ngột khi chạy thử nghiệm phương thức lưu giao dịch vào cơ sở dữ liệu (CSDL) H2. Hệ thống bị sập và ném ra ngoại lệ liên quan đến vi phạm toàn vẹn dữ liệu.

        Dưới đây là mã nguồn gây lỗi (Java):

        import java.sql.Connection;
        import java.sql.PreparedStatement;

        public class TransactionRepository {
        private Connection connection;

            public void saveTransaction(String transactionId, Long userId, double amount) throws Exception {
                String sql = "INSERT INTO transactions (id, user_id, amount) VALUES (?, ?, ?)";
                PreparedStatement ps = connection.prepareStatement(sql);
                ps.setString(1, transactionId);
                ps.setLong(2, userId);
                ps.setDouble(3, amount);
                ps.executeUpdate();
            }
        }

        Và đây là đoạn dấu vết lỗi xuất hiện ở Console (Stack Trace):
        org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException: Referential integrity constraint violation: "CONSTRAINT_FOREIGN_KEY_USER: PUBLIC.TRANSACTIONS FOREIGN KEY(USER_ID) REFERENCES PUBLIC.USERS(ID) (99)"; SQL statement:

        INSERT INTO transactions (id, user_id, amount) VALUES (?, ?, ?) [23506-214] at org.h2.message.DbException.getJdbcSQLException(DbException.java:502)
        at TransactionRepository.saveTransaction(TransactionRepository.java:13) at Main.main(Main.java:10)

        Nhiệm vụ của bạn:

        Giải nghĩa thông báo lỗi: Phân tích chi tiết đoạn Stack Trace trên và chỉ ra Nguyên nhân gốc rễ (Root Cause) dẫn đến việc hệ thống bị sập tại dòng TransactionRepository.java:13.

        Tối ưu và Sửa đổi mã nguồn:

        Hãy viết lại class TransactionRepository trên một cách an toàn và tối ưu hơn.

        Thêm một phương thức bổ trợ hoặc logic kiểm tra (Validation) xem userId có thực sự tồn tại trong bảng USERS của cơ sở dữ liệu hay không trước khi thực hiện câu lệnh INSERT vào bảng transactions.

        Nếu userId KHÔNG tồn tại, hãy ném ra một ngoại lệ tùy chỉnh có tên là UserNotFoundException (vui lòng định nghĩa nhanh class Exception này) thay vì để CSDL ném lỗi hệ thống thô ra ngoài.

        Đảm bảo giải phóng tài nguyên hệ thống (PreparedStatement, ResultSet) một cách chính xác (khuyến khích dùng Try-with-resources của Java).

        Yêu cầu đầu ra: Hãy trình bày rõ ràng phần giải nghĩa lỗi trước, sau đó cung cấp toàn bộ mã nguồn Java đã sửa đổi hoàn chỉnh, sạch và sẵn sàng đưa vào sử dụng.

2.  Minh chứng chạy thực tế:
    Chào bạn, với tư cách là một Chuyên gia Gỡ lỗi Java và Kiến trúc sư Hệ thống Tài chính - Ngân hàng, tôi đã phân tích sự cố và tối ưu lại mã nguồn để đảm bảo tính phòng thủ dữ liệu cao nhất cho hệ thống SecureBank.

    Dưới đây là phần giải nghĩa lỗi chi tiết cùng mã nguồn đã được xử lý triệt để.

    ***

    ## 🔍 1. Phân tích Stack Trace & Nguyên nhân gốc rễ (Root Cause)

    Dựa trên vết lỗi (Stack Trace) thu được từ màn hình Console, chúng ta có các thông tin cốt lõi sau:
    - **Ngoại lệ phát sinh:** `org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException`
    - Đây là ngoại lệ vi phạm ràng buộc toàn vẹn dữ liệu do cơ sở dữ liệu quan hệ H2 ném ra khi một hành động ghi dữ liệu vi phạm quy tắc thiết kế bảng.

    - **Chi tiết thông báo:** `Referential integrity constraint violation: "CONSTRAINT_FOREIGN_KEY_USER: PUBLIC.TRANSACTIONS FOREIGN KEY(USER_ID) REFERENCES PUBLIC.USERS(ID) (99)"`
    - Hệ thống đang vi phạm **Ràng buộc khóa ngoại (Foreign Key Constraint)** có tên là `CONSTRAINT_FOREIGN_KEY_USER`.
    - Quy tắc của DB chỉ định: Cột `USER_ID` trong bảng `PUBLIC.TRANSACTIONS` bắt buộc phải tham chiếu tới một ID hợp lệ và đã tồn tại trong cột `ID` của bảng cha `PUBLIC.USERS`.
    - Giá trị nằm trong ngoặc `(99)` chính là giá trị runtime được truyền vào biến `userId` khi thực hiện câu lệnh.

    - **Vị trí gây lỗi:** Lỗi xảy ra tại dòng số 13 trong file `TransactionRepository.java` khi phương thức `ps.executeUpdate()` được gọi và đẩy lệnh xuống Database.

    **Nguyên nhân gốc rễ (Root Cause):**
    Mã nguồn ứng dụng đã cố tình chèn một bản ghi giao dịch mới với `user_id = 99`. Tuy nhiên, tại thời điểm đó, **bảng `USERS` hoàn toàn không tồn tại bất kỳ bản ghi nào có `id = 99**`. Do tính toàn vẹn tham chiếu, hệ quản trị CSDL H2 đã chặn đứng hành động này và ném lỗi ngược lại ứng dụng. Do mã nguồn cũ thiếu bước validate chủ động nên ngoại lệ thô này đã làm sập luồng xử lý hiện tại.

    ***

    ## 💻 2. Tối ưu và Sửa đổi mã nguồn (Java 17 Standard)

    Để giải quyết triệt để lỗi này, tôi tiến hành áp dụng chiến lược **Lập trình phòng thủ (Defensive Programming)**: Chủ động kiểm tra sự tồn tại của khách hàng trước bằng lệnh `SELECT COUNT(1)` và đóng gói tài nguyên tự động bằng cú pháp **Try-with-resources** nhằm tránh rò rỉ bộ nhớ (Resource Leak).

    ### Bước 2.1: Định nghĩa Custom Exception nghiệp vụ

    Chúng ta tạo ra lớp ngoại lệ `UserNotFoundException`. Việc dùng ngoại lệ kiểm soát (`checked exception` - kế thừa từ `Exception`) hoặc ngoại lệ runtime (`unchecked exception` - kế thừa từ `RuntimeException`) tùy thuộc vào kiến trúc, ở đây tôi chọn `Exception` để đồng bộ với cấu trúc phương thức cũ của bạn.

    ```java
    package com.securebank.repository.exception;

    /**
    * Ngoại lệ nghiệp vụ ném ra khi không tìm thấy thông tin Người dùng trong Hệ thống SecureBank.
    */
    public class UserNotFoundException extends Exception {
        public UserNotFoundException(String message) {
            super(message);
        }
    }

    ```

    ### Bước 2.2: Tối ưu hóa Class TransactionRepository

    Dưới đây là mã nguồn đã được tái cấu trúc hoàn chỉnh:

    ```java
    package com.securebank.repository;

    import com.securebank.repository.exception.UserNotFoundException;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;

    public class TransactionRepository {
        private final Connection connection;

        // Khởi tạo thông qua Constructor (Dependency Injection tốt cho việc Unit Test)
        public TransactionRepository(Connection connection) {
            this.connection = connection;
        }

        /**
        * Lưu giao dịch mới một cách an toàn. Kiểm tra tính toàn vẹn của khách hàng trước khi chèn dữ liệu.
        * * @param transactionId Mã định danh giao dịch (Khóa chính bảng transactions)
        * @param userId        Mã định danh khách hàng (Khóa ngoại tham chiếu bảng users)
        * @param amount        Số tiền giao dịch
        * @throws UserNotFoundException Nếu mã khách hàng không tồn tại trong cơ sở dữ liệu
        * @throws Exception             Nếu có các lỗi hệ thống hoặc SQL không mong muốn khác
        */
        public void saveTransaction(String transactionId, Long userId, double amount) throws Exception {

            // Bước 1: Kiểm tra tính tồn tại của userId bằng câu lệnh tối ưu hiệu năng SELECT COUNT(1)
            String checkUserSql = "SELECT COUNT(1) FROM users WHERE id = ?";

            try (PreparedStatement checkPs = connection.prepareStatement(checkUserSql)) {
                checkPs.setLong(1, userId);

                try (ResultSet rs = checkPs.executeQuery()) {
                    if (rs.next()) {
                        int count = rs.getInt(1);
                        if (count == 0) {
                            // Khách hàng không tồn tại, chủ động chặn đứng luồng xử lý và ném lỗi nghiệp vụ sạch
                            throw new UserNotFoundException("Giao dịch thất bại: Khách hàng có ID [" + userId + "] không tồn tại trên hệ thống.");
                        }
                    }
                }
            }

            // Bước 2: Thực hiện INSERT dữ liệu an toàn sau khi điều kiện tiền quyết (Pre-condition) đã thỏa mãn
            String insertSql = "INSERT INTO transactions (id, user_id, amount) VALUES (?, ?, ?)";

            try (PreparedStatement insertPs = connection.prepareStatement(insertSql)) {
                insertPs.setString(1, transactionId);
                insertPs.setLong(2, userId);
                insertPs.setDouble(3, amount);

                insertPs.executeUpdate();
            }
        }
    }

    ```

    ***

    ## 🛠️ Các điểm cải tiến mang tính chất Production-Ready:
    1. **Chuyển đổi từ Lỗi Kỹ thuật sang Lỗi Nghiệp vụ:** Thay vì để Database ném ra một ngoại lệ JDBC thô (`JdbcSQLIntegrityConstraintViolationException`), ứng dụng hiện tại sẽ ném ra `UserNotFoundException`. Điều này giúp các tầng phía trên (như `@ControllerAdvice` trong Spring Boot) dễ dàng bắt lại để trả về mã lỗi HTTP `404 Not Found` hoặc thông báo tường minh bằng tiếng Việt ra giao diện cho người dùng cuối.
    2. **Khối Try-with-resources:** Đoạn mã sử dụng tính năng quản lý tài nguyên tự động cho cả `PreparedStatement` và `ResultSet`. Ngay khi khối lệnh kết thúc (dù thành công hay gặp lỗi), Java sẽ tự động đóng (`close()`) các tài nguyên này theo thứ tự ngược lại, ngăn chặn triệt để tình trạng treo hoặc cạn kiệt connection pool của Database.
    3. **Tối ưu truy vấn kiểm tra:** Câu lệnh `SELECT COUNT(1) FROM users WHERE id = ?` giúp Database kiểm tra sự tồn tại cực nhanh qua Index của khóa chính mà không cần tốn chi phí tải toàn bộ dữ liệu của bản ghi lên bộ nhớ.
