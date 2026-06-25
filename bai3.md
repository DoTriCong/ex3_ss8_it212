# BÀI 3: Thực hành Refactor Clean Code - Refinement Process

## Vòng 1: Robustness & Clean Code
- **Mục tiêu:** Loại bỏ if-else lồng nhau sâu, áp dụng Guard Clauses (return early).
- **Thay đổi:** Đặt tên biến rõ nghĩa (`accounts`, `branchName`, `includeActiveOnly`).
- **Kết quả:**

```java
import java.util.List;

public class LedgerBalanceCalculator {

    public double calculateTotalBalance(List<Account> accounts, String branchName, boolean includeActiveOnly) {
        if (accounts == null || accounts.isEmpty()) {
            return 0;
        }

        double totalBalance = 0;
        for (Account account : accounts) {
            if (account == null) continue;
            if (!branchName.equals(account.getBranch())) continue;
            if (includeActiveOnly && !"ACTIVE".equals(account.getStatus())) continue;
            if (account.getBalance() <= 0) continue;

            totalBalance += account.getBalance();
        }
        return totalBalance;
    }
}
### Vòng 2: Maintainability & OOP
Mục tiêu: Tối ưu bằng Java 17 Stream API, dễ đọc, ngắn gọn.

Thêm: Annotation @Transactional(readOnly = true) nếu chạy trong Spring Boot.

Kết quả:

java
import java.util.List;
import org.springframework.transaction.annotation.Transactional;

public class LedgerBalanceCalculator {

    @Transactional(readOnly = true)
    public double calculateTotalBalance(List<Account> accounts, String branchName, boolean includeActiveOnly) {
        if (accounts == null || accounts.isEmpty()) {
            return 0;
        }

        return accounts.stream()
                .filter(account -> account != null)
                .filter(account -> branchName.equals(account.getBranch()))
                .filter(account -> !includeActiveOnly || "ACTIVE".equals(account.getStatus()))
                .mapToDouble(Account::getBalance)
                .filter(balance -> balance > 0)
                .sum();
    }
}
#### Vòng 3: Logging & Context Tuning
Mục tiêu: Tích hợp logging để ghi nhận lỗi số học hoặc dữ liệu bất thường.

Thêm: Lombok @Slf4j để log cảnh báo khi gặp dữ liệu không hợp lệ.

Kết quả:

java
import java.util.List;
import org.springframework.transaction.annotation.Transactional;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class LedgerBalanceCalculator {

    @Transactional(readOnly = true)
    public double calculateTotalBalance(List<Account> accounts, String branchName, boolean includeActiveOnly) {
        if (accounts == null || accounts.isEmpty()) {
            log.warn("Account list is null or empty for branch {}", branchName);
            return 0;
        }

        return accounts.stream()
                .filter(account -> {
                    if (account == null) {
                        log.warn("Encountered null account in branch {}", branchName);
                        return false;
                    }
                    return true;
                })
                .filter(account -> branchName.equals(account.getBranch()))
                .filter(account -> {
                    if (includeActiveOnly && !"ACTIVE".equals(account.getStatus())) {
                        log.info("Skipping inactive account with ID {}", account.getId());
                        return false;
                    }
                    return true;
                })
                .mapToDouble(account -> {
                    if (account.getBalance() <= 0) {
                        log.warn("Invalid balance {} for account ID {}", account.getBalance(), account.getId());
                        return 0;
                    }
                    return account.getBalance();
                })
                .sum();
    }
}
Kết luận
Vòng 1: Loại bỏ if-else lồng nhau, đặt tên biến rõ nghĩa.

Vòng 2: Tối ưu bằng Stream API, annotation @Transactional(readOnly = true).

Vòng 3: Tích hợp logging với Lombok @Slf4j, giúp hệ thống dễ giám sát và bảo trì.
