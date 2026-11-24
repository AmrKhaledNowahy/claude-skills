# Before / after examples

## Guards first, happy path last
```java
// Before
public void approve(Long id) {
    Order o = repo.findById(id).orElse(null);
    if (o != null) {
        if (o.getStatus() == Status.PENDING) {
            o.setStatus(Status.APPROVED);
        }
    }
}

// After
@Transactional
public void approve(Long id) {
    Order order = repo.findById(id)
            .orElseThrow(() -> new BaseException(ErrorCode.ORDER_NOT_FOUND));
    if (order.getStatus() != Status.PENDING) {
        throw new BaseException(ErrorCode.ORDER_NOT_PENDING);
    }
    order.setStatus(Status.APPROVED);
}
```

## Enum behavior instead of if/else chains
```java
// Before
if (type.equals("GOLD")) rate = 0.2; else if (type.equals("SILVER")) rate = 0.1; else rate = 0;

// After
enum Tier {
    GOLD(new BigDecimal("0.20")), SILVER(new BigDecimal("0.10")), BASIC(BigDecimal.ZERO);
    private final BigDecimal discountRate;
    Tier(BigDecimal discountRate) { this.discountRate = discountRate; }
    BigDecimal discountRate() { return discountRate; }
}
```

## Named predicate instead of long condition
```java
// Before
if (r.getStatus() == Status.PENDING && r.getDate().isBefore(today) && !r.isHidden()) { ... }

// After
if (isPendingAndOverdue(r, today)) { ... }
```

## Parameter object instead of many parameters
```java
// Before
search(String name, String status, LocalDate from, LocalDate to, int page, int size)

// After
search(SearchRequest request, Pageable pageable)
```

## Do not swallow exceptions
```java
// Before
try { send(msg); } catch (Exception e) { }

// After
try { send(msg); }
catch (IOException e) { throw new BaseException(ErrorCode.SEND_FAILED, e); }
```

## Test name and ticket tag
```java
@Test
@DisplayName("PROJ-123")
void approve_whenAlreadyApproved_throwsBaseException() { ... }
```
