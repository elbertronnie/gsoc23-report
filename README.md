# GSoC Final Report

## Project Goals

Organisation: [CCextractor Development](https://summerofcode.withgoogle.com/programs/2023/organizations/ccextractor-development)

Project Link: [Rewrite the base parts of lib_ccx to Rust](https://summerofcode.withgoogle.com/programs/2023/projects/SdPnVLaq)

Entire source code of CCExtractor was initially written in C. This was starting to become harder to maintain and also has too many footguns to step into. Beginning from 2021, this code was starting to be converted to rust starting with the 708 decoder, which was completed in the GSoC 2021. Then the hardsubx module was converted to rust in GSoC 2022. The only problem with the conversion was that the rust code was not idiomatic and filled with unsafe or extern "C" functions and structs that were imported from C using bindgen, disregarding majority of the benefits that rust provided. This essentially became my inspiration to build a foundation upon which CCExtractor source code could be rewritten using idiomatic rust.

## Project Implementation

I found the modules which had the least or no dependencies to other parts of the codebase and started to port them in order. I had decided to place the new rust code inside a new crate named as `lib_ccxr`, which is a normal rust library crate. This was because the already existing crate `ccx_rust` is a static library crate which does not allow doctests. This would also helped in seperating FFI code to idiomatic Rust code. 

Since the Rust approach and C approach to solve a problem are very different, there are 3 layers of indirection I had used for porting:

#### 1. The C-FFI layer

This layers will have function names same as defined in C but with the prefix of ccxr_. These functions will handle the conversion between C-native types and Rust-native types and then pass it on to the C-like Rust layer. These are the functions defined in the `ccx_rust` crate under `libccxr_exports` module.

#### 2. The C-like Rust layer

This layer will have function names same as defined in C but work with Rust-native types. The code written using these functions would still be procedural but in Rust, hence the name C-like Rust. The entire task of these functions will be to translate the procedural code to idiomatic Rust code. These are the functions defined in the c_functions.rs file in appropriate modules inside the `lib_ccxr` crate.

#### 3. The Idiomatic Rust layer

This layer will be the code written as one is supposed to write in idiomatic Rust. It will have complete documentation and tests. This code will will be situated in the `lib_ccxr` crate.

## Contributions

- [CCExtractor/ccextractor#1551](https://github.com/CCExtractor/ccextractor/pull/1551) Create `lib_ccxr` and `libccxr_exports`
- [CCExtractor/ccextractor#1552](https://github.com/CCExtractor/ccextractor/pull/1552) Add bits and levenshtein module in `lib_ccxr`
- [CCExtractor/ccextractor#1553](https://github.com/CCExtractor/ccextractor/pull/1553) Add log module in `lib_ccxr`
- [CCExtractor/ccextractor#1554](https://github.com/CCExtractor/ccextractor/pull/1554) Add encoding module in `lib_ccxr`
- [CCExtractor/ccextractor#1555](https://github.com/CCExtractor/ccextractor/pull/1555) Add constants module in `lib_ccxr`
- [CCExtractor/ccextractor#1556](https://github.com/CCExtractor/ccextractor/pull/1556) Add time units module in `lib_ccxr`
- [CCExtractor/ccextractor#1557](https://github.com/CCExtractor/ccextractor/pull/1557) Add net module in `lib_ccxr`
- [CCExtractor/ccextractor#1558](https://github.com/CCExtractor/ccextractor/pull/1558) Add timing module in `lib_ccxr`
- [CCExtractor/ccextractor#1559](https://github.com/CCExtractor/ccextractor/pull/1559) Add options module in `lib_ccxr`
- [CCExtractor/ccextractor#1560](https://github.com/CCExtractor/ccextractor/pull/1560) Add teletext module in `lib_ccxr`

## Work left to do

Certain parts of the codebase is marked with a comment or with `todo!()` if they depend on some code which is not yet converted to Rust. Such parts will have to be revisited once their dependencies are also converted to rust. One example of this is the `send_gui` function of log module which is dependant on the activity.c file.

Most of the functions have been intergated into C except for the teletext module. This should be the obvious next step after the teletext module is added.

## Challenges
Most of the challenges revolve around the differences between C and Rust like strings. Dealing with C-Strings in the FFI Code was probably the hardest part. But I was already prepared for that. What took me by surprise was the byte format used for sending captions across the network was also using C-style Strings in its data. It is not a deal breaker but something worth considering if a new byte format will ever be made.

Apart from strings, the original source code has a lot of global static data which is something that is Rust is known for hating. I have tried to convert global data into local wherever possible. The only place where I failed miserably was in the timing module, partly becasue I could not understand the code. `GLOBAL_TIMING_INFO` in the timing module is one of the worst parts of the new idiomatic rust code.

## Acknowledgements

I want to express my gratitude to my mentors **Carlos Fernandez Sanz** and **Punit Lodha** who gave me this amazing opportunity to contribute to their codebase.

Finally, thanks to Google for organizing such a great program that fosters the open source community and helps students from all over the world.
