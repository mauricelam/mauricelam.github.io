"Leaky abstractions" is a term that is used in software design and API reviews, often to nudge the API designer to hide more of their implementation details away from users of those APIs in order to provide a cleaner interface. Searching on the Internet (and in my personal experience in API reviews), there seem to be a general sentiment that leaky abstractions are bad, and it signifies a poor job by the software designer to protect the purity of their abstractions.

However, if you read [Joel Spolsky's now famous blog post about leaky abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/), you can see that it's not what the term meant at all. On quite the contrary, the Law of Leaky Abstractions states that **All non-trivial abstractions, to some degree, are leaky.** An alternative name I would suggest for the law is "the Futility of Perfect Abstractions", because while ideal, perfect abstractions don't really exist. If they do exist, they would merely be alternative representations of the same thing.

Coming from the Java and JavaScript world, when I first started using Rust, one of the things that I find fascinating and frustrating at the same time is how leaky Rust crate tends to be. To give an example, here is the documentation for `std::vec`:

> A contiguous growable array type with heap-allocated contents, written `Vec<T>`.
>
> Vectors have _O_(1) indexing, amortized _O_(1) push (to the end) and _O_(1) pop (from the end).
>
> Vectors ensure they never allocate more than `isize::MAX` bytes.

While on one-hand, it is cool to know which operations are O(1), and that they are allocated on the heap, on the other hand I was thinking to myself: "Why do I need to know all of these? Can't Rust just abstract it all away from me so I don't have to think about it? I never have to think about whether things are on the stack or the heap in Java", while secretly also thinking that the Rust library writers have not thought hard enough about how to properly abstract away the implementation details.

Of course, that thinking is just me showing my ignorance. What `vec` is providing is a model (a contiguous growable array) together with a specific implementation that has specific, well-documented characteristics, so that you as the user, can decide based on your context whether those characteristics and tradeoffs are suitable for you. It doesn't save you from learning about what a growing vector is, or what the tradeoffs of stack allocations versus heap allocations are. But it does provide you with a well tested and documented implementation that works in a wide variety of situations.

This pattern extends to other parts of the Rust ecosystem as well, not just in `vec` or in the standard library. The good crates on crates.io all follow this pattern – short readme pages to get you started on how to use it, and long explanations on the working model and the tradeoffs involved. Instead of hiding details, it exposes them in a way that requires you to learn about, and to think hard about the tradeoffs involved. Instead of an abstraction, they give a model to think about the problem and pair that with the documentation for when the abstraction inevitably leaks.

I am sure there are good Java or JavaScript libraries that follow this pattern too, but as an ecosystem in general, the Rust crates has done an exceptionally good job in preparing me to deal with the abstraction leaks, both in the type constraints and in documentation. 
