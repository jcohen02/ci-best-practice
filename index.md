---
title: Continuous Integration for Research Software
layout: page
---
Table of Contents

* TOC
{:toc}

# Introduction

Continuous Integration (CI) has become widely adopted software development
practice and something we use in the Imperial Research Software Engineering
group as a matter of course. I don't want to spend many column inches defining
Continuous Integration or what you gain by doing it so I'll let the Software
Sustainability Institute [do that for me][SSI]. Suffice it to say that CI is a
vital tool as soon as you want your software to run on more than one computer or
expect more than one person to contribute to it.

[SSI]: https://software.ac.uk/using-continuous-integration-build-and-test-your-software

This document is aimed at developers or maintainers of research software who are
interested in current best practice for CI. It should be of interest whether you
are looking at setting up CI for the first time or reviewing an existing
setup. This document aims to provide a (somewhat opinionated) overview of the
different options for using CI with your code and how to address common
challenges faced by research software.

This is a living document that is intended to be updated to reflect best
practice and the ever changing technology landscape around CI. This document is
mainly written for those developing research software in an academic setting. If
you think anything in this document is out of date (or you just plain disagree)
then please feel free to create an issue in the [GitHub
repo]({{ site.github.repository_url }}) to discuss. If you'd like to write
anything for inclusion please see
[CONTRIBUTING.md]({{ site.github.repository_url }}/CONTRIBUTING.md).

## Use Your Research Software Engineers

Before setting up a CI system consider getting in touch with your [local
Research Software Engineering team](groups.html) for advice. They may be able to
point you in the direction of existing institutional or departmental resources
that you can take advantage of. Plus, they're just good people to talk to
anyway.

## Decisions to Make

To get up and running with CI there are two major choices to make:

1. Which software/service you will use (we'll call this the `front-end`)
1. Where the compute power for your CI jobs will come from (we'll call this the
   `back-end`)

Both of these will be impacted by the individual circumstances and requirements
of your project. We'll start by providing a high-level overview of the different
options available then consider how the differing requirements of a project
might influence or constrain your choices.

# CI Overview

There are **lots** of different options available for doing CI. Fundamentally
however these all consist of the same thing. A `front-end` integrated with your
version control hosting that spawns and coordinates the execution of `jobs`. A
`job` is a computational workload which performs a task such as compiling a new
version of your code or running tests. The jobs are executed by a `back-end`
which is made up of `runners`. These are machines (virtual or otherwise) that
are linked to the `front-end`, pick up `jobs` to run and then stream the results
back. The `front-end` then collects the outcome from all `jobs` and reports an
overall passed status or failed status.

To use a CI system you need to write instructions for the `front-end` which
define the `jobs` that should run when you make changes to your code. This
*generally* takes the form of a yaml file using the approved syntax of your
chosen system and checked into your code repository. You then need a `back-end`
that will pick up those jobs. Depending on the system you choose a `back-end`
may or may not be provided for you. If a `back-end` is provided it will likely
be subject to usage restrictions of one form or another.

To keep things interesting there is no universal terminology used for the
above. Although there is some overlap, individual CI services may use different
terms e.g. `agent` in place of `runner`.

## Online services

Many CI systems are available as commercially hosted services. Typically these
include both a `front-end` and a hosted `back-end`. The business model for these
services generally works by offering their `front-end` for free with tiered
access to their `back-end`. Fortunately competition is driving the free tier of
most services towards increasingly generous offerings for open source
projects. If your project is closed source generally at least some free
resources are offered but consider carefully your current (and future)
requirements.

Broadly, one can distinguish between services included as part of code hosting
(e.g. GitLab CI, GitHub Actions, BitBucket Pipelines) and stand alone services
(e.g. Azure Pipelines, Travis, Circle CI). **Unless you have a compelling reason
it is recommended to use the CI system integrated with your code hosting**. This
is generally easier to setup and will provide the best overall user experience.

If you find that the hosted `back-end` is insufficient it is also generally
supported to add your own infrastructure as a supplement. If you have available
compute resources they can usually be registered as a `runner` that will pick up
`jobs` from your project.

### Anvil

There is one hosted service that is worth a special mention. [Anvil][] is an
STFC-hosted Jenkins instance of particular note as its `back-end` is provided by
SCARF, an HPC cluster. This gives Anvil some unique capabilities such as running
multi-node `jobs`, access to common licensed softwares and other trappings of
numerically intensive simulation environments. This also comes with some
downsides however, in particular the lack of support for containers

[Anvil]: https://anvil.softeng-support.ac.uk/

One notable restriction that applies to Anvil however is that there are
considerable restrictions on eligible projects based on STFC remit.

## Self Hosting

Instead of using a hosted online service it is also possible to host your own CI
solution entirely. Some of the online services also offer the ability to
download and self-host (e.g. Circle CI, GitLab CI), though this sometimes
requires an enterprise license. There are also some systems, typically [FOSS][]
ones, that are more geared towards self hosting (e.g. Jenkins, Buildbot). You
will have to setup your own `back-end` compute resources to process your
workloads.

[FOSS]: https://en.wikipedia.org/wiki/Free_and_open-source_software

If do go the route of hosting a `front-end` Jenkins is a widely used FOSS
option. Consider using the Blue Ocean plugin for enhanced usability and a nicer
interface. It could be worth looking for ways to spread the maintenance
responsibilities, for example by running a single CI instance for multiple
projects across a group or department.

## Recommendations

We suggest the below ordering in terms of preference for a CI setup:

1. Use the `front-end` and `back-end` provided by your version control host
   i.e. GitHub Actions or GitLab CI.
1. Use the `front-end` and `back-end` provided by a third-party online
   service.
1. Use a hosted `front-end` and `back-end` supplemented with additional
   self-hosted `runners`.
1. Use a hosted `front-end` and a fully self-hosted `back-end`.
1. Self-host the `front-end` and `back-end`.

The reasoning behind the above is to use off-the-shelf offerings as far as
possible. If you are looking for a robust and long-term CI system it's easy to
make the case for hosted services. Achieving the same reliability as a
professionally hosted service is difficult and it's easy to underestimate the
amount of time you'll spend maintaining your own installation.

You will need to balance the above ordering against potential costs,
particularly if your project is not open-source. As a rule of thumb any of the
above hosted options should be viable for *most* open-source projects at no
cost. For closed-source, free usage limits of hosted `back-ends` vary
considerably between offerings.

# CI for Research Software

Having described the lay of the land and suggested some ideological preferences
we now consider the particular challenges posed by CI for research software and
how these might influence the choices you make.

## General advice

1. The majority of CI systems offer pretty similar functionality. It may be best
   to think about what will be the least effort to maintain and use rather than
   go for a unique feature that saves half a day of setup but commits you to a
   sub-optimal workflow.
1. Use Docker wherever possible in your `jobs`. Any hosted `back-end` worth its
   salt should support Docker on its `runners`. Using Docker containers gives
   you:
  * Reproducibility - any `job` that fails in the `back-end` can be easily rerun
    locally for interactive debugging.
  * Reliability - `jobs` are less likely to break spontaneously due to
    configuration changes on the `runner`.
  * Flexibility - `jobs` will care a lot less about where they are run meaning
    you're not as tied in to a particular CI system or set of `runners`.

## Challenges for Research Software and CI

The capabilities of available CI systems often reflect the requirements of
commercial or open-source software rather than typical research software. In
this section we discuss some of challenges that can arise when using CI with
research software.

### Computational Intensiveness

Ideally you don't want to be in a situation where building and testing your
project consumes a large amount of cpu time or memory. CI is focused on rapid
iteration so the day-to-day tests you use in development should support this
with short run times and modest data sizes. Most CI systems support scheduling
jobs to run at regular intervals. Consider pulling out your most computationally
intensive tests to run once a week rather than every time you push a commit.

Now lets assume the advice above is impossible or too onerous to implement. If
you're considering a hosted CI `back-end` be careful to check the specs on the
`runners` provided and the usage restrictions that are applied. Whilst total
usage is often not capped for open-source projects there can be limits on the
number of simultaneous `jobs` that can run. Watch out for individual `job` time
limits as well as these can vary considerably between providers. If you can't
find a hosted option that meets your requirements you're stuck in the territory
of providing your own runners which can be configured to run workloads of any
size or length you like.

### Accelerators

We're pretty much talking about GPUs here. This is a tricky one, at present
there are no services we're aware of (paid or free) that include GPUs on their
hosted runners.

Providing your own hardware as an additional `runner` is the most obvious
option. It is also, in principle, possible to have the CI system spin up GPU
resources via a cloud provider (e.g. using something like [GitHub Actions for
Azure][azure_actions]). With cloud pricing this can be done for pennies per
run...  if you can work out a charging model your institution is happy with.

[azure_actions]: https://azure.microsoft.com/en-gb/blog/github-actions-for-azure-is-now-generally-available/

### Specialist Dependencies/Operating Systems

Research software can have complex dependencies (including other pieces of
research software) or rely on restrictively licensed products such as the Intel
compiler suite or Red Hat.

#### My software has complex, but installable dependencies

The most obvious solution in this case is to provide a local `runner` where you
have set up all of the dependencies correctly. This problem can also be worked
around for hosted `back-ends` by use of Docker. You can prepare a Docker image
that contains all of the required dependencies which is then used to run your CI
jobs. Each job will have to download the image from a publicly accessible source
e.g. DockerHub instead of having to build and install dependencies each
time. Watch out for the size of the image though or each job may have to spend a
considerable amount of time downloading it. Some hosted `back-ends` provide
caching for images to avoid having to download it each time.

#### My software depends on a licensed product

Generally the issue with this is that any `runner` executing a job will need to
be able to talk to a license server that is not available outside of your
institution's network. This can be a real challenge for hosted `back-ends`, it
may be worth discussing with your local ICT if a VPN or SSH tunnel solution can
be concocted. Otherwise, with the exception of the below suggestions, your only
option is to self-host one or more `runners` for this purpose.

In a few cases there might be workarounds you can use to avoid self-hosting:

* Anvil provides access to a HPC `backend` that includes common dependencies for
  research software.
* Red Hat have started to provide [docker images][redhat] for their operating
  system. Only a limited number of packages are available to install however
  unless you have access to a copy of the Red Hat repositories.
* Intel has started to release versions of their compilers and numerical
  libraries under their [oneAPI Toolkits][oneapi]. Hope you don't mind an 18GB
  Docker image though.
* Look into non-enterprise equivalents e.g. CentOS instead of Red Hat, GNU
  Octave instead of Matlab

[redhat]: https://catalog.redhat.com/software/containers/search
[oneapi]: https://software.intel.com/content/www/us/en/develop/tools/oneapi.html#oneapi-toolkits

### Multi-node Testing

This primarily applies to MPI capable HPC codes that expect to routinely run
across multiple compute nodes. Similar to using accelerators, we're not aware of
any hosted services that support this.

Using Anvil (if your project is eligible) is one way to avoid self-hosting
`runners` for this. To run on an institutional HPC environment there are various
plugins available for Jenkins that integrate with HPC schedulers (available for
[SGE][], [LSF][] and [PBS][]).

[SGE]: https://github.com/jenkinsci/sge-cloud-plugin
[LSF]: https://github.com/LaisvydasLT/lsf-cloud
[PBS]: https://github.com/biouno/pbs-plugin

### Multiple Operating Systems

Typically this makes a strong case for using a hosted `back-end` and may
strongly determine which `front-end` you go with. The issues with self-hosting
tend to be compounded when dealing with multiple different operating
systems. Macs in particular can be tricky and require dedicated hardware. Many
CI services are now offering hosted Linux, Windows and Mac `runners`
however. Whilst typically only one Linux distribution will be available others
can be readily used via Docker.

# Case Studies

Finally we have a couple of case studies that have gone in different directions
when setting up their CI. We try to examine the different challenges faced by
each project and why they made different choices.

[Case Studies](case-studies.html)
