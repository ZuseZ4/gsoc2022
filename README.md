# gsoc2022
A summary of my activities during gsoc 2022. 

This project started with some final test fixes and the merging of 
https://github.com/EnzymeAD/Enzyme/pull/719, which introduces Tablegen usage in Enzyme.

This allowed an easier integration of new rules into Enzyme, even from new contributors:
https://github.com/EnzymeAD/Enzyme/pull/667
https://github.com/EnzymeAD/Enzyme/pull/793

Some of these rules could benefit from calling llvm intrinsic functions directly, rather than using libm calls.
Therefore support for them has been added in 
https://github.com/EnzymeAD/Enzyme/pull/807,
which allowed improving some of the previous rules, as well as the addition of new rules as part of the PR.

The tablegen integration did unfortunately cause multiple build issues. We were able to fix some of them in advance, 
however some of these reached users and caused downstream issues. Since Enzyme does support a large amount of LLVM versions
and aims to have low build requirements, I had to add some inconvenient cmake fixes as seen here: https://github.com/EnzymeAD/Enzyme/pull/719
Thanks to the users reporting those issues, as well as support from @wsmoses and @vchuravy we now should have fixed those issues.

Independently from these general rules, I have been working on adding explicit autodiff rules for blas calls via tablegen.
These can be used to handle optimized and parallelized BLAS libraries, without falling back to differentiate a generic and single-threaded BLAS
implementation. We had some BLAS specific rules in Enzyme, however they have been hard to read and required hundreds of lines of code for each single blas call.
By using tablegen to handle multiple blas calls with the same backend I manage to not only generate more uniform looking code,
but was also able to simplify it and make it much more readiable. That resulted in recognizing that the previous implementation
had a bug in the case of different stride values for vectors, which was fixed in https://github.com/EnzymeAD/Enzyme/commit/35be55ccc297c679d5ab3ff145f4c5d1a1f6a266.

The new tablegen based blas rules are currently blocked on some incorrect handling of a float scalar. float vectors as well as integer scalars are handled correctly,
as it is seen in the previous testcases. However, due to circular dependencies, we want to add multiple blas functions simultaneously. Given the existing
testcase and the correct handling of other types it shouldn't be too much work to fix this missing test. 

Beside of the blas tablegen we also have two other small usecases in these PRs: https://github.com/EnzymeAD/Enzyme/pull/836, https://github.com/EnzymeAD/Enzyme/pull/838.
There we handle simple OpenMP cases, as well as rules which don't completely overlap with the first set of tablegen rules. 
The main difference is that they don't need to specify complex rules to differentiate each possibly active input argument,
but in exchange have to be more precise about the modes under which they work and which input argument they require to be active.

For the ongoing mentoring I would like to thank @wsmoses, as well as @reikdas and @tansongchen, which implemented earlier Blas handling code by hand and therefore helped me by providing a good comparison implementation.
