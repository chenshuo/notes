# Bugs

## GCC

### [Bug 54812](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54812): `private` `=default` destructor is `public`

Delete expression doesn't respect access of defaulted destructor

```cpp
struct Base
{
 private:
  ~Base() = default;
};

int main()
{
  Base* p = new Base;
  delete p;
}
```

> [Quote:](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54812#c10) The bug affects protected destructors too, and they're commonly used for base classes to prevent deletion via pointer-to-base.

Fixed in 4.9.0, by [r203985](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=203985).
Test case added by [r204057](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=204057).

## Glibc

### `strstr(3)`

Since glibc 2.9, strstr(3) may use [Two Way algorithm](http://www-igm.univ-mlv.fr/~lecroq/string/node26.html) in some cases, but the impl. had several bugs:

- [Bug 12092](https://sourceware.org/bugzilla/show_bug.cgi?id=12092)
- [Bug 12100](https://sourceware.org/bugzilla/show_bug.cgi?id=12100)
- [Bug 14602](https://sourceware.org/bugzilla/show_bug.cgi?id=14602)
- More?


