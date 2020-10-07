---
layout: post
title:  "Category Theory"
---

In this post, I will talk about the definition of category theory, how I think about it and why it's relevant (not much of this though).

## Sets and functions acing on them

Think about a collection of sets and all functions acting on them. Let me state some properties of these:

- **Composition**: We know that functions compose. Composition of functions $$f: A \to B$$ and $$g: B \to C$$ is written as $$g \circ f: A \to C$$ which is defined as $$(g \circ f)(x) = g(f(x)) \:\forall x \in A$$.
- **Associativity**: We know that function compositions are associative i.e. for functions $$f: A \to B$$, $$g: B \to C$$, $$h: C \to D$$; we have $$h \circ (g \circ f) = (h \circ g) \circ f$$ i.e. $$\forall x \in A, (h \circ (g \circ f))(x) = h(g(f(x))) = ((h \circ g) \circ f)(x)$$
- **Identity**: For each set $$A$$ we have a function $$id_A : A \to A$$ such that $$id_A(x) = x \:\forall x \in A$$

The sets and functions can be thought of as a directed graph where for each set, you have a node and for each function mapping set $$A$$ to $$B$$, you have an arrow going from node $$A$$ to node $$B$$ in your directed graph.

In the directed graph as described above, arrows (functions) compose, the composition is associative and there is an identity arrow (identity function) for each object (set). A category looks just like this picture but we abstract out sets and functions to objects and morphisms respectively.

## Category : Definition

A category $$C$$ consists of

- A collection[^1] of objects
- A collection of morphisms, or arrows, between the objects. Each morphism $$f$$ has a source object $$A$$ and a target object $$B$$, and we say "$$f$$ is a morphism from $$A$$ to $$B$$". We denote collection of all morphisms between two objects $$A$$ and $$B$$ as $$hom(A, B)$$.
- For every three objects $$A$$, $$B$$ and $$C$$, a binary operation $$\circ : hom(A, B) \times hom(B, C) \to hom(A, C)$$ called composition of morphisms.

The following conditions must hold:

- **Associativity**: If $$f: A \to B$$, $$g: B \to C$$ and $$h: C \to D$$ then $$h \circ (g \circ f) = (h \circ g) \circ f$$
- **Identity**: For every object $$A$$, there exists a morphism $$id_A : A \to A$$ called the identity morphism for $$A$$, such that for every morphism $$f : X \to A$$ and $$g : A \to X$$, we have
$$id_A \circ f = f$$ and $$g \circ id_A = g$$

## Thinking categorically

Observe that when we went from the directed graph of sets and functions to a category, we abstracted out set to an object. **While we knew the elements of a set, the object is atomic or irreducible** Then, one might ask, what is a morphism? For a function we knew what it does based on its action on elements of a set, and we distinguished functions based on that. A morphism or arrow has a source object and a target object. There is a composition operator defined on two compatible morphisms, and one can think of as also having equality operators for both objects and morphisms. And that's it. We don't know what a morphism does to an object, we can only reason about it by how it composes with other morphisms.

Reiterating the main point:

1. Objects are atomic
2. As a consequence, morphisms can be reasoned about only by how they compose with other morphisms

So, in category theory, morphisms and how they compose take the central stage. One can observe the difference by just observing the definition of identity function for a set changed when we defined the identity morphism for an object in a category.

Now let me clarify by saying that when you *are* working with a specific category (such as the category of sets and functions) you might know more about the objects, meaning they might have components, and you might know what the morphisms actually do. In that case, you can probably derive more results *but* the results derived for an abstract category would still hold. However, when you are working with an abstract category and have no more information, objects are atomic and the only thing going for you is the composition of morphisms.

## Relevance

Data types and functions acting on them can also be modelled as a category[^2] and that means, we can directly take ideas from category theory to programming. A lot of ideas in functional programming (specifically the Haskell programming language) have apparently come from Category Theory, and it is still a treasure trove for exploring. I had written in an [earlier post about Monads]({% post_url 2020-06-04-monads %}), which are also directly taken from the Monads defined in Category Theory. Category Theory also provides a very general framework whose results would be applicable to a whole lot of areas since a lot of things can be modelled as a category.

I can't do justice to the relevance of category theory because I'm still learning about it. For a gentle introduction geared towards programmers, I highly recommend Bartosz Milewski's series of [blog posts](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/) or the [book](https://github.com/hmemcpy/milewski-ctfp-pdf) made from his blog posts and the accompanying lectures which can be found on [Bartosz's Youtube channel](https://www.youtube.com/user/DrBartosz/playlists). I would highly recommend checking them out if you're interested, I have been going through them for a while now, though at a very slow pace

## Footnotes

[^1]: I've deliberately used the word collection instead of set because the collection could possibly be larger than a set. It can be shown that collection of all sets can't be a set (checkout [Russel's Paradox](https://en.wikipedia.org/wiki/Russell%27s_paradox)), so the collection of objects in a category is a [class](https://en.wikipedia.org/wiki/Class_(set_theory))
[^2]: Well, not really. Due to the presence of infinite loops, the structure formed isn't exactly a category. Search for it if you're curious but the discussion is fairly involved (I don't understand it myself either)
