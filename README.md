# Go Linear Programming library

Golp is a library for moddeling and solving linear programming problems. It uses [GLPK](https://www.gnu.org/software/glpk/) for the actual number crunching and offers a slightly higher-level interface for problem modelling relative to the underlying library.
Since the intention is providing a simpler interface, the underlying API is not completely exposed. If there are features of the low-level library you'd like to see exposed by golp, please open an issue.

**Warning**: the API is currently not stable.

# Dependencies

You need the libglpk-dev (Debian variants) or glpk-devel (Red Hat variants) package installed in order to be able to compile golp.

# Installing

If you have a properly set up GOPATH, just run:

```bash
$ go get github.com/costela/golp/golp
```

# Known Problems

The underlying C library is not reentrant and therefore models cannot be safely solved by two separate goroutines.
The current version of the library does not attempt to shield the user from this limitation.

# Example usage

The model of the following problem:

```
Maximize:
  z = x1 + 2 x2 - 3 x3
With:
  0 <= x1 <= 40
  5 <= x3 <= 11
Subject to:
  0 <= - x1 + x2 + 5.3 x3 <= 10
  -inf <= 2 x1 - 5 x2 + 3 x3 <= 20
  x2 - 8 x3 = 0
```

can be expressed with Golp like this:

```go
package main

import (
    "github.com/costela/golp/golp"
    "math"
    "fmt"
)

func main() {
  model := golp.NewModel("some model", golp.Maximize)
  x1, _ := model.AddVariable("x1")
  x1.SetBounds(0, 40)
  x2, _ := model.AddVariable("x2")
  // alternatively, all variable-related information can be given at once:
  x3, _ := model.AddDefinedVariable("x3", golp.ContinuousVariable, 3, 5, 11)

  model.AddConstraint(0, 10, []*golp.Variable{x1, x2, x3}, []float64{-1, 1, 5.3})
  model.AddConstraint(math.Inf(-1), 20, []*golp.Variable{x1, x2, x3}, []float64{2, -5, 3})
  model.AddConstraint(0, 0, []*golp.Variable{x1, x3}, []float64{1, -8})
  ⋮
```

The model can than be solved and the resulting values can than be retrieved as follows:

```go
  ⋮
  result, _ := model.SolveSimplex() // you should check for errors

  fmt.Printf("z = %f\n", result.GetObjectiveValue())
  fmt.Printf("x1 = %f\n", result.GetValue(x1))
  ⋮
}

```
