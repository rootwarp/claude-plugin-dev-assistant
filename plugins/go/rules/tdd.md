# Test-Driven Development (TDD)

- Write tests BEFORE implementing functionality
- Follow the Red-Green-Refactor cycle:
  1. Write a failing test that defines expected behavior
  2. Write minimal code to make the test pass
  3. Refactor while keeping tests green
- Each test should test ONE behavior
- Use descriptive test names: `TestFunctionName_Scenario_ExpectedBehavior`
- Place tests in `*_test.go` files in the same package
- Use subtests (`t.Run`) for related test cases
- Mock external dependencies; use interfaces for testability
- Aim for meaningful coverage, not 100% coverage
- Run tests before committing: `go test ./...`
