---
description: Go-specific trophy testing. Verify Go implementations against specs using table-driven tests, real database connections, and integration-focused testing.
---

# Go Trophy Command

Go-specific trophy testing methodology using Go's testing conventions and tooling.

## What This Command Does

1. **Ingest Specs** - Read OpenSpec WHEN/THEN scenarios
2. **Generate Go Tests** - Create table-driven integration tests
3. **Use Real Resources** - Test databases, HTTP clients
4. **Run Tests** - Execute with `go test`
5. **Report Results** - Show scenario pass/fail status

## When to Use

Use `/go-trophy` when:
- Verifying Go implementations against specs
- Creating integration tests for Go services
- Testing HTTP handlers and database operations
- Validating business logic in Go

## Go Trophy Patterns

### Table-Driven Integration Tests

```go
func TestUserRegistration(t *testing.T) {
    // Setup real test database
    db := setupTestDB(t)
    defer db.Close()

    tests := []struct {
        name     string
        input    UserInput
        wantErr  bool
        scenario string // Links to spec scenario
    }{
        {
            name:     "valid registration",
            input:    UserInput{Email: "test@example.com", Password: "SecurePass123!"},
            wantErr:  false,
            scenario: "auth/spec.md: User registers successfully",
        },
        {
            name:     "duplicate email",
            input:    UserInput{Email: "existing@example.com", Password: "SecurePass123!"},
            wantErr:  true,
            scenario: "auth/spec.md: Duplicate email rejected",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := RegisterUser(db, tt.input)

            if (err != nil) != tt.wantErr {
                t.Errorf("scenario %q: got error %v, wantErr %v",
                    tt.scenario, err, tt.wantErr)
            }
        })
    }
}
```

### Real Database Testing

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()

    db, err := sql.Open("postgres", os.Getenv("TEST_DATABASE_URL"))
    if err != nil {
        t.Fatalf("failed to connect to test database: %v", err)
    }

    // Run migrations
    if err := runMigrations(db); err != nil {
        t.Fatalf("failed to run migrations: %v", err)
    }

    // Cleanup on test end
    t.Cleanup(func() {
        db.Exec("TRUNCATE users, orders, payments CASCADE")
        db.Close()
    })

    return db
}
```

### HTTP Handler Testing

```go
func TestCreateUserHandler(t *testing.T) {
    db := setupTestDB(t)
    handler := NewUserHandler(db)

    // WHEN valid user data is submitted
    body := `{"email": "new@example.com", "password": "SecurePass123!"}`
    req := httptest.NewRequest("POST", "/users", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    rec := httptest.NewRecorder()

    handler.ServeHTTP(rec, req)

    // THEN user is created
    if rec.Code != http.StatusCreated {
        t.Errorf("expected status 201, got %d", rec.Code)
    }

    // AND response contains user ID
    var resp map[string]interface{}
    json.NewDecoder(rec.Body).Decode(&resp)
    if resp["id"] == nil {
        t.Error("expected response to contain user ID")
    }
}
```

### Mocking External Services Only

```go
// Mock external payment API - not our code
type mockPaymentGateway struct {
    chargeFunc func(amount int64, token string) (*Charge, error)
}

func (m *mockPaymentGateway) Charge(amount int64, token string) (*Charge, error) {
    return m.chargeFunc(amount, token)
}

func TestPaymentProcessing(t *testing.T) {
    db := setupTestDB(t) // Real database

    gateway := &mockPaymentGateway{
        chargeFunc: func(amount int64, token string) (*Charge, error) {
            return &Charge{ID: "ch_test", Status: "succeeded"}, nil
        },
    }

    service := NewPaymentService(db, gateway)

    // Test with real DB, mocked external gateway
    result, err := service.ProcessPayment(100, "tok_test")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result.Status != "completed" {
        t.Errorf("expected status completed, got %s", result.Status)
    }
}
```

## Test Organization

```
myproject/
├── internal/
│   ├── users/
│   │   ├── users.go
│   │   └── users_test.go      # Integration tests
│   └── payments/
│       ├── payments.go
│       └── payments_test.go
├── pkg/
│   └── calculator/
│       ├── calculator.go
│       └── calculator_test.go  # Unit tests (complex pure functions)
├── cmd/
│   └── server/
│       └── main.go
├── e2e/
│   └── critical_test.go        # E2E tests
└── testdata/
    └── fixtures/               # Test fixtures
```

## Running Go Trophy Tests

```bash
# Run all tests with race detection
go test -race ./...

# Run specific package tests
go test -v ./internal/users/

# Run with coverage
go test -cover -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Run integration tests only (by build tag)
go test -tags=integration ./...

# Run with timeout
go test -timeout 30s ./...
```

## Go-Specific Patterns

### Subtests for Scenarios

```go
func TestDocumentProcessor(t *testing.T) {
    t.Run("Scenario: PPTX file detected", func(t *testing.T) {
        result := DetectType("test.pptx")
        if result != "pptx" {
            t.Errorf("expected pptx, got %s", result)
        }
    })

    t.Run("Scenario: Unsupported file type", func(t *testing.T) {
        _, err := DetectType("test.xyz")
        if err == nil {
            t.Error("expected error for unsupported file type")
        }
    })
}
```

### Test Fixtures

```go
//go:embed testdata/sample.pptx
var samplePPTX []byte

func TestProcessDocument(t *testing.T) {
    // Write fixture to temp file
    tmpfile, err := os.CreateTemp("", "test-*.pptx")
    if err != nil {
        t.Fatal(err)
    }
    defer os.Remove(tmpfile.Name())

    if _, err := tmpfile.Write(samplePPTX); err != nil {
        t.Fatal(err)
    }
    tmpfile.Close()

    // Test with real file
    result, err := ProcessDocument(tmpfile.Name())
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result.PageCount != 5 {
        t.Errorf("expected 5 pages, got %d", result.PageCount)
    }
}
```

### Error Assertion

```go
func TestValidation(t *testing.T) {
    tests := []struct {
        name      string
        input     string
        wantErr   error
        scenario  string
    }{
        {
            name:     "empty email",
            input:    "",
            wantErr:  ErrEmptyEmail,
            scenario: "validation/spec.md: Empty email rejected",
        },
        {
            name:     "invalid format",
            input:    "not-an-email",
            wantErr:  ErrInvalidEmail,
            scenario: "validation/spec.md: Invalid format rejected",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.input)
            if !errors.Is(err, tt.wantErr) {
                t.Errorf("scenario %q: got error %v, want %v",
                    tt.scenario, err, tt.wantErr)
            }
        })
    }
}
```

## Trophy Test Report (Go)

```
=== RUN   TestUserRegistration
=== RUN   TestUserRegistration/valid_registration
=== RUN   TestUserRegistration/duplicate_email
--- PASS: TestUserRegistration (0.15s)
    --- PASS: TestUserRegistration/valid_registration (0.08s)
    --- PASS: TestUserRegistration/duplicate_email (0.07s)

Trophy Test Results:
✅ 8/10 scenarios verified
❌ 2 scenarios failed:
   - auth/spec.md: "Password reset expired" - Token not invalidated
   - users/spec.md: "Profile update" - Missing validation

PASS
coverage: 85.2% of statements
```
