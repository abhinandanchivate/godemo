# **TDD in Go: Complete Guide with Employee Management Example**

## **📚 TDD Theory Overview**

### **What is TDD?**
**Test-Driven Development (TDD)** is a software development methodology where:
- Tests are written **BEFORE** production code
- Development follows the **RED-GREEN-REFACTOR** cycle
- Each feature starts with a failing test

### **Why Use TDD in Go?**
| Benefit | Go-Specific Advantage |
|---------|----------------------|
| **Compiler Safety** | Go's compiler catches type errors early |
| **Fast Test Execution** | Go tests run in parallel, very fast |
| **Built-in Testing** | `testing` package is in standard library |
| **Interface Power** | Easy mocking with Go interfaces |
| **Concurrency Testing** | Native support for testing goroutines |

### **When to Use TDD in Go?**
```
✅ IDEAL FOR:
• Business logic/domain models
• API handlers and services
• Database repositories
• Utility packages
• Concurrency-heavy code

⚠️ LESS SUITABLE FOR:
• UI/rendering code
• Main initialization (func main)
• Generated code
• Simple one-off scripts
```

### **How TDD Works in Go?**
```
1. Write test in *_test.go file
2. Run `go test` → See failure (RED)
3. Write minimal code to pass test (GREEN)
4. Refactor while tests pass (BLUE)
5. Repeat for next behavior
```

### **Where in Go Architecture?**
```
┌─────────────────────────────────────┐
│          HTTP Handlers              │  ← Integration Tests
├─────────────────────────────────────┤
│          Service Layer              │  ← Unit Tests with Mocks
├─────────────────────────────────────┤
│       Domain Models                 │  ← Unit Tests (TDD Focus)
├─────────────────────────────────────┤
│       Data Access Layer             │  ← Integration Tests
└─────────────────────────────────────┘
```

---

## **👥 Example: Person & Employee Management**

### **Project Structure**
```
employee-tdd/
├── go.mod
├── domain/
│   ├── person.go
│   ├── person_test.go      # TDD for Person entity
│   ├── employee.go
│   └── employee_test.go    # TDD for Employee (extends Person)
└── service/
    ├── payroll.go
    └── payroll_test.go     # TDD for business logic
```

---

## **🔴🟢🔵 TDD Cycle Example: Person Domain**

### **Step 1: RED - Write Failing Test**
```go
// domain/person_test.go
package domain

import (
    "testing"
    "time"
)

func TestNewPerson_ValidInput_CreatesPerson(t *testing.T) {
    // Test FIRST - Person doesn't exist yet
    dob := time.Date(1990, 1, 1, 0, 0, 0, 0, time.UTC)
    
    person, err := NewPerson("John", "Doe", dob, "john@example.com")
    
    // This will fail: undefined: NewPerson
    if err != nil {
        t.Fatalf("expected no error, got %v", err)
    }
    
    if person.FirstName != "John" {
        t.Errorf("expected FirstName John, got %s", person.FirstName)
    }
    
    if person.Age() < 30 { // Dynamic calculation
        t.Errorf("expected age > 30, got %d", person.Age())
    }
}
```

**Run Test (RED Phase):**
```bash
$ go test ./domain/...
# FAIL: undefined: NewPerson
```

### **Step 2: GREEN - Minimal Implementation**
```go
// domain/person.go
package domain

import (
    "time"
)

type Person struct {
    FirstName string
    LastName  string
    BirthDate time.Time
    Email     string
}

func NewPerson(firstName, lastName string, birthDate time.Time, email string) (*Person, error) {
    // Minimal code to pass test
    return &Person{
        FirstName: firstName,
        LastName:  lastName,
        BirthDate: birthDate,
        Email:     email,
    }, nil
}

func (p *Person) Age() int {
    now := time.Now()
    years := now.Year() - p.BirthDate.Year()
    
    // Adjust if birthday hasn't occurred this year
    if now.YearDay() < p.BirthDate.YearDay() {
        years--
    }
    return years
}
```

**Run Test (GREEN Phase):**
```bash
$ go test ./domain/...
PASS
```

### **Step 3: REFACTOR - Add Validation & Improve**
```go
// Add validation and error handling
func NewPerson(firstName, lastName string, birthDate time.Time, email string) (*Person, error) {
    if firstName == "" || lastName == "" {
        return nil, ErrInvalidName
    }
    
    if birthDate.After(time.Now()) {
        return nil, ErrFutureBirthDate
    }
    
    // Simple email validation
    if !strings.Contains(email, "@") {
        return nil, ErrInvalidEmail
    }
    
    return &Person{
        FirstName: firstName,
        LastName:  lastName,
        BirthDate: birthDate,
        Email:     email,
    }, nil
}

// Add test for validation
func TestNewPerson_EmptyFirstName_ReturnsError(t *testing.T) {
    dob := time.Date(1990, 1, 1, 0, 0, 0, 0, time.UTC)
    
    _, err := NewPerson("", "Doe", dob, "john@example.com")
    
    if err != ErrInvalidName {
        t.Errorf("expected ErrInvalidName, got %v", err)
    }
}
```

---

## **💼 TDD Example: Employee Extends Person**

### **Employee Entity with TDD**
```go
// domain/employee_test.go
package domain

import (
    "testing"
    "time"
)

func TestNewEmployee_ValidInput_CreatesEmployee(t *testing.T) {
    dob := time.Date(1990, 1, 1, 0, 0, 0, 0, time.UTC)
    hireDate := time.Date(2020, 6, 1, 0, 0, 0, 0, time.UTC)
    
    emp, err := NewEmployee(
        "Jane", 
        "Smith", 
        dob, 
        "jane@company.com",
        "EMP001",
        "Software Engineer",
        hireDate,
        75000, // Salary in cents
    )
    
    if err != nil {
        t.Fatal(err)
    }
    
    if emp.EmployeeID != "EMP001" {
        t.Errorf("expected EmployeeID EMP001, got %s", emp.EmployeeID)
    }
    
    if emp.YearsOfService() < 2 {
        t.Errorf("expected >2 years service, got %d", emp.YearsOfService())
    }
}
```

```go
// domain/employee.go
package domain

import "time"

type Employee struct {
    Person               // Embedded Person
    EmployeeID   string
    Position     string
    HireDate     time.Time
    SalaryCents  int64   // Store in smallest currency unit
    Department   string
    IsActive     bool
}

func NewEmployee(
    firstName, lastName string,
    birthDate time.Time,
    email string,
    employeeID, position string,
    hireDate time.Time,
    salaryCents int64,
) (*Employee, error) {
    
    // First create the Person
    person, err := NewPerson(firstName, lastName, birthDate, email)
    if err != nil {
        return nil, err
    }
    
    // Validate employee-specific fields
    if employeeID == "" {
        return nil, ErrInvalidEmployeeID
    }
    
    if hireDate.After(time.Now()) {
        return nil, ErrFutureHireDate
    }
    
    if salaryCents <= 0 {
        return nil, ErrInvalidSalary
    }
    
    return &Employee{
        Person:      *person,
        EmployeeID:  employeeID,
        Position:    position,
        HireDate:    hireDate,
        SalaryCents: salaryCents,
        IsActive:    true,
    }, nil
}

func (e *Employee) YearsOfService() int {
    now := time.Now()
    years := now.Year() - e.HireDate.Year()
    
    if now.YearDay() < e.HireDate.YearDay() {
        years--
    }
    return years
}

// Business rule: Annual bonus calculation
func (e *Employee) CalculateBonus() int64 {
    if !e.IsActive {
        return 0
    }
    
    // Example: 10% bonus for employees with >5 years service
    if e.YearsOfService() > 5 {
        return e.SalaryCents / 10
    }
    return e.SalaryCents / 20 // 5% bonus otherwise
}
```

---

## **📊 TABLE-DRIVEN TESTS in Go**

### **What are Table-Driven Tests?**
A Go idiom where multiple test cases are defined in a slice/table and iterated over.

### **Why Use Table-Driven Tests?**
1. **Compact**: Many test cases in one function
2. **Maintainable**: Easy to add new test cases
3. **Clear**: Input/output pairs are explicit
4. **Consistent**: Same test structure for all cases

### **When to Use Table-Driven Tests?**
- Testing multiple input combinations
- Boundary value testing
- Error condition testing
- Parameter validation

### **How: Employee Validation Example**
```go
// domain/employee_test.go
func TestEmployeeValidation_TableDriven(t *testing.T) {
    baseDob := time.Date(1990, 1, 1, 0, 0, 0, 0, time.UTC)
    baseHireDate := time.Date(2020, 1, 1, 0, 0, 0, 0, time.UTC)
    
    tests := []struct {
        name        string
        firstName   string
        lastName    string
        email       string
        employeeID  string
        salaryCents int64
        wantErr     error
    }{
        {
            name:        "valid employee",
            firstName:   "John",
            lastName:    "Doe",
            email:       "john@company.com",
            employeeID:  "EMP001",
            salaryCents: 5000000, // $50,000
            wantErr:     nil,
        },
        {
            name:        "empty first name",
            firstName:   "",
            lastName:    "Doe",
            email:       "john@company.com",
            employeeID:  "EMP002",
            salaryCents: 5000000,
            wantErr:     ErrInvalidName,
        },
        {
            name:        "invalid email",
            firstName:   "John",
            lastName:    "Doe",
            email:       "invalid-email",
            employeeID:  "EMP003",
            salaryCents: 5000000,
            wantErr:     ErrInvalidEmail,
        },
        {
            name:        "zero salary",
            firstName:   "John",
            lastName:    "Doe",
            email:       "john@company.com",
            employeeID:  "EMP004",
            salaryCents: 0,
            wantErr:     ErrInvalidSalary,
        },
        {
            name:        "negative salary",
            firstName:   "John",
            lastName:    "Doe",
            email:       "john@company.com",
            employeeID:  "EMP005",
            salaryCents: -10000,
            wantErr:     ErrInvalidSalary,
        },
        {
            name:        "empty employee ID",
            firstName:   "John",
            lastName:    "Doe",
            email:       "john@company.com",
            employeeID:  "",
            salaryCents: 5000000,
            wantErr:     ErrInvalidEmployeeID,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := NewEmployee(
                tt.firstName,
                tt.lastName,
                baseDob,
                tt.email,
                tt.employeeID,
                "Developer",
                baseHireDate,
                tt.salaryCents,
            )
            
            // Check if error matches expected
            if err != tt.wantErr {
                t.Errorf("NewEmployee() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

### **Bonus Calculation Table Test**
```go
func TestCalculateBonus_TableDriven(t *testing.T) {
    baseDob := time.Date(1980, 1, 1, 0, 0, 0, 0, time.UTC)
    
    tests := []struct {
        name           string
        hireDate       time.Time
        salaryCents    int64
        isActive       bool
        expectedBonus  int64
    }{
        {
            name:          "senior active employee",
            hireDate:      time.Date(2015, 1, 1, 0, 0, 0, 0, time.UTC), // 8+ years
            salaryCents:   10000000, // $100,000
            isActive:      true,
            expectedBonus: 1000000,  // 10% = $10,000
        },
        {
            name:          "junior active employee",
            hireDate:      time.Date(2022, 1, 1, 0, 0, 0, 0, time.UTC), // 1 year
            salaryCents:   8000000, // $80,000
            isActive:      true,
            expectedBonus: 400000,  // 5% = $4,000
        },
        {
            name:          "inactive employee",
            hireDate:      time.Date(2015, 1, 1, 0, 0, 0, 0, time.UTC),
            salaryCents:   10000000,
            isActive:      false,
            expectedBonus: 0,
        },
        {
            name:          "exactly 5 years service",
            hireDate:      time.Date(2019, 1, 1, 0, 0, 0, 0, time.UTC), // 5 years
            salaryCents:   9000000, // $90,000
            isActive:      true,
            expectedBonus: 450000,  // 5% = $4,500
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            emp, err := NewEmployee(
                "Test",
                "Employee",
                baseDob,
                "test@company.com",
                "TEST001",
                "Tester",
                tt.hireDate,
                tt.salaryCents,
            )
            
            if err != nil {
                t.Fatal(err)
            }
            
            emp.IsActive = tt.isActive
            bonus := emp.CalculateBonus()
            
            if bonus != tt.expectedBonus {
                t.Errorf("CalculateBonus() = %d cents, want %d cents", bonus, tt.expectedBonus)
            }
        })
    }
}
```

---

## **🏗️ Architecture Flow with TDD**

### **Complete Employee Management System**
```
┌─────────────────────────────────────────────────────┐
│                HTTP Layer (REST API)                 │
│                 - Employee Handlers                  │
│                 [Integration Tests]                  │
├─────────────────────────────────────────────────────┤
│               Service Layer (Business Logic)         │
│            - PayrollService                          │
│            - EmployeeService                         │
│            [Unit Tests with Mocks]                   │
├─────────────────────────────────────────────────────┤
│                Domain Layer (Entities)               │
│            - Person                                  │
│            - Employee                                │
│            [Unit Tests - TDD FOCUS]                  │
├─────────────────────────────────────────────────────┤
│              Repository Layer (Data Access)          │
│            - EmployeeRepository                      │
│            [Integration Tests]                       │
└─────────────────────────────────────────────────────┘
```

### **Service Layer TDD Example**
```go
// service/payroll_test.go
package service

import (
    "testing"
    "employee-tdd/domain"
    "github.com/stretchr/testify/mock"
)

// Mock repository
type MockEmployeeRepository struct {
    mock.Mock
}

func (m *MockEmployeeRepository) FindActive() ([]*domain.Employee, error) {
    args := m.Called()
    return args.Get(0).([]*domain.Employee), args.Error(1)
}

func TestPayrollService_CalculateTotalPayroll(t *testing.T) {
    // Arrange
    mockRepo := new(MockEmployeeRepository)
    
    // Create mock employees
    emp1 := &domain.Employee{SalaryCents: 5000000} // $50,000
    emp2 := &domain.Employee{SalaryCents: 7500000} // $75,000
    
    mockRepo.On("FindActive").Return([]*domain.Employee{emp1, emp2}, nil)
    
    service := NewPayrollService(mockRepo)
    
    // Act
    total, err := service.CalculateMonthlyPayroll()
    
    // Assert
    if err != nil {
        t.Fatal(err)
    }
    
    expected := 12500000 // $125,000/year
    if total != expected {
        t.Errorf("CalculateMonthlyPayroll() = %d, want %d", total, expected)
    }
    
    mockRepo.AssertExpectations(t)
}
```

```go
// service/payroll.go
package service

import "employee-tdd/domain"

type EmployeeRepository interface {
    FindActive() ([]*domain.Employee, error)
}

type PayrollService struct {
    repo EmployeeRepository
}

func NewPayrollService(repo EmployeeRepository) *PayrollService {
    return &PayrollService{repo: repo}
}

func (s *PayrollService) CalculateMonthlyPayroll() (int64, error) {
    employees, err := s.repo.FindActive()
    if err != nil {
        return 0, err
    }
    
    var total int64
    for _, emp := range employees {
        total += emp.SalaryCents
    }
    
    // Convert annual to monthly (simplified)
    return total / 12, nil
}
```

---

## **🎯 Key Takeaways**

### **TDD Best Practices for Go:**
1. **Start with `*_test.go`** - Write test before code
2. **Use table-driven tests** for multiple scenarios
3. **Follow RED-GREEN-REFACTOR** strictly
4. **Test behavior, not implementation**
5. **Use interfaces for testability**
6. **Keep tests fast and isolated**

### **Employee Domain Specifics:**
1. **Use integer for money** (cents, not dollars)
2. **Embed Person in Employee** for code reuse
3. **Test business rules thoroughly** (bonuses, validations)
4. **Mock external dependencies** in service layer

### **Running Tests:**
```bash
# Run all tests
go test ./...

# Run with verbose output
go test -v ./...

# Run specific package
go test ./domain/...

# Run with coverage
go test -cover ./...

# Run table tests only
go test -run TestEmployeeValidation_TableDriven
```

This complete example shows **TDD from theory to practice** in Go, covering **Person/Employee domain** with **table-driven tests** and proper **architecture flow**.
