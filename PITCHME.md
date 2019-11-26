---?color=linear-gradient(175deg, black 80%, silver 20%)

@snap[north span-100 text-13 text-pink]
A story about porting in-house 3D engine to Vulkan
@snapend

@snap[west span-40 text-blue text-bold]
Olgierd Humeńczuk
@snapend

@snap[east span-60 text-07 text-gray]
BIO
@ul[list-square-bullets text-silver](false)
- Have been programming for past 18 years using different programming languages
- Have created multiple game engines including software rendering
- Have been working on high performance networking server and clients in C and C++
@ulend
@snapend

---?color=linear-gradient(92deg, #AAAAAA 50%, silver 50%)

@snap[north span-100 text-pink text-13 text-bold]
About OpenGL and Vulkan
@snapend

@snap[west span-50 text-white text-bold text-07]
OpenGL
@ul[list-square-bullets]
- OpenGL 1.0 year 1992
- Driver responsibility and overhead: high
- API: bunch of getters and setters for global state
- Debugging: get-last-error approach, verification during execution
@ulend
@snapend

@snap[east span-50 text-black text-bold text-07]
Vulkan
@ul[list-square-bullets]
- Vulkan 1.0 years 2015/2016
- Driver responsibility and overhead: low
- API: still functions but operates on entities a.k.a. named states
- Dubugging: special debug layers, verification only through debug layers
@ulend
@snapend

---?color=linear-gradient(92deg, #AAAAAA 50%, silver 50%)

@snap[north span-100 text-pink text-13 text-bold]
API comparition
@snapend

@snap[north-west span-100 text-white text-bold]
OpenGL
```cpp
glClearColor(1.0f, 0.0f, 0.0f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);
```
@snapend

@snap[south-east span-100 text-black text-bold]
Vulkan
```cpp
VkCommandBufferBeginInfo beginInfo = {};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
beginInfo.flags = VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT;
VkClearColorValue clearColor = { 164.0f/256.0f, 30.0f/256.0f, 34.0f/256.0f, 0.0f };
VkClearValue clearValue = {};
clearValue.color = clearColor;
VkResult res = vkBeginCommandBuffer(m_cmdBufs[i], &beginInfo);
vkEndCommandBuffer(m_cmdBufs[i]);
```
@snapend

---
@snap[north span-100]
### How about metaprogamming ?
@snapend

@snap[west span-50]
#### The bad
@ul
- Code complexity
- Error reporting
- Compile times
@ulend
@snapend

@snap[east span-50]
#### The good
@ul
- Type safety
- Code correctness
- More optimizations
@ulend
@snapend

---

### Language

Basics of metaprogramming

* S.F.I.N.A.E. [example](https://godbolt.org/z/sA01si)
* Parameter packs [example](https://godbolt.org/z/i8BeI2)
* Type lists and iteration [example](https://godbolt.org/z/p569Mp)

---

#### What kind of problems we can solve using metaprogramming ? (1/4)

Metaprogramming can be used:
* Whenever the choice or structure is limited and depends on type or types
    * Calling functions in some order described via type lists
    * Generating bindings for an API like a programming language or just a library

---

#### What kind of problems we can solve using metaprogramming ? (2/4)

Metaprogramming can be used:
* For generating data using math functions
    * Lookup tables ( trigonometric functions, textures )
    * Datastructures accessed at compile time

---

#### What kind of problems we can solve using metaprogramming ? (3/4)

Metaprogramming can be used:
* For selecting processing path based on type qualities
    * Using memcpy instead of assignment operator for types that are trivially copyable
    * Auto serialization/deserialization

---

#### What kind of problems we can solve using metaprogramming ? (4/4)

Metaprogramming can be used:
* For optimisation
    * Small buffer
    * Static polymorphism
    * Enabling SSE using proper data layout and accessors
* For type safe type erasure

---

### Tacit style programming

What is Tacit programming ?

Remember pipes ?

```bash
ls -al | grep -i "sdl" | sort
```

Functional programming approach:

```cpp
const auto result = funA( funB( funC( data ) ) );
```

---

### Tacit ( style ) metaprogramming

Few facts about TMP
* Designed and created by Odin Holmes
* Goal is to be the fastest metaprogramming library

[Example of basic vocabulary types used in TMP](https://godbolt.org/z/OyHhEw)

---
#### Old approach vs TMP ( 1/2 )
Example from Nova Engine:

```cpp
template < typename M >
struct message_traits;

template < typename ID, int I >
struct message_idx{
    using type = empty_type;
};

template < typename ID, int I = 0,
    typename T = typename message_idx<ID,I>::type >
struct message_gather{
    using type = typename mpl::append< mpl::list<T>,
        typename message_gather< ID, I + 1 >::type >;
};

template < typename ID, int I >
struct message_gather< ID, I, empty_type >{
    using type = typename message_gather< ID, I + 1 >::type;
};

template < typename ID >
struct message_gather< ID, 100, empty_type >{
    using type = mpl::empty_list;
};
```
---
#### Old approach vs TMP ( 2/2 )
Example from Nova Engine:

```cpp
template < typename M > struct message_traits;

template < typename ID, int I > struct message_idx {
    using type = empty_type;
};

namespace detail {
template < typename ID > struct i2type {
    template < int I >
    using predicate =
    typename conditional<
        is_same< typename message_idx< ID, I >::type, empty_type >::value >::
            template type< tml::nop< typename message_idx< ID, I >::type >,
                        tml::prepend< typename message_idx< ID, I >::type > >;
};
} // namespace detail

template < typename ID > struct message_gather
{
    using type = tml::call_f< tml::gen_n_types_by_index<
        100, detail::i2type< ID >::template predicate, tml::fold<> > >;
};
```

---

#### Example of using templates for debugging
OpenGL function binding

```cpp
template < typename R, typename... Args, typename... TArgs >
auto gl_call( R ( *func )( Args... ), TArgs&&... args ) -> R
{
    const auto e0 = glGetError(); R ret{0};

    if ( e0 == GL_NO_ERROR ) {
        ret = func( static_cast< Args >(
            std::forward< TArgs >( args ) )... ); }
    else { report_gl_dirty( e0 ); return ret; }

    const auto e1 = glGetError();
    if ( e1 != GL_NO_ERROR ) { report_gl_error( e1 ); }
    return ret;
}
```

---

### Example of using detection idiom for generating bindings

[Generating VAO description from type](https://godbolt.org/z/Qnpbyu)

---

### Example of using type erasure

[Multithread logger](https://godbolt.org/z/DrcyHG)

---

### Summary
* Use libraries !
* Use Tacit approach in order to divide complex functions into a set of simple ones
* Use proper naming
* Hide complex code behind simple to use interfaces

---

### Thank You!
Questions ?

