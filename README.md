# llvm-devmtg15-poster

Code Clone Detection in Clang Static Analyzer poster for [LLVM Developer's Meeting 2015](http://llvm.org/devmtg/2015-10/).

[Compiled PDF version](https://omtcyfz.github.io/assets/code-clone-detection-poster.pdf) is availible on my Github static site.

## Description

This repository contains LaTeX sources for the poster I wrote for LLVM Developer's Meeting 2015.
LLVM Developers' Meeting is the largest conference for compiler specialists from all over the world held every year. Doznes of Google,
Apple and Intel engineers are attending the conference and exchanging valuable experience.

The results of this research resulted in `alpha.clone.CloneChecker` appearance in
[Clang Static Analyzer](http://clang-analyzer.llvm.org/index.html). This check is capable of detecting a part of what the original
imporementation was able to detect, but is more stable and production-ready.

To see what it's capable of see related [unit tests](https://github.com/omtcyfz/clang/tree/master/test/Analysis/copypaste).

Clang Static Analyzer is shipped with Clang binary and if one wants to see what the upstream implementation can detect this is how the
check should be invoked:

`$ clang++ -cc1 -analyze -analyzer-checker=core.alpha.clone.CloneChecker source.cpp`

## Google Summer of Code 2015

The work described in this poster was done in terms of [Google Summer of Code 2015](https://developers.google.com/open-source/gsoc/) and
supported by Google.

I was working with LLVM Community under mentorship of [Vassil Vassilev](https://github.com/vgvassilev)
([CERN](https://home.cern/)/[FNAL](http://www.fnal.gov/)). Vassil is a known compiler specilist and the creator of
[Cling](https://root.cern.ch/cling), an interactive C++ interpreter used in CERN.

The [GSoC project page](https://www.google-melange.com/archive/gsoc/2015/orgs/llvm/projects/arcadiaq.html) sadly doesn't contain
much information due to my lack of knowledge that a large part of the summary I wrote won't be accessible from there. This poster,
though, containes an extensive overview of the work done during Summer 2015.

## Motivation

Despite Code Clone Detection being quite popular topic over the past years there was no good solution, which was extensible, open,
easy-to-use and would actually do its job really good. While some solutions existed most of them didn't take advantage of modern
compiler technologies and very naive attempts lead to detecting a small part of widely existing code clones.

Numerous research papers proposed text-based approach, which is both not scalable and inefficient. Only few attempts (most notably,
[this one](http://www.semanticdesigns.com/Company/Publications/ICSM98.pdf)) focused on AST analysis, which gives significantly better
results. Taking an advantage of reusing Clang infrastructure, which provides a rich AST for C, C++ and Objective-C, leads to even better
solution.

## Key results

Reusing Clang infrastructure allows to parse the up-to-date C and C++ dialects and detecting very sophisticated code clone instances.

The poster shows that the proposed implementation outperforms existing solutions (those I am aware of) in performance, range of detected
code clones and usability. Many approaches, which are aiming for speed, are not able most Type II and Type III clones (please refer to
Code Clone Taxonomy in Notes). Those trying to detect more types of similarity have serious performance issue. My work combines
performance efficiency while not limiting detection capabilities.

The code used for this paper is availible in [my fork of Clang repository](https://github.com/omtcyfz/clang/tree/CloneDetection).

The following table shows that even a naive implementation of Code Clone check is able to process huge open-source projects and find
many similar pieces of code:

|Project|Normal build time|Build with BasicCloneCheck time|Clones found|
|---|---|---|---|
|OpenSSL|1m26s|9m27s|180|
|Git|0m26s|2m46s|34|
|SDL|0m26s|1m59s|170| 

## Future work

During Summer 2016 I was an intern in Google Munich, where I introduced major improvements to
[clang-rename](http://clang.llvm.org/extra/clang-rename.html) and started clang-refactor (see
[design doc](https://docs.google.com/document/d/1w9IkR0_Gqmd5w4CZ2t_ZDZrNLYVirQPyMS41533HQZE) for reference). Therefore I was unable to
continue my work on coding side and only participated in few discussions. [Raphael Isemann](https://github.com/Teemperor) under
mentorship of Vassil Vassilev and help of Apple's engineers did a great job improving current infrastructure (see
[GSoC project page](https://docs.google.com/document/d/1w9IkR0_Gqmd5w4CZ2t_ZDZrNLYVirQPyMS41533HQZE)) and finally pushing the code to
the Clang repository.

Clang Static Analyzer isn't able to pass information between translation units and this, unfortunately, is a huge limitation for Code
Clone Detection because of its nature: most clones end up in different translation units and are not reported by the check. If a
proper solution is to be made, there is a need to overcome described limitation. My work on clang-refactor might become useful for
an efficient solution.

## Acknowledgement

I would like to thank Vassil Vassilev for guidance and support, LLVM Community for great suggestions and all the work done towards
supporting new contributors and, of course, Google - for creating a great opportunity for students from all over the world and funding.

## Notes {#notes}

[0] For reference on Code Clone Taxonomy see [fairly recent paper](http://www.sciencedirect.com/science/article/pii/S0167642309000367)
by Roy et al.
