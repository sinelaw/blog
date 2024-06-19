---
title: "Const when you need it"
date: "2015-10-01"
tags: 
  - "haskell"
  - "infernu"
  - "javascript"
---

_infernu uses row-type polymorphism to propagate read/write capabilities on record fields. Using row-type polymorphism to describe more than just which fields are present bears a vague resemblance to polymorphic constraints._

In C, a pointer to a field in a const struct is automatically const'ed:

```
struct foo { int field; };

void useInt(const int *);

int main(void) {

    const struct foo x;

    useInt(&x.field); // no warnings because &x.field is 'const int *'

    return 0;
}

```

Thus, a function that extracts a pointer to a (possibly deep) field from a const struct, will also return a const pointer:

```
const int *getField(const struct foo *x) {

    return &x->field;

}

```

(All the code compiles with \`-Wall\` and \`-Wextra\`)

But, what if I want to use \`getField\` on a non-const struct, to get an accessor to a field within it? Almost works:

```
struct foo y;
int *bla = getField(&y);
*bla = 2;

```

Uh oh. We get a warning:

```
warning: initialization discards ‘const’ qualifier 
from pointer target type [enabled by default]
     int *bla = getField(&y);
                ^

```

The compiler is angry because \`int \*bla\` should be \`**const** int \*bla\`. But we don't want that! We just want to get an accessor - a writable accessor - to a field in our not-const struct value.

C++ (not C) does have a non-solution: [const\_cast](http://en.cppreference.com/w/cpp/language/const_cast). That isn't what we want: it's unsafe. What we want is, if a **function doesn't get a const struct**, the 'non-constness' should propagate to the field accessor being returned (and vice versa: if the given struct was const, so should the accessor).

In fancier words, we need **const polymorphism**, which I imagine would be written with a 'constness type variable' `C` like this made-up syntax:

```
const<C> int *getField(const<C> struct foo *x) {
    return &x->field;
}

```

And then we would expect this to compile with no problems:

```
    struct foo y;
    int *bla = getField(&y);

```

...because, as 'y' is not const, ergo the pointer returned from getField is not pointing at a const.

Unfortunately, no such thing. We could represent this in a type system in a number of ways. One simple way is to say that constness is a constraint on a type (using something like Haskell's type classes). Another way is to have 'write into a field' be a kind of a capability that's part of the type.

The latter, write-capability approach is what I use in [Infernu](http://sinelaw.github.io/infernu.org/s/). Here there are no structs (it's JavaScript) but there are polymorphic records. The type system includes two flavors for each field label: Get and Set. If a field is only being read, the record (or object or row) that contains it only needs to have the 'Get' accessor for that field. Here's infernu's output for a simple example:

```
//  obj :  { subObj:  { val: Number } }
var obj = { subObj: { val: 3 } };

```

Our object is simple. The comment is what infernu infers, a reasonably simple type.

In the notation I (shamelessly) invented, read-only fields have a prefix 'get' in the type, and read/write fields don't have any prefix. So a read-only field 'bla' would be: `{ get bla : t }`. If 'bla' is required to be writable, the type is written as `{ bla : t }`. So in the above 'obj' example, we see that literal objects are by default inferred to be writable (type annotations would allow you to control that).

Next let's make a function that only reads 'subObj':

```
//       readSubObj : ( { get subObj: h | g} -> h)
function readSubObj(x) { return x.subObj; }

```

The type inferred says "readSubObj is a function, that takes an object with a **readable field subObj**, (hence "get subObj": it doesn't require the 'Set' capability!). subObj has any type 'h', and the function returns that same type, 'h'. (By the way, that '`| g`' means the passed object is allowed to contain also other fields, we don't care.)

Example of a nested read:

```
//       readVal : ( { get subObj:  { get val: d | c} | b} -> d)
function readVal(x) { return x.subObj.val; }

```

Now we need to 'get subObj' but subObj itself is an object with a readable field 'val' of type d. The function returns a 'd'.

We can use readSubObj on a writable object with no problems:

```
//  sub :  { val: Number }
var sub = readSubObj(obj);

```

When infernu supports type annotations (eventually) one could take advantage of this type-system feature by marking certain fields 'get'.

While this isn't exactly the same as the problem we discussed with C const pointers, the same idea _could_ be used to implement polymorphic constness.

The main idea here is that ideas from row-type polymorphism can be used to implement a certain kind of 'capabilities' over types, constraints that are propagated. This may be a nicer way to implement (some kind of) [polymorphic constraints](http://stackoverflow.com/questions/12718268/polymorphic-constraint).

(For example, in a language that supports row extension/reduction, a function `{ x : Int | r } -> { | r }` would retain the unspecified constraints from 'r'. I'm sure there are more interesting examples.)

If you can refer me to something like this, please do!
