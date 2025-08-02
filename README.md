# Abstractable Benchmark Framework

A comprehensive, reusable benchmark framework for systematic performance testing that can be used across multiple Go libraries and packages.

## Overview

This framework provides:

- **Parameterized benchmarking** with input size scaling
- **Systematic test organization** using benchmark tables
- **Performance regression detection** with configurable thresholds
- **Memory profiling and allocation tracking**
- **Concurrency testing** under realistic conditions
- **Comprehensive reporting** with scaling analysis
- **Abstractable design** for use in any Go library

## Key Features

### 🚀 **Systematic Benchmark Organization**

```go
suite := NewBenchmarkTable("MyLibrary").
    WithCase("FastFunction", myFunction, arg1, arg2).
    WithCase("SlowFunction", myFunction, arg3, arg4).
    WithInputSizeScaling().
    WithConcurrency(1, 4, 8, 16).
    WithThreshold("FastFunction", 10.0, 0). // 10ns max, 0 allocs
    Build()
```

### 📊 **Input Size Scaling Analysis**

Automatically tests performance across different input sizes (10, 100, 1K, 10K, 100K) to identify:

- **O(1)** vs **O(n)** vs **O(n²)** complexity patterns
- Performance breaking points
- Memory scaling characteristics

### 🔍 **Performance Regression Detection**

```go
// Set performance thresholds
suite.SetThreshold("CriticalFunction", PerformanceThreshold{
    MaxNsPerOp:       5.0,    // 5ns maximum
    MaxAllocsPerOp:   0,      // Zero allocations
    TolerancePercent: 10.0,   // 10% variance allowed
})

// Compare against baselines
regressions := CompareResults(baseline, current, 10.0)
```

### 💾 **Memory Profiling Integration**

- Allocation tracking across input sizes
- Memory leak detection for long-running operations
- GC pressure analysis under load

### ⚡ **Concurrency Testing**

Test performance under realistic concurrent access patterns:

```go
suite.WithConcurrency(1, 2, 4, 8, 16, 32)
runner.RunConcurrencyBenchmarks(b)
```

## Quick Start

### 1. Basic Benchmark Suite

```go
func BenchmarkMyLibrary(b *testing.B) {
    suite := NewBenchmarkTable("MyLibrary").
        WithCase("Function1", myFunc1, "test input").
        WithCase("Function2", myFunc2, 42, "param").
        WithInputSizeScaling().
        Build()
    
    runner := NewBenchmarkRunner(suite)
    runner.RunStandardBenchmarks(b)
}
```

### 2. Advanced Configuration

```go
func BenchmarkAdvanced(b *testing.B) {
    generator := &DataGenerator{}
    
    suite := NewBenchmarkTable("AdvancedSuite").
        // Test cases with different scenarios
        WithCase("ValidInput", validateFunc, generator.GenerateStrings(100, 20)).
        WithCaseExpectingError("InvalidInput", validateFunc, "bad input").
        
        // Multiple scaling dimensions
        WithScaling("InputSize", 10, 100, 1000, 10000).
        WithScaling("Complexity", "simple", "medium", "complex").
        
        // Performance requirements
        WithThreshold("ValidInput", 50.0, 2).
        WithConcurrency(1, 4, 8).
        
        // Load baseline for regression testing
        WithBaseline("benchmarks/baseline.json").
        
        Build()
    
    runner := NewBenchmarkRunner(suite)
    
    // Run comprehensive benchmark suite
    b.Run("Standard", runner.RunStandardBenchmarks)
    b.Run("Scaling", runner.RunScalingBenchmarks) 
    b.Run("Concurrency", runner.RunConcurrencyBenchmarks)
    b.Run("Memory", runner.RunMemoryProfilingBenchmarks)
    
    // Generate reports
    report := runner.GenerateReport()
    SaveResults(runner.GetResults(), "benchmarks/results.json")
}
```

## Framework Components

### Core Types

#### `BenchmarkCase`

Defines a single test case with all parameters:

```go
type BenchmarkCase struct {
    Name         string
    Function     TestableFunction
    Args         []interface{}
    InputSize    int
    ExpectError  bool
    Setup        func()
    Teardown     func()
    Tags         []string
    Metadata     map[string]interface{}
}
```

#### `TestableFunction`

Any function that can be benchmarked:

```go
type TestableFunction func(args ...interface{}) error
```

#### `BenchmarkSuite`

Organizes related benchmarks:

```go
type BenchmarkSuite struct {
    Name        string
    Cases       []BenchmarkCase
    Scaling     []ScalingDimension
    Thresholds  map[string]PerformanceThreshold
    Baselines   map[string]BenchmarkResult
    Concurrency []int
}
```

### Utilities

#### `DataGenerator`

Generate test data at scale:

```go
generator := &DataGenerator{}
ints := generator.GenerateInts(10000, 1, 1000000)
strings := generator.GenerateStrings(1000, 50)
floats := generator.GenerateFloats(5000, 0.0, 100.0)
```

#### `ResultAnalyzer`

Analyze performance scaling:

```go
analyzer := NewResultAnalyzer(results)
scaling := analyzer.AnalyzeScaling()

for funcName, analysis := range scaling.Functions {
    fmt.Printf("%s complexity: %s\n", funcName, analysis.Complexity)
}
```

#### `ReportGenerator`

Create comprehensive reports:

```go
generator := NewReportGenerator(results)
markdown := generator.GenerateMarkdownReport()
```

## Usage Examples

### HTTP Client Library

```go
func BenchmarkHTTPClient(b *testing.B) {
    suite := NewBenchmarkTable("HTTPClient").
        WithCase("GET_Small", httpGet, "https://api.example.com/user/123").
        WithCase("GET_Large", httpGet, "https://api.example.com/users").
        WithCase("POST_JSON", httpPost, "https://api.example.com/users", jsonPayload).
        WithScaling("PayloadSize", 1024, 10240, 102400). // 1KB to 100KB
        WithConcurrency(1, 10, 100).
        WithThreshold("GET_Small", 10000000.0, 3). // 10ms max
        Build()
    
    NewBenchmarkRunner(suite).RunScalingBenchmarks(b)
}
```

### Database ORM

```go
func BenchmarkORM(b *testing.B) {
    suite := NewBenchmarkTable("ORM").
        WithCase("FindByID", findByID, 123).
        WithCase("FindMany", findMany, 100).
        WithCase("Create", createRecord, userData).
        WithCase("Update", updateRecord, 123, updateData).
        WithScaling("RecordCount", 10, 100, 1000, 10000).
        WithConcurrency(1, 5, 10, 20).
        Build()
        
    runner := NewBenchmarkRunner(suite)
    runner.RunConcurrencyBenchmarks(b)
}
```

### JSON Processing

```go
func BenchmarkJSONProcessor(b *testing.B) {
    generator := &DataGenerator{}
    
    suite := NewBenchmarkTable("JSONProcessor").
        WithCase("Marshal", jsonMarshal, testStruct).
        WithCase("Unmarshal", jsonUnmarshal, jsonString).
        WithCase("Validate", jsonValidate, jsonSchema, jsonData).
        WithScaling("ObjectSize", 10, 100, 1000, 10000).
        WithThreshold("Marshal", 1000.0, 5).
        Build()
    
    NewBenchmarkRunner(suite).RunMemoryProfilingBenchmarks(b)
}
```

## Integration with CI/CD

### Save Baseline Performance

```go
// Save current results as baseline for future comparisons
results := runner.GetResults()
SaveResults(results, "benchmarks/baseline.json")
```

### Regression Detection in CI

```go
// Load baseline and compare with current results
baseline, _ := LoadResults("benchmarks/baseline.json")
current := runner.GetResults() 
regressions := CompareResults(baseline, current, 15.0) // 15% tolerance

if len(regressions) > 0 {
    // Fail CI build or create alerts
    for _, reg := range regressions {
        log.Printf("Performance regression: %s is %.1f%% slower", 
            reg.Name, reg.TimeDiffPercent)
    }
}
```

### Generate Reports

```go
// Create markdown report for documentation
report := NewReportGenerator(results).GenerateMarkdownReport()
ioutil.WriteFile("PERFORMANCE.md", []byte(report), 0644)
```

## Best Practices

### 1. **Organize by Complexity**

Group benchmarks by expected performance characteristics:

```go
suite.AddCase("O1_ConstantTime", constantTimeFunc, data)
suite.AddCase("On_LinearTime", linearTimeFunc, data)  
suite.AddCase("On2_QuadraticTime", quadraticTimeFunc, data)
```

### 2. **Use Representative Data**

Generate realistic test data:

```go
// Don't use simple incrementing data
badData := []int{1, 2, 3, 4, 5}

// Use varied, realistic data
generator := &DataGenerator{}
goodData := generator.GenerateInts(1000, 1, 1000000)
```

### 3. **Test Both Success and Failure Paths**

```go
suite.WithCase("ValidInput", validator, validData).
     WithCaseExpectingError("InvalidInput", validator, invalidData)
```

### 4. **Set Appropriate Thresholds**

Base thresholds on production requirements:

```go
// For a web API endpoint that must respond in <100ms
suite.WithThreshold("APIEndpoint", 100000000.0, 50) // 100ms, 50 allocs
```

### 5. **Use Scaling Analysis**

Always test across multiple input sizes to understand scaling:

```go
suite.WithInputSizeScaling() // Tests 10, 100, 1K, 10K, 100K automatically
```

## Advanced Features

### Custom Scaling Dimensions

```go
suite.AddScalingDimension(ScalingDimension{
    Name: "Concurrency",
    Values: []interface{}{1, 2, 4, 8, 16, 32},
})

suite.AddScalingDimension(ScalingDimension{
    Name: "CacheSize", 
    Values: []interface{}{100, 1000, 10000, 100000},
})
```

### Stability Testing

```go
suite.Iterations = 10 // Run each benchmark 10 times
runner.RunStabilityBenchmarks(b) // Detects high variance
```

### Memory Leak Detection

```go
// The framework automatically tracks allocations and can detect
// growing memory usage patterns across iterations
runner.RunMemoryProfilingBenchmarks(b)
```

## Migration from Standard Benchmarks

### Before (Standard Go Benchmarks)

```go
func BenchmarkMyFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        MyFunction("test")
    }
}

func BenchmarkMyFunctionLarge(b *testing.B) {
    for i := 0; i < b.N; i++ {
        MyFunction(largeInput)
    }
}
```

### After (Framework)

```go
func BenchmarkMyFunction(b *testing.B) {
    suite := NewBenchmarkTable("MyLibrary").
        WithCase("SmallInput", wrapMyFunction, "test").
        WithCase("LargeInput", wrapMyFunction, largeInput).
        WithInputSizeScaling().
        Build()
    
    NewBenchmarkRunner(suite).RunScalingBenchmarks(b)
}

func wrapMyFunction(args ...interface{}) error {
    MyFunction(args[0].(string))
    return nil
}
```

## Contributing

This framework is designed to be:

- **Library-agnostic**: Works with any Go library
- **Extensible**: Easy to add new analysis types
- **Maintainable**: Clear separation of concerns
- **Performant**: Minimal overhead from the framework itself

Feel free to extend the framework for your specific use cases!
