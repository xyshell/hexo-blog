---
title: julia-note
date: 2020-07-09 13:31:27
tags: julia
toc: true
---

# Variable

## Stylistic Conventions

- Names of variables are in lower case.

- Word separation can be indicated by underscores ('_'), but use of underscores is discouraged unless the name would be hard to read otherwise.

- Names of Types and Modules begin with a capital letter and word separation is shown with upper camel case instead of underscores.

- Names of functions and macros are in lower case, without underscores.

- Functions that write to their arguments have names that end in !. These are sometimes called "mutating" or "in-place" functions because they are intended to produce changes in their arguments after the function is called, not just return a value.
