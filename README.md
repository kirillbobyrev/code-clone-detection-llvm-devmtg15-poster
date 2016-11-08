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

`$ clang++ -cc1 -analyze -analyzer-checker=alpha.clone.CloneChecker source.cpp`

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

## Similar pieces of code found using proposed technique

Compared to C++ projects, projects written in C suffer from code duplication issues significantly more.

The following pieces of code can be easily wrapped into functions to prevent potential errors, such as fixing a bug in one of the
clone intstances and ignoring the others.

### OpenSSL

A little more throughout analysis was able to identify around 500 big enough (see following examples) code clones.

Project commit used for analysis: b77b6127e8de38726f37697bbbc736ced7b49771.

#### crypto/bf/bf_cfb64.c

```c
    if (encrypt) {
        while (l--) {
            if (n == 0) {
                n2l(iv, v0);
                ti[0] = v0;
                n2l(iv, v1);
                ti[1] = v1;
                BF_encrypt((BF_LONG *)ti, schedule);
                iv = (unsigned char *)ivec;
                t = ti[0];
                l2n(t, iv);
                t = ti[1];
                l2n(t, iv);
                iv = (unsigned char *)ivec;
            }
            c = *(in++) ^ iv[n];
            *(out++) = c;
            iv[n] = c;
            n = (n + 1) & 0x07;
        }
    } else {
        while (l--) {
            if (n == 0) {
                n2l(iv, v0);
                ti[0] = v0;
                n2l(iv, v1);
                ti[1] = v1;
                BF_encrypt((BF_LONG *)ti, schedule);
                iv = (unsigned char *)ivec;
                t = ti[0];
                l2n(t, iv);
                t = ti[1];
                l2n(t, iv);
                iv = (unsigned char *)ivec;
            }
            cc = *(in++);
            c = iv[n];
            iv[n] = cc;
            *(out++) = c ^ cc;
            n = (n + 1) & 0x07;
        }
    }
```

#### crypto/ec/ecp_smpl.c

```c
    if (!b->Z_is_one) {
        if (!field_sqr(group, Zb23, b->Z, ctx))
            goto end;
        if (!field_mul(group, tmp1, a->X, Zb23, ctx))
            goto end;
        tmp1_ = tmp1;
    } else
        tmp1_ = a->X;
    if (!a->Z_is_one) {
        if (!field_sqr(group, Za23, a->Z, ctx))
            goto end;
        if (!field_mul(group, tmp2, b->X, Za23, ctx))
            goto end;
        tmp2_ = tmp2;
    } else
        tmp2_ = b->X;

    /* compare  X_a*Z_b^2  with  X_b*Z_a^2 */
    if (BN_cmp(tmp1_, tmp2_) != 0) {
        ret = 1;                /* points differ */
        goto end;
    }

    if (!b->Z_is_one) {
        if (!field_mul(group, Zb23, Zb23, b->Z, ctx))
            goto end;
        if (!field_mul(group, tmp1, a->Y, Zb23, ctx))
            goto end;
        /* tmp1_ = tmp1 */
    } else
        tmp1_ = a->Y;
    if (!a->Z_is_one) {
        if (!field_mul(group, Za23, Za23, a->Z, ctx))
            goto end;
        if (!field_mul(group, tmp2, b->Y, Za23, ctx))
            goto end;
        /* tmp2_ = tmp2 */
    } else
        tmp2_ = b->Y;
```

#### crypto/rsa/rsa_x931g.c
```c
    if (Xp && rsa->p == NULL) {
        rsa->p = BN_new();
        if (rsa->p == NULL)
            goto err;

        if (!BN_X931_derive_prime_ex(rsa->p, p1, p2,
                                     Xp, Xp1, Xp2, e, ctx, cb))
            goto err;
    }

    if (Xq && rsa->q == NULL) {
        rsa->q = BN_new();
        if (rsa->q == NULL)
            goto err;
        if (!BN_X931_derive_prime_ex(rsa->q, q1, q2,
                                     Xq, Xq1, Xq2, e, ctx, cb))
            goto err;
    }
```

### Vim

Patch 8.0.0071.

Around 300 similar code pieces in total found in Vim.

#### src/farsi.c

```c
    // Chunk 1.
    switch (gchar_cursor())
    {
	case ALEF:
		tempc = ALEF_;
		break;
	case ALEF_U_H:
		tempc = ALEF_U_H_;
		break;
	case _AYN:
		tempc = _AYN_;
		break;
	case AYN:
		tempc = AYN_;
		break;
	case _GHAYN:
		tempc = _GHAYN_;
		break;
	case GHAYN:
		tempc = GHAYN_;
		break;
	case _HE:
		tempc = _HE_;
		break;
	case YE:
		tempc = YE_;
		break;
	case IE:
		tempc = IE_;
		break;
	case TEE:
		tempc = TEE_;
		break;
	case YEE:
		tempc = YEE_;
		break;
	default:
		tempc = 0;
    }

...
    // Chunk 2.
    switch (gchar_cursor())
    {
	case ALEF:
		tempc = ALEF_;
		break;
	case ALEF_U_H:
		tempc = ALEF_U_H_;
		break;
	case _AYN:
		tempc = _AYN_;
		break;
	case AYN:
		tempc = AYN_;
		break;
	case _GHAYN:
		tempc = _GHAYN_;
		break;
	case GHAYN:
		tempc = GHAYN_;
		break;
	case _HE:
		tempc = _HE_;
		break;
	case YE:
		tempc = YE_;
		break;
	case IE:
		tempc = IE_;
		break;
	case TEE:
		tempc = TEE_;
		break;
	case YEE:
		tempc = YEE_;
		break;
	default:
		tempc = 0;
    }
```

#### evalfunc.c

```c
    // Code chunk 1.
    /* Optional arguments: line number to stop searching and timeout. */
    if (argvars[1].v_type != VAR_UNKNOWN && argvars[2].v_type != VAR_UNKNOWN)
    {
	lnum_stop = (long)get_tv_number_chk(&argvars[2], NULL);
	if (lnum_stop < 0)
	    goto theend;
#ifdef FEAT_RELTIME
	if (argvars[3].v_type != VAR_UNKNOWN)
	{
	    time_limit = (long)get_tv_number_chk(&argvars[3], NULL);
	    if (time_limit < 0)
		goto theend;
	}
#endif
    }

...
  // Code chunk 2.
	if (argvars[5].v_type != VAR_UNKNOWN)
	{
	    lnum_stop = (long)get_tv_number_chk(&argvars[5], NULL);
	    if (lnum_stop < 0)
		goto theend;
#ifdef FEAT_RELTIME
	    if (argvars[6].v_type != VAR_UNKNOWN)
	    {
		time_limit = (long)get_tv_number_chk(&argvars[6], NULL);
		if (time_limit < 0)
		    goto theend;
	    }
#endif
	}
```

### Git

Commit be5a750939c212bc0781ffa04fabcfd2b2bd744e.

#### remote-curl.c

```c
       } else if (!strcmp(name, "cloning")) {
		if (!strcmp(value, "true"))
			options.cloning = 1;
		else if (!strcmp(value, "false"))
			options.cloning = 0;
		else
			return -1;
		return 0;
	} else if (!strcmp(name, "update-shallow")) {
		if (!strcmp(value, "true"))
			options.update_shallow = 1;
		else if (!strcmp(value, "false"))
			options.update_shallow = 0;
		else
			return -1;
		return 0;
```

#### xdiff/xprepare.c

This one looks especially weird.

```c
	if ((mlim = xdl_bogosqrt(xdf1->nrec)) > XDL_MAX_EQLIMIT)
		mlim = XDL_MAX_EQLIMIT;
	for (i = xdf1->dstart, recs = &xdf1->recs[xdf1->dstart]; i <= xdf1->dend; i++, recs++) {
		rcrec = cf->rcrecs[(*recs)->ha];
		nm = rcrec ? rcrec->len2 : 0;
		dis1[i] = (nm == 0) ? 0: (nm >= mlim) ? 2: 1;
	}

	if ((mlim = xdl_bogosqrt(xdf2->nrec)) > XDL_MAX_EQLIMIT)
		mlim = XDL_MAX_EQLIMIT;
	for (i = xdf2->dstart, recs = &xdf2->recs[xdf2->dstart]; i <= xdf2->dend; i++, recs++) {
		rcrec = cf->rcrecs[(*recs)->ha];
		nm = rcrec ? rcrec->len1 : 0;
		dis2[i] = (nm == 0) ? 0: (nm >= mlim) ? 2: 1;
	}
```

## Notes

[0] For reference on Code Clone Taxonomy see [fairly recent paper](http://www.sciencedirect.com/science/article/pii/S0167642309000367)
by Roy et al.
