# go-tokenauth
[![Build Status](https://travis-ci.org/Bplotka/go-tokenauth.svg?branch=master)](https://travis-ci.org/Bplotka/go-tokenauth) [![Go Report Card](https://goreportcard.com/badge/github.com/Bplotka/go-tokenauth)](https://goreportcard.com/report/github.com/Bplotka/go-tokenauth)


This package contains different useful Auth Sources that can return valid auth token for future use in a form of string.

It also contains tiny interface called `tokenauth.Source` that allows to create more generic (e.g HTTP, gRPC) Clients:

```go
type Source interface {
	// Name of the auth source.
	Name() string

	// Token allows the source to return a valid token for specific authorization type in a form of string.
	//
	// Example usage:
	// - filling Authorization HTTP header with valid auth.
	// In that case it is up to caller to properly save it into specific http request header (usually called "Authorization")
	// and add "bearer" prefix if needed.
	Token(context.Context) (string, error)
}
```

## Example usage:

The usefulness of this interface can be shown in this example, when we wrap http.RoundTripper to inject required auth:

```go
package example

import (
    "fmt"
    "net/http"
    
    "github.com/Bplotka/go-tokenauth"
    "github.com/Bplotka/go-tokenauth/direct"
)

func newExampleTripper(parent http.RoundTripper /* , <any configuration here> */) http.RoundTripper {
    auth := directauth.New("direct", "token1")
      // or:
      // oidcauth.New(...)
      // oauth2auth.New(...)
      // k8sauth.New(...)

    return &exampleTripper{
        parent: parent,
        auth: auth,
    }
}

type exampleTripper struct {
    parent http.RoundTripper
    auth tokenauth.Source
}

func (t *exampleTripper) RoundTrip(req *http.Request) (*http.Response, error) {
    token, err := t.auth.Token(req.Context())
    if err != nil {
        // handle err
    }
    
    req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", token))
    return t.parent.RoundTrip(req)
}

```

See ready to use [tripper](./http/tripper.go)