<pre class='metadata'>
Title: constexpr bit_cast for null pointers
Shortname: D####
Revision: 0
!Draft Revision: 0
Audience: LEWG, EWG
Status: D
Group: WG21
URL:
!Source: <a href="https://github.com/ecatmur/constexpr-bit-cast-nullptr/blob/main/paper.md">github.com/ecatmur/constexpr-bit-cast-nullptr/blob/main/paper.md</a>
!Current: <a href="https://htmlpreview.github.io/?https://github.com/ecatmur/constexpr-bit-cast-nullptr/blob/r0/D####R0.html">github.com/ecatmur/constexpr-bit-cast-nullptr/blob/r0/D####R0.html</a>
Editor: Ed Catmur, ed@catmur.uk
Markup Shorthands: markdown yes, biblio yes, markup yes
Abstract:
    bit_cast is non-constexpr for pointers and member pointers. We propose to relax this for null pointers.
Date: 2022-12-22
</pre>
<pre class='biblio'>
{
    "slack": {
        "title": "is there a type trait that tells me if a trivially constructible type can be value-initialized by memset(0) into its storage? - cpplang Slack",
        "href": "https://cpplang.slack.com/archives/C21PKDHSL/p1671657641805229"
    }
}
</pre>

## 1. Changelog

: v0
:: Initial submission

## 2. Motivation and Scope

Suppose we want to check whether a type's value-initialized representation is all zeros; this can be useful for optimization[[slack]]. We could write:

```c++
template<class T>
constexpr bool is_value_initialized_to_zero_v = []
{
    using A = std::array<unsigned char, sizeof(T)>;
    return A{} == std::bit_cast<A>(T{});
}();
```

But this won't work if `T` contains a subobject of pointer or pointer to member type.
(Null pointers are usually all zeros, but null pointer to member is -1 on Itanium.)
This makes very little sense; there's no harm in being able to find out the value representation of a null pointer at compile time.
The only conversions that are actually infeasible or unsafe at compile time are:
* converting a symbolic pointer to e.g. `intptr_t`;
* constructing a wild pointer;
* constructing a pointer to inaccessible member.

## 3. Impact On the Standard

This is a library extension, though in practice it is implemented through compiler builtins.

## 4. Technical specification

Amend 22.15.3 \[bit.cast] as follows:

<quote>
Remarks: This function is constexpr if and only if To, From, and the types of all subobjects <ins>`o`</ins>of To and From are types T such that:
* is_­union_­v<T> is false;
* is_­pointer_­v<T> is false<ins>, or `o` is a null pointer value</ins>;
* is_­member_­pointer_­v<T> is false<ins>, or `o` is a null member pointer value, or `o` is a subobject of From</ins>;
* is_­volatile_­v<T> is false; and
* T has no non-static data members of reference type.
</quote>

Update `__cpp_lib_bit_cast` to the year and month of adoption.

## 5. Acknowledgements
