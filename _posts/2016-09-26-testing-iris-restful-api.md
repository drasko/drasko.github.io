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


Mainflux project adopts [TDD](https://en.wikipedia.org/wiki/Test-driven_development) approach, and Go is a great language with powerful [testing](https://golang.org/pkg/testing/) capabilities.
