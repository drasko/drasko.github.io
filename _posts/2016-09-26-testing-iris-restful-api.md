---
layout: post
published: true
mathjax: false
featured: false
comments: true
title: Testing Iris RESTful API
---
## Go Tests for Iris
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

As explained [here](https://blog.golang.org/examples), tests can be run via `go test` in the current dir. In order to test the whole project Travis for example uses:

```bash
go test -v ./...
```
