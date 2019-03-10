# Unit testing in Go

In this post we will talk about unit testing in Go. Before we begin, we should remember there are a lot of tools and  
different techniques writing tests. Here we will write the way we used it.

This post we would name as a beta version because many things could change.

Ok, let's begin.

## Libraries and dependencies

- [Testify](https://github.com/stretchr/testify) it's a library for writing tests and mocks
- [Gorilla mux](https://github.com/gorilla/mux) for building our router for our API endpoints

## Writing the test first

We have our `get_all.go` file and `get_all_test.go` file.

`Testify` has a `suite` module where we can write our setup and teardown methods. This is convenient because we could  
setup the test escenary and focus in our tests.

### `get_all_test.go`

```go
package users

import (
	"github.com/ervitis/golang-testing/routes"
	"github.com/stretchr/testify/mock"
	"github.com/stretchr/testify/suite"
	"net/http"
	"net/http/httptest"
	"testing"
)

// We define this struct where all the data used in the tests is defined  
// Take a look at the req and rec variables. Because we will test the get_all functionality we can  
// define this here and inside the setup function initialize it
type GetUsersTestSuite struct {
	suite.Suite
	server *routes.Server

	req *http.Request
	rec *httptest.ResponseRecorder
}

func (suite *GetUsersTestSuite) SetupTest() {
	suite.server = &routes.Server{Addr: "http://localhost", Port: "10000"}

	suite.req, _ = http.NewRequest(http.MethodGet, suite.server.Url(), nil)
	suite.rec = httptest.NewRecorder()
}

func (suite *GetUsersTestSuite) AfterTest(_, _ string) {}

// Inside the test, just instance the mocker struct data and mock the function used inside the request handler
func (suite *GetUsersTestSuite) TestGetAllUsersOk() {
	mockito := new(mocker)

	mockito.On("ReadData", mock.Anything).Return(mockUsers(), nil)

	h := ReqHandler{
		Reader: mockito,
	}

	h.GetAllUsers(suite.rec, suite.req)

	suite.Equal(http.StatusOK, suite.rec.Code)
}

func TestGetUsers(t *testing.T) {
	suite.Run(t, new(GetUsersTestSuite))
}
```

### `mocks.go`

Because we are using `testify` we will use its mock component. Let's look at it:

```go
package users

import (
	"encoding/json"
	"github.com/stretchr/testify/mock"
)

// Testify needs a struct type and use the Mock type for mocking the functions
type mocker struct {
	mock.Mock
}

// ReadData is the function we mock, it needs the exact number and type of parameters and also the return values too
// The return values from args are the ones we define inside the function Return() used in the test before.
func (m *mocker) ReadData(path string) ([]byte, error) {
	args := m.Called(path)
	b, _ := json.Marshal(args.Get(0))
	return b, args.Error(1)
}

func mockUsers() []byte {
	return []byte(`[{"id": 1, "name": "test", "surname": null, "email": "test@test.com", "gender": null, "country": "Spain"}]`)
}
```

## Use interfaces everywhere

We are using the interfaces for writing mocks and tests.  

With this we can test everything. As we have said before, maybe this is not the perfect technique, so any comments are  
welcome and we will update this post so it could be helpful for everyone.

## Source code

Is stored [here](https://github.com/ervitis/golang-testing) and it contains the following folders:

- apis: for API endpoints and routing
- config: for future configurations
- data: some JSON objects used in the project
- helpers: utilities
- server: utilities for server configuration