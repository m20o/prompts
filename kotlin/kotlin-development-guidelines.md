# Kotlin Development Guidelines

## Prerequisites

- JDK 21 or higher
- Kotlin 1.9.25
- Spring Boot 3.4.4
- Gradle (the project uses the Gradle wrapper, so you don't need to install it separately)

## Building the Project

The project uses Gradle as the build tool. Here are the common build commands:

```bash
# Build the project
./gradlew build

# Clean and build the project
./gradlew clean build

# Run the application
./gradlew bootRun

# Build a specific module
./gradlew :{module name}:build

# Run tests for a specific module
./gradlew :{module name}:test
```

## Testing Information

### Testing Framework

The project uses JUnit 5 for testing, along with Kotlin's built-in testing utilities. Key testing dependencies include:

- JUnit 5 (jupiter)
- Kotlin Test JUnit 5 integration
- Spring Boot Test for integration testing
- Kotest for assertions and property-based testing
- Mockk for mocking

### Running Tests

```bash
# Run all tests
./gradlew test

# Run a specific test class
./gradlew test --tests "com.example.MyTest"

# Run a specific test method
./gradlew test --tests "com.example.MyTest.data class should have correct properties"
```

### Writing Tests

Tests should follow these guidelines:

1. Use the "Given-When-Then" pattern for test organization
2. Focus on behavior rather than implementation details
3. Limit assertions to 2 per test case
4. Do not prefix test methods with "test"
5. Use backticks for descriptive test method names

Example test:

```kotlin
@Test
fun `flight data class should have correct properties`() {
    // Given
    val airline = Airline("AA", "American Airlines")
    // ... other setup code

    // When
    val flight = Flight(
        flightId = "ABC123",
        flightNumber = "AA123",
        // ... other properties
    )

    // Then
    assertEquals("ABC123", flight.flightId)
    // ... other assertions
}
```

### Testing Coroutines

For testing suspending functions, use the `runTest` function from `kotlinx-coroutines-test`:

```kotlin
@Test
fun `test suspending function`() = runTest {
        // Given
        val repository = mockk<FlightRepository>()

        // When
        val result = withContext(Dispatchers.Default) {
            repository.findById("123")
        }

        // Then
        // assertions
    }
```

## Additional Development Information

### Kotlin Style Guidelines

1. Follow the [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
2. Keep functions short, single-purpose, and preferably written as expressions
3. Use meaningful and descriptive names for functions, classes, and variables
4. Avoid abbreviations unless they are widely understood
5. Prefer immutability - use `val` over `var` where possible
6. Use data classes for domain entities and DTOs

### Naming Conventions

1. Use camelCase for variables, functions, and properties
2. Use PascalCase for classes, interfaces, and types
3. Use uppercase with underscores for constants
4. Avoid Hungarian notation and unnecessary prefixes

### Domain Model Implementation

1. Domain entities should be implemented as immutable `data class` objects
2. Business logic should be encapsulated within domain services
3. Use value objects for concepts with no identity but with business value
4. Domain services should not depend on infrastructure or application layers
5. Domain exceptions should be specific and descriptive

### Coroutines Implementation

1. Always launch coroutines within appropriate scopes
2. Use `CoroutineScope` tied to application lifecycle for top-level coroutines
3. Avoid using `GlobalScope` as it can lead to memory leaks
4. Properly implement cancellation by checking `isActive` in long-running operations
5. Use the appropriate dispatcher based on the workload:
    - `Dispatchers.LOOM` for I/O operations
    - `Dispatchers.Default` for CPU-intensive tasks
6. You will use my dispatcher patterns (see below).

#### Recommended Dispatcher Pattern
```kotlin
val Dispatchers.LOOM: @BlockingExecutor CoroutineDispatcher
    get() = Executors.newVirtualThreadPerTaskExecutor().asCoroutineDispatcher()

object Dispatchers {
    val DEFAULT = Dispatchers.Default
    val CPU get() = DEFAULT
    val IO get() = Dispatchers.LOOM
    
    // to wrap IO operations (e.g. RestTemplate)
    suspend fun <T> blockingIO(block: suspend () -> T): T = 
        withContext(IO) { block() }
    
    // to wrap cpu-intensive operations (e.g. serialization)
    suspend fun <T> cpuIntensive(block: suspend () -> T): T = 
        withContext(CPU) { block() }
        
    // With timeout control
    suspend fun <T> blockingIOWithTimeout(
        timeoutMs: Long, 
        block: suspend () -> T
    ): T = withContext(IO) {
        withTimeout(timeoutMs) { block() }
    }
}
```

This pattern provides several benefits:
1. Clear intent communication through descriptive function names
2. Efficient use of virtual threads (Project Loom) for IO operations
3. Direct dispatcher context switching without unnecessary nesting
4. Optional timeout control for blocking operations
5. Consistent dispatcher strategy throughout the application

### Error Handling

1. Define specific domain exceptions for business rule violations
2. Use a global exception handler to translate exceptions to appropriate HTTP responses
3. Log exceptions appropriately, with different levels based on severity
4. Never throw generic exceptions from domain or application code
5. Handle coroutine exceptions properly using CoroutineExceptionHandler or try/catch

### Security Requirements

1. **Authentication & Authorization**
   - Implement proper JWT validation with signature verification
   - Use Spring Security with role-based access control
   - Enforce the principle of least privilege for all operations
   - Implement token refresh mechanisms with appropriate expiration times
   - Never store sensitive credentials in code or configuration files

2. **Data Protection**
   - Encrypt sensitive data at rest using industry-standard algorithms
   - Use HTTPS for all communications
   - Implement proper input validation at API boundaries
   - Use parameterized queries to prevent SQL injection
   - Apply content security policies to prevent XSS attacks

3. **Secure Coding Practices**
   - Follow OWASP Top 10 guidelines
   - Avoid exposing stack traces or detailed error messages to clients
   - Implement rate limiting for authentication endpoints
   - Use secure random number generators for sensitive operations
   - Regularly update dependencies to address security vulnerabilities

4. **Auditing & Compliance**
   - Log all authentication and authorization decisions
   - Maintain audit logs for sensitive operations
   - Never log sensitive information (PII, credentials, tokens)
   - Implement proper log rotation and retention policies
   - Ensure compliance with relevant regulations (GDPR, CCPA, etc.)

5. **Security Testing**
   - Perform regular security scans using tools like OWASP ZAP
   - Include security tests in the CI/CD pipeline
   - Conduct periodic penetration testing
   - Implement dependency vulnerability scanning
   - Document security test results and remediation actions

### Troubleshooting and Observability

1. **Logging Strategy**
   - Use structured logging with JSON format
   - Include correlation IDs in all logs for request tracing
   - Log at appropriate levels (ERROR for exceptions, INFO for business events, DEBUG for development)
   - Include contextual information (user ID, request URI, operation type)
   - Avoid logging sensitive information
   
   ```kotlin
   private val logger = KotlinLogging.logger {}
   
   // Good practice example
   logger.info { "Processing flight booking requestId=$requestId userId=$userId flightNumber=$flightNumber" }
   
   // Include exception details properly
   logger.error(exception) { "Failed to process payment for booking=$bookingId reason=${exception.message}" }
   ```

2. **Metrics Collection**
   - Use Micrometer to collect application metrics
   - Monitor key performance indicators:
     - Response time (95th and 99th percentiles)
     - Error rates by endpoint
     - Resource utilization (memory, CPU, connections)
     - Business metrics (bookings per hour, search volume)
   - Implement custom metrics for critical business operations
   
   ```kotlin
   private val bookingTimer = registry.timer("booking.creation.time")
   
   fun createBooking(request: BookingRequest): Booking {
       return bookingTimer.record<Booking> {
           // booking creation logic
       }
   }
   ```

3. **Health Checks**
   - Implement Spring Boot Actuator health endpoints
   - Create custom health indicators for critical dependencies
   - Include readiness and liveness probes for Kubernetes
   - Monitor external service dependencies
   - Implement circuit breakers for fault tolerance
   
   ```kotlin
   @Component
   class DatabaseHealthIndicator(private val dataSource: DataSource) : AbstractHealthIndicator() {
       override fun doHealthCheck(builder: Health.Builder) {
           try {
               val connection = dataSource.connection
               val query = connection.createStatement()
               query.execute("SELECT 1")
               builder.up()
           } catch (e: Exception) {
               builder.down().withException(e)
           }
       }
   }
   ```

4. **Debugging Tools**
   - Enable conditional debugging using Spring Profiles
   - Use distributed tracing with Sleuth and Zipkin
   - Create dedicated debug endpoints (protected by authentication)
   - Document common error scenarios and their solutions
   - Implement thread dumps and heap dump generation on demand
   
   ```kotlin
   @Profile("debug")
   @RestController
   @RequestMapping("/debug")
   @PreAuthorize("hasRole('ADMIN')")
   class DebugController {
       @GetMapping("/threads")
       fun dumpThreads(): Map<String, Any> {
           // Return thread dump information
       }
   }
   ```

5. **Failure Recovery**
   - Implement graceful degradation for non-critical services
   - Use retry policies with exponential backoff
   - Document recovery procedures for common failures
   - Implement automatic recovery where possible
   - Create runbooks for manual intervention scenarios

### API Design Principles

When designing APIs for the Flight Platform, follow Werner Vogels' 6 rules for good API design:

1. **APIs are Forever** - Once an API is released, you need to support it forever or risk breaking client integrations.
   Design with longevity in mind and be extremely careful about what you expose.

2. **Never Break Backward Compatibility** - Clients might not update their code when you update your API. Always
   maintain backward compatibility and use versioning when necessary.

3. **Work Backwards from Customer Use Cases** - Start with the customer's needs and work backwards to the API design.
   Create APIs that solve real customer problems rather than exposing internal system capabilities.

4. **Create APIs That Are Self-Describing and Have a Clear, Specific Purpose** - APIs should be intuitive and focused on
   specific functionality. They should be easy to understand without extensive documentation.

5. **Create APIs with Explicit and Well-Documented Failure Modes** - Define how your API behaves when things go wrong.
   Document all error conditions, status codes, and messages.

6. **Avoid Leaking Implementation Details at All Costs** - The API should hide the implementation details from the
   consumer. Don't expose internal structures, database schemas, or implementation-specific concepts.
