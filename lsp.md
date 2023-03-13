https://github.com/elbywan/crystalline


- Memory usage is high due to the boehm GC behaviour and the crystal compiler
itself.

- Due to Crystal having a wide type inference system (which is incredibly
convenient and practical), compilation times can unfortunately be relatively
long for big projects and depending on the hardware. This means that the LSP
will be stuck waiting for the compiler to finish before being able to provide a
response. Crystalline tries to mitigate that by caching compilation outcome
when possible.

- Methods that are not called anywhere will not be analyzed, as this is how the
Crystal compiler works.
- The parser is not permissive, nor incremental which means that the features
will sometimes not work. It would involve a massive amount of work to change
that.
