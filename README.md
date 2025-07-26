# Playwright API Testing Framework

A comprehensive TypeScript-based API testing framework using Playwright Test for robust, scalable API automation.

## Features

- **TypeScript Support**: Full TypeScript integration with type safety
- **Modular Architecture**: Clean separation of concerns with helpers, models, and utilities
- **Multiple Environments**: Support for dev, staging, and production environments
- **Authentication**: Built-in auth helper with token management
- **Data Generation**: Comprehensive test data generation utilities
- **Validation**: Robust response validation and schema checking
- **Logging**: Structured logging with configurable levels
- **Reporting**: Multiple report formats (HTML, JSON, JUnit, Allure)
- **Parallel Execution**: Configurable parallel test execution
- **Fixtures**: Reusable test fixtures for consistent setup

## Project Structure

```
playwright-api-framework/
├── src/
│   ├── config/           # Environment and test configuration
│   ├── fixtures/         # Test fixtures for reusable setup
│   ├── helpers/          # API, auth, and utility helpers
│   ├── models/           # TypeScript interfaces and types
│   ├── tests/            # Test specifications organized by domain
│   └── utils/            # Logging, validation, and other utilities
├── test-data/            # Static test data files
├── reports/              # Generated test reports
└── Configuration files
```

## Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd playwright-api-framework
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Install Playwright browsers**
   ```bash
   npx playwright install
   ```

4. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

## Configuration

### Environment Variables

Create a `.env` file based on `.env.example`:

- `BASE_URL`: Default API base URL
- `NODE_ENV`: Environment (development/staging/production)
- `LOG_LEVEL`: Logging level (DEBUG/INFO/WARN/ERROR)
- Authentication credentials for each environment

### Playwright Configuration

The `playwright.config.ts` file contains:

- Test directory and global setup
- Reporter configuration
- Multiple projects for different environments
- Base URL and HTTP headers
- Timeout and retry settings

## Running Tests

### Basic Commands

```bash
# Run all tests
npm test

# Run tests with UI mode
npm run test:ui

# Run tests in headed browser
npm run test:headed

# Run tests in debug mode
npm run test:debug

# Show test report
npm run test:report
```

### Environment-Specific Tests

```bash
# Run tests for specific environment
npm run test:env:dev
npm run test:env:staging

# Run smoke tests
npm run test:smoke

# Run regression tests
npm run test:regression
```

### Test Filtering

```bash
# Run specific test file
npx playwright test user-crud.spec.ts

# Run tests matching pattern
npx playwright test --grep "@smoke"

# Run tests in specific project
npx playwright test --project="API Tests - Development"
```

## Writing Tests

### Basic Test Structure

```typescript
import { test, expect } from '@fixtures/api-fixture';
import { User } from '@models/response.model';
import { ResponseValidator } from '@utils/validators';

test.describe('API Tests', () => {
  test('should get user by ID @smoke', async ({ apiHelper }) => {
    const response = await apiHelper.get<User>('/users/1');
    
    ResponseValidator.validateStatusCode(response, 200);
    ResponseValidator.validateUser(response.data);
    expect(response.data.id).toBe(1);
  });
});
```

### Using Test Fixtures

The framework provides several fixtures:

- **apiHelper**: Generic API request helper
- **authHelper**: Authentication and token management
- **dataGenerator**: Test data generation utilities
- **apiContext**: Raw Playwright API request context

```typescript
test('authenticated request', async ({ apiHelper, authHelper }) => {
  // Login and get tokens
  await authHelper.login({ username: 'user', password: 'pass' });
  
  // Make authenticated request
  const response = await apiHelper.get('/protected-endpoint', {
    headers: authHelper.getAuthHeaders()
  });
  
  expect(response.status).toBe(200);
});
```

### Data Generation

```typescript
test('create user with generated data', async ({ apiHelper, dataGenerator }) => {
  const userData = dataGenerator.generateUser({
    email: 'custom@example.com' // Override specific fields
  });
  
  const response = await apiHelper.post<User>('/users', {
    data: userData
  });
  
  ResponseValidator.validateUser(response.data);
});
```

### Response Validation

```typescript
test('validate response structure', async ({ apiHelper }) => {
  const response = await apiHelper.get<User[]>('/users');
  
  // Validate status and headers
  ResponseValidator.validateStatusCode(response, 200);
  ResponseValidator.validateContentType(response, 'application/json');
  
  // Validate array response
  ResponseValidator.validateArrayResponse(
    response.data, 
    1, // min length
    10, // max length
    ResponseValidator.validateUser // item validator
  );
});
```

## Test Organization

### Test Tags

Use tags to organize and filter tests:

- `@smoke`: Critical functionality tests
- `@regression`: Full regression suite
- `@dev`: Development environment tests
- `@staging`: Staging environment tests
- `@negative`: Negative test cases
- `@security`: Security-focused tests
- `@load`: Performance/load tests

### File Organization

```
src/tests/
├── auth/
│   ├── login.spec.ts
│   └── token-refresh.spec.ts
├── users/
│   ├── user-crud.spec.ts
│   └── user-validation.spec.ts
└── posts/
    ├── posts.spec.ts
    └── comments.spec.ts
```

## Advanced Features

### Custom Helpers

Create domain-specific helpers:

```typescript
// src/helpers/user-helper.ts
export class UserHelper extends ApiHelper {
  async createUser(userData: Partial<User>): Promise<User> {
    const response = await this.post<User>('/users', { data: userData });
    return response.data;
  }
  
  async getUserPosts(userId: number): Promise<Post[]> {
    const response = await this.get<Post[]>(`/users/${userId}/posts`);
    return response.data;
  }
}
```

### Custom Validators

```typescript
// src/utils/custom-validators.ts
export class CustomValidator extends ResponseValidator {
  static validateBusinessRule(data: any): void {
    // Custom business logic validation
    expect(data.businessField).toBeDefined();
  }
}
```

### Environment Configuration

```typescript
// src/config/environments.ts
export const environments = {
  development: {
    baseUrl: 'https://dev-api.example.com',
    timeout: 30000,
    credentials: { /* ... */ }
  },
  // ... other environments
};
```

## Reporting

### Available Reports

1. **HTML Report**: Interactive test results
2. **JSON Report**: Machine-readable results
3. **JUnit Report**: CI/CD integration
4. **Allure Report**: Rich test reporting

### Viewing Reports

```bash
# Open HTML report
npx playwright show-report

# Generate Allure report (requires Allure CLI)
allure serve reports/allure-results
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: API Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run API tests
        run: npm test
        env:
          BASE_URL: ${{ secrets.API_BASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: reports/
```

## Best Practices

### Test Design

1. **Use descriptive test names** that explain the scenario
2. **Tag tests appropriately** for easy filtering
3. **Validate both positive and negative scenarios**
4. **Use data generators** for consistent test data
5. **Implement proper cleanup** for stateful tests

### Data Management

1. **Generate dynamic test data** to avoid conflicts
2. **Use fixtures** for reusable setup/teardown
3. **Externalize test data** in JSON files when appropriate
4. **Clean up created resources** after tests

### Error Handling

1. **Use appropriate assertions** with meaningful messages
2. **Validate error responses** thoroughly
3. **Handle async operations** properly
4. **Log important information** for debugging

### Performance

1. **Run tests in parallel** when possible
2. **Use appropriate timeouts** for different operations
3. **Monitor response times** as part of tests
4. **Optimize test data** to reduce execution time

## Troubleshooting

### Common Issues

1. **Connection Timeouts**
   - Check network connectivity
   - Verify base URL configuration
   - Increase timeout values if needed

2. **Authentication Failures**
   - Verify credentials in environment variables
   - Check token expiration handling
   - Validate auth headers format

3. **Test Data Conflicts**
   - Use unique identifiers in test data
   - Implement proper cleanup
   - Consider test isolation strategies

4. **Environment Issues**
   - Verify environment-specific configuration
   - Check API availability
   - Validate environment variables

### Debug Mode

```bash
# Run specific test in debug mode
npx playwright test user-crud.spec.ts --debug

# Run with verbose logging
LOG_LEVEL=DEBUG npm test

# Use headed mode to see browser interactions
npm run test:headed
```

## Contributing

1. Follow the established project structure
2. Add appropriate tests for new features
3. Update documentation for changes
4. Use consistent coding standards
5. Add proper error handling and logging

## License

This project is licensed under the MIT License - see the LICENSE file for details.
