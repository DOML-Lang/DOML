# DOML Style Guide

This guide is meant to present a standard style, you of course could form your own style based upon your native language but this is more meant to guide you then instruct you.
> For example in C++ the standard is to have variables with lower camel-case and functions with upper camel-case, but in GO the standard is that public variables/functions are upper camel-case and private are lower camel-case

## Creation Assignments

```C
@ Blue = System.Color ...
```

- The new object name should be descriptive, upper camel-case, and should not just be a single letter like X/Y
  - The formal spec/readme will break this sometimes just to simplify things (i.e. `@ X = Y.Z` is simpler then `@ Blue = System.Color` and applies to all cases)
  - Should avoid the use of things like `My` or `A`/`The`
- The namespace (i.e. `System`) should be upper camel-case, and should by small and descriptive of the entire project, for example if you had a project called Bumble Porcupine you could either call it `Bumble` or `Porcupine`