---
layout: post
title: "Learning Go: Implicit Addressing and Deference in Method Calls"
description: ""
category:
tags: [Go]
---
{% include JB/setup %}

An interesting [sample Go program](https://groups.google.com/forum/?fromgroups=#!searchin/golang-nuts/reciever/golang-nuts/IA7JkkyZuO4/DdfJOxXMwQAJ) was discussed in Google group:

	package main
	import &quot;fmt&quot;
	type Foo struct {
		X float64
	}
	func (v *Foo) Neg() float64 {
		return -v.X
	}
	func main() {
		var v0 Foo = Foo{3}
		var v1 *Foo = &amp; v0
		var v2 **Foo = &amp; v1
		var v3 ***Foo = &amp; v2
		fmt.Println(
			v0.Neg(), //OK, due to Rule 2.
			v1.Neg(), //OK, the normal case.
			v2.Neg(), //OK, due to Rule 3.
			v3.Neg()) //v3.Neg undefined (type ***Foo has no field or method Neg)
	}

From this program ,we can see three rules about addressing and
deference in method calls:

  * [Rule 1](http://golang.org/doc/effective_go.html#pointers_vs_values):
    Methods can be defined for any named type that is not a pointer or
    an interface.

  * [Rule 2](http://golang.org/ref/spec#Calls) (implicit addressing):
    If x is addressable and &amp;x's method set contains m, x.m() is
    shorthand for (&amp;x).m().

  * Rule 3 (implicit dereference): `x.m()` might be a shorthand for
    `(*x).m()`, because Go has only "." but not "->".

There would be no way to call a method using a reciever type like
`***Foo`, because we cannot attach a method to a pointer type as
stated in Rule 1; so the furthest implicit dereference reaches
reciever type `**Foo`.
