# Hybrid Agent Example

## Objective

Demonstrate context-aware code review.

## Prompt

Use the Hybrid Agent pattern.

Review the following Spring Boot controller.

Perform:

1. Bug analysis
2. Security review
3. Performance review
4. Maintainability review
5. Generate improved code

```java
@RestController
public class OrderController {

    @Autowired
    private OrderService service;

    @GetMapping("/orders")
    public List<Order> getOrders() {
        return service.getAllOrders();
    }
}
```

## Expected Outcome

### Summary

### Security Findings

### Performance Findings

### Maintainability Findings

### Recommendations

### Improved Code

## Pattern Characteristics

* Quick detection
* Deep reasoning
* Context awareness
