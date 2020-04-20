---
title: "Bazel, Go and Protobufs"
categories: bugs
---

Suppose you're setting up your first project using Bazel, written in Go and
using Protobufs as a data format.

It's easy to make mistakes. I did, and ended up with the following build error:

```
compilepkg: error running subcommand: exit status 2
/***/file.go:xx:xx: cannot use o (type *protopb.Object) as type protoreflect.ProtoMessage in argument to "google.golang.org/protobuf/proto".Unmarshal:
	*protopb.Object does not implement protoreflect.ProtoMessage (missing ProtoReflect method)
/***/file.go:yy:yy: cannot use o (type *protopb.Object) as type protoreflect.ProtoMessage in argument to "google.golang.org/protobuf/proto".Marshal:
	*protopb.Object does not implement protoreflect.ProtoMessage (missing ProtoReflect method)
```

In particular, the phrase "missing ProtoReflect method" was unknown[^1] to
Google at time of writing!

There is a common pitfall where one tries to pass a message object directly to
`proto.Marshal` or `proto.Unmarshal`. These functions require a pointer to your
message object, since they expect an interface that is defined on the pointer
receiver by the proto compiler. This seems very similar, but we are passing a
pointer as we should be.

So what's going on?

It turns out there is a new Go Protobuf API. The new APIv2 is the module at
[google.golang.org/protobuf](https://google.golang.org/protobuf), and is less
than two months old. The old module lives at
[github.com/golang/protobuf](https://github.com/golang/protobuf). The new one
compiles your proto messages to implement `protoreflect.ProtoMessage`, which is
expected by the marshaling functions.

However, the Bazel Go rules have not been upgraded to support APIv2 (see the
ticket [here](https://github.com/bazelbuild/rules_go/issues/2395), open at time
of posting). The Go build rules implicitly use the older module to compile your
protobufs, without you needing to declare this dependency.

So maybe you're coding away, and type something like `proto.Marshal`. Your
well-meaning vim plugin automatically adds an include for
`google.golang.org/protobuf/proto`. This module ends up in `go.mod`, and
sometime later you run Gazelle, which happily includes a dependency into your
`WORKSPACE`. Then you build your project. First the protos are compiled using
APIv1. Next your library is compiled, but fails because the marshaling
implementation expects an interface only defined on protos compiled using APIv2.
Without being aware of any of the above, have fun debugging!

The solution is to unwind all of this and use the old module until
`bazelbuild/rules_go` gains support for APIv2.

Managing complexity is hard! I don't really have any kind of point I'm trying to
make. Hopefully this will be a short-lived issue. In the meantime, maybe Google
will pick this up and it will help someone.

[^1]: Almost. There was a result in Chinese which seemed to imply that the fix was to `go install google.golang.org/protobuf/cmd/protoc-gen-go`. I had dismissed it because it looked like they weren't using Bazel, and I was convinced Bazel was the problem. Looking at the page more closely, it looks like it does actually contain the problem and answer!
