- I used to be a software engineer, now a PhD student in Meteorology department at UoR
- part of a growing computational research group in the department (show slide)
- we design numerical methods for Computational Fluid Dynamics (CFD) simulations
- we write C++ code using OpenFOAM, an open source C++ CFD library, and we all use ubuntu

- when I started four years ago, our group was smaller (show slide of me and H)
- I ran simulations on the university's locally-installed, locally-managed compute cluster.  It was painful:
  - OpenFOAM was not an officially supported service
  - compile OpenFOAM and its dependencies from source (and new versions of OpenFOAM are released each week!)
  - shared resouce with no limits on individual usage (machines crippled by long-running, long-forgotten processes)
  - unreliable cluster FS
  - unreliable MPI
  - missing or outdated tools: git, gdb, valgrind, strace

- I'll talk today about how I switched to AWS EC2, and more generally about our computational workflow
- three parts: coding, modelling, publishing

## Coding

- code is openly shared on github
- what code (and text) do I have?
  - thesis, journal articles: latex, gnuplot
  - test suite: AtmosTests
  - OpenFOAM extension code: AtmosFOAM, AtmosFOAM-tools, highOrderFit
  - third party mesh generators: gmd-geodesic-mesh, HRgrids, asam\_grid, gmv2openfoam
- pushing to github triggers a Travis CI build that
  - downloads the latest version of OpenFOAM
  - compiles our OpenFOAM extension code, "AtmosFOAM"
  - runs automated tests (not nearly enough of them!)
  - creates a binary Debian package and deploys to a Debian APT repository hosted on AWS S3

## Modelling

- AtmosTests: a suite of idealised atmospheric simulations
- singularity image contains OpenFOAM, AtmosFOAM, mesh generators etc
- AWS EC2 instances are ubuntu+singularity, nothing more
- AWS EC2 instances manually created, manually started
- one or more simulations are run with ninja:
  `singularity exec -e atmostests.img ninja build/mountainAdvect-cutCell-linearUpwind/s3.uploaded`
- the results are uploaded to AWS S3
- some simulations last days: AWS configured to automatically shutdown idle instances

## Publishing

- simulation results are downloaded from S3, rendered as figures, included into LaTeX documents
- again, all this is automated with ninja-build and wrapped into singularity images


# What's left?

- AWS cluster, MPI across EC2 instances (+ singularity!)
- EC2 instance management is not automated -- not sure what's best here... and is more complexity worth it?
- a lot of bespoke 'glue' code that's not especially reusable: debian package configuration, travis-ci configuation, Singularity recipes, ninja build scripts 
- took some software engineering experience and a lot of time to set up... is this a job for RSEs?  can it all be made simpler?
- will require some ongoing maintenance after I leave... who will [be able to] do it?
