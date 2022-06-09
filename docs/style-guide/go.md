# Go

### How to return a pointer rules

If we want to return a pointer we have two options:
- return only not nil value if we have no second returning value
- if we want to return nil in some cases, we should have the second returning value that we can check to define
  wether returning pointer is nil or not (second returning value may be error or boolen ok for example). This gives an opportunity to avoid extra if pointer != nil checks.

<table>
<tbody>
<tr><th>Always do</th></tr>
<tr><td>

```go
func getSomething(...) *something {
    // function has a straightforward flow without possible nil returning pointer
    return &something{...}
}

func getSomething(...) (*something, error) {
    // function has a conditional flow with possible nil returning pointer
    // error != nil means that *something is nil
    if ... {
        return nil, errors.New(...)
    }
    return &something{...}, nil
}
// or
func getSomething(...) (*something, bool) {
    // function has a conditional flow with possible nil returning pointer
    // bool == false means that *something is nil
    if ... {
        return nil, false
    }
    return &something{...}, true
}

```

</td></tr>
<tr><th>Instead of</th></tr>
<tr><td>

```go
func getSomething(...) *something {
    // function has a straightforward flow without possible nil returning pointer
    return &something{...}
}

func getSomething(...) *something {
    // function has a conditional flow with possible nil returning pointer
    if ... {
        return nil
    }
    return &something{...}
}

```

</td></tr>
</tbody></table>


### Pointer dereferencing
According to the internal [returning a pointer rules](https://github.com/edenlabllc/poc.bp.api/blob/master/playbook.md#how-to-return-a-pointer-rules) we can skip pointer != nil checks before dereferncing or using pointers's fields/methods
if we get it from internal function:

<table>
<tbody>
<tr><th>From fn with single returning pointer</th></tr>
<tr><td>

```go
func getSomething() *something {...}

func do(...) error {
    s := getSomething(...)
    // we trust that r != nil
    return printSomething(*r)
}
```

</td></tr>
<tr><th>From fn with second returning value</th></tr>
<tr><td>

```go
func getSomething() (*something, error) {...}

func do(...) error {
    s, err := getSomething(...)
    if err != nil {
        return err
    }
    // we trust that r != nil if err == nil
    return printSomething(*r)
}

// or

func getSomething() (*something, bool) {...}

func do(...) error {
    s, ok := getSomething(...)
    if !ok {
        return errors.New(...)
    }
    //we trust that r != nil if ok == true
    return printSomething(*r)
}
```

</td></tr>
</tbody></table>


But we should always check a pointer for not nil before dereferencing or using its properties if we get it from third party package's function. The only exception is when we receive both pointer and error. Currently we have an agreement to trust that if error is nil then the value is not nil for all third party packages. This allows us to avoid extra if pointer != nil checks. If you will find any examples that violates this rule, please tell us and we'll discuss it again

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func do(...) error {
    r := someThirdPartyPkg.CalculateRates(...)
    if r == nil {
        return errors.New("got nil rates")
    }
    return printRates(*r)
}
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func do(...) error {
    r := someThirdPartyPkg.calculateRates(...)
    // potential panic
    return printRates(*r)
}
```

</td></tr>

<tr><th>But</th></tr>
<tr><td>

```go
func do(...) error {
    r, err := someThirdPartyPkg.CalculateRates(...)
    if err != nil {
        return err
    }
    // we trust that r is not nil because err == nil
    printRates(*r)
}
```

</td></tr>
</tbody></table>


### Code order

The most common code order:

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
const c ...
var v ...
type i interface ...
type str struct ...
func newStr() *str ...
func (s *str) ... 
func f() ...
```

</td></tr>
</tbody></table>


### Local variables naming

We prefer short (acronym) local variable names with short life cycle.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func do(...) {
    la := newLoanApplicaion()
}
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func do(...) (err error) {
    loanApplicaion := newLoanApplicaion()
}
```

</td></tr>
</tbody></table>

But sometimes such approach leads to lower readability. Feel free to choose suitable alternative in such case.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func do(...) {
    sign := newLoanApplicaionParticipantDocument()
    // or docSign := newLoanApplicaionParticipantDocument()
}
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func do(...) (err error) {
    lapds := newLoanApplicaionParticipantDocumentSignature()
}
```

</td></tr>
</tbody></table>


### Interface naming

We like to use [go interface naming approach](https://golang.org/doc/effective_go#interface-names) without I-prefix.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
type JobRunner interface ...
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
type IJobRunner interface ...
```

</td></tr>
</tbody></table>


### If error != nil first

We like to check if error is not nil instead of check that error is nil.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func f(...) (*Value, error) {
    ...
    v, err := newValue()
    if err != nil {
        return nil, err
    }
    ...
    return v, nil
} 
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func f(...) (*Value, error) {
    ...
    v, err := newValue()
    if err == nil {
        ...
        return v, err
    }
    return nil, err
} 
```

</td></tr>
</tbody></table>


### Early returns, avoid else

We like use ```return, break/continue``` as early as posible and avoid else block whereve is possible.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
const minAvaliableInt = -999
func searchMin(...) int {
    values := searchValues(...)
    if len(values) == 0 {
        return minAvaliableInt
    }
    min := // find min value in slice
    if min < minAvaliableInt {
        return minAvaliableInt
    }
    return min
} 
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
const minAvaliableInt = -999
func searchMin(...) int {
    values := searchValues(...)
    var min int
    if len(values) != 0 {
        min = // find min value in slice
    } else {
        min = minAvaliableInt
    }
    if min < minAvaliableInt {
        min = minAvaliableInt
    }
    return min
}
```

### Local variables initialization

We like use ```:=``` syntax to initialize local variables.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func f(...) {
    s := "string value"
} 
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func f(...) {
    var s string = "string value"
}
```

</td></tr>
</tbody></table>

</td></tr>
</tbody></table>


### Local variable weak link

We don't like to declare variables that will be used only once.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
func f(...) string {
    return findLoanApplication(...).number
} 
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
func f(...) string {
    la := findLoanApplication(...)
    return la.number
} 
```

</td></tr>
</tbody></table>


### Constant groups prefix

We like to use prefix for constants that belong to one group.

<table>
<tbody>
<tr><th>Prefer</th></tr>
<tr><td>

```go
// HTTP headers
const (
	HeaderXRequestID      = "X-Request-ID"
	HeaderXRequestContext = "X-Request-Context"
)
```

</td></tr>
<tr><th>Over</th></tr>
<tr><td>

```go
// HTTP headers
const (
	XRequestID      = "X-Request-ID"
	XRequestContext = "X-Request-Context"
)
```

</td></tr>
</tbody></table>
