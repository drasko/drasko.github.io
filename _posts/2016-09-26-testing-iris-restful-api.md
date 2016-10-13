---
layout: post
published: true
mathjax: false
featured: false
comments: true
title: Testing Iris RESTful API
---
## Iris and Mainflux
![](https://hd.unsplash.com/photo-1466027397211-20d0f2449a3f)
[Iris](https://github.com/kataras/iris) is a HTTP framework written in Go. We have choosen Iris as HTTP API protocol development framework for [Mainflux](https://github.com/Mainflux/mainflux) IoT platform because of it's impressive bechmarks and simplicity. As robustness and consequently test coverage is very important for any serious and professional project, this article explains Iris RESTful API testing techniques on the example of Mainflux server.

>**EDIT 1** - Due to the needs for leanest approach possible, Mainflux project decided to strip Iris 
>(and any other framework) and stick with bare Go standard lib for HTTP development.

>**EDIT 2** - Interesting discussion on this subject can be found on [Reddit](https://www.reddit.com/r/golang/comments/54yvdq/testing_iris_restful_api/)

Mainflux project adopts [TDD](https://en.wikipedia.org/wiki/Test-driven_development) approach, and Go is a great language with powerful [testing](https://golang.org/pkg/testing/) capabilities - so it is well suited for writing test scenarios in an organized and well designed manner.

## Go Testing Framework
Go `Testing` framework is nicely explained [here](https://golang.org/doc/code.html#Testing).
In short, recipe for testing `<my_module>.go` is following:

- Create `<my_module>_test.go` and make it belong to the same package (it is actually package we are testing, not only this particular module)
- In your test, use `import "testing"`
- Define function `func TestSomething(t *testing.T)`
- Optionally, if some prerequisite set-up is needed you can use `func TestMain(m *testing.M)`

As explained [here](https://blog.golang.org/examples), tests can be run via `go test` in the current dir. In order to test the whole project, Travis for example uses:

```bash
go test -v ./...
```

So this is the way you should test your project from the project root.

## Testing Go HTTP RESTful APIs
Usually, for testing HTTP servers Go already provides testing library called [httptest](https://golang.org/pkg/net/http/httptest/). Some examples of using this library can be found [here](https://gist.github.com/cespare/4992458), [here](http://www.markjberger.com/testing-web-apps-in-golang/), [here](https://groups.google.com/forum/#!topic/golang-nuts/GWYSt70uvlQ) or [here](https://blog.pivotal.io/labs/labs/a-rubyist-leaning-go-testing-http-handlers). Or even [here](https://elithrar.github.io/article/testing-http-handlers-go/).

However, Iris does not use Go's `net/http` - it uses [fasthttp](https://github.com/valyala/fasthttp) (look at the imports [here](https://github.com/kataras/iris/blob/master/http.go)). This means that you can not use `httptest` to test the server.

[Gin Gonic](https://github.com/gin-gonic/gin) for example uses `net/http` (as can be seen in imports [here](https://github.com/gin-gonic/gin/blob/develop/gin.go), so you can write tests like [this](https://github.com/gin-gonic/gin/issues/549#issuecomment-203419679).

[@kataras](https://github.com/kataras), Iris author, reccomends usage of the [httpexpect](https://github.com/gavv/httpexpect) testing framework for testing Iris-based HTTP servers - please refer to [this link](https://github.com/kataras/iris#testing).

### Httpexpect, Fasthttp and Iris
This is a hard part ;). Let's go slowly in code disecting.

In [this](https://github.com/gavv/httpexpect/blob/master/example/iris.go) `httpexpect` example we can see that func `IrisHandler()` is returning `fasthttp.RequestHandler`, which is `api.Router`, where `api` is `iris.New()`, i.e. Iris instance. Then [here](https://github.com/gavv/httpexpect/blob/master/example/iris_test.go) in `irisTester()` function we use this handler (`Router` of the Iris instance) to create `httpexpect` instance.

Further, when we look `FrameworkAPI` interface over [here](https://github.com/kataras/iris/blob/master/iris.go), we can see that this structure contains member function `Tester()`, defined in the following way: `Tester(*testing.T) *httpexpect.Expect`, which is basically a getter which fetches `testFramework` member of the structure `Framework`. This member is initialized via function `NewTester()` in which `httpexpect.Expect` instance is created based on `api.Router`.

This means that every Iris server already have member `testFramework` which is correct `httpexpect.Expect` instance for our fasthttp handler. It is static member (begins with lowercase letter, thus not reachable from other packages) and can be obtained via `api.Tester()` getter.

## Mainflux Iris Test Example
Mainflux `http_server_test.go` uses `iris.Tester(t)` to fetch correct `httpexpect` instance of the current Iris Framework started in `HttpServer()` gorutine just before.

Code of the test function is following:

```go
func TestServer(t *testing.T) {
	// Config
	var cfg config.Config
	cfg.Parse()

	go HttpServer(cfg)

	// prepare test framework
	if ok := <-iris.Available; !ok {
		t.Fatal("Unexpected error: server cannot start, please report this as bug!!")
	}


	e := iris.Tester(t)
	r := e.Request("GET", "/status").Expect().Status(iris.StatusOK).JSON()
	fmt.Println("%v", r)
}
```

One more thinng should be noted - and that is the use of `Available` channel. It is explained [here](https://github.com/kataras/iris/blob/master/iris.go), where we can see the comment in the code:

```go
// Available is a channel type of bool, fired to true when the server is opened and all plugins ran
// never fires false, if the .Close called then the channel is re-allocating.
// the channel remains open until you close it.
//
// look at the http_test.go file for a usage example
Available chan bool
```

In test function we use this flag to block (we are syncing on the channel) untill HTTP server is started. Then we can continue with the test (i.e. sending HTTP requests).


## Data Mocking, Interfaces and DockerMock
One more non-trivial thing makes Mainflux server testing more complicated - and that is the use of database. Usual practice during the test phase is that databse responses are mocked (stubbed). Similar approach is taken for other external libraries. This way we focus to testing of our module, simulating the responses from external functions (like database calls).

For this mocking strategy, Go uses interfaces - as explained [here](http://relistan.com/writing-testable-apps-in-go/) or [here](https://medium.com/@matryer/5-simple-tips-and-tricks-for-writing-unit-tests-in-golang-619653f90742#.b05i7m1om).

Interfaces are very important concept in Go, and be sure to know them. There is very good [article](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go) that will help you understand them well.

Never the less, mocking all Mongo functions is not a trivial job. A more realistic and modern way of this kind of testing can be achieved using Docker, as described in [this article](http://developers.almamedia.fi/painless-mongodb-testing-with-docker-and-golang/). Following this principle, a [dockertest](https://github.com/ory-am/dockertest) project implements dockerized testing environment that firs perfectly for Mainflux use-case and can be used even with Travis CI.

Mainflux [http_server_test.go](https://github.com/Mainflux/mainflux/blob/master/servers/http_server_test.go) uses `dockertest` which is initialized in the `TestMain()` function. This function is Go testing framework's special function and you can read more about it [here](https://golang.org/pkg/testing/)

## Conclusion
In the conclusion of this short tutorial, I hope that it brings better unerstanding of several concepts needed for Iris HTTP server with MongoDB testing:

- Fasthttp
- Httpexpect
- `iris.Tester()` getter and `iris.Available` sync channel
- Dockertest
