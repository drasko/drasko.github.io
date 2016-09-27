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
[Iris](https://github.com/kataras/iris) is a HTTP framework written in Go. We have choosen Iris as HTTP API protocol developemnt framework for [Mainflux](https://github.com/Mainflux/mainflux) IoT platform becaus of it's impressive bechmarks and simplicity. As robustness and consequently test coverage is very important for any serious and professionla project, this article explains Iris RESTful API testing techniques on the example of Mainflux server.


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

However, Iris does not use Go's `net/http` - it uses [fasthttp](https://github.com/valyala/fasthttp)(look at the imports [here](https://github.com/kataras/iris/blob/master/http.go)). This means that you can use `httptest` to test the server.

[Gin Goinc](https://github.com/gin-gonic/gin) for example uses `net/http` (as can be seen in imports [here](https://github.com/gin-gonic/gin/blob/develop/gin.go), so you can write tests like [this](https://github.com/gin-gonic/gin/issues/549#issuecomment-203419679).

[@kataras](https://github.com/kataras), Iris author, reccomends usage of the [httpexpect[(https://github.com/gavv/httpexpect) testing framework for testing Iris-based HTTP servers - please refer to [this link](https://github.com/kataras/iris#testing).

### Fasthttp
In [this](https://github.com/gavv/httpexpect/blob/master/example/iris.go) `httpexpect` example we can see that func `IrisHandler()` is returning `fasthttp.RequestHandler`, which is `api.Router`, where `api` is `iris.New()`, i.e. Iris instance. Then [here](https://github.com/gavv/httpexpect/blob/master/example/iris_test.go) in `irisTester()` function we use this handler to create `httpexpect` instance.

Further, when we look `FrameworkAPI` interface over [here](https://github.com/kataras/iris/blob/master/iris.go), we can see that this structure contains member function `Tester()`, defined in the following way: `Tester(*testing.T) *httpexpect.Expect`, which is basically a getter which fetches `testFramework` member of the structure `Framework`. This member is initialized via function `NewTester()`




## Data Mocking, Interfaces and DockerMock
