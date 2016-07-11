SCLs and Python extensions
==========================

This repo provides demonstration instructions and scripts for some
problems that can arise when using Python runtimes provided for
CentOS and RHEL as Software Collections to install command line
scripts.

Running the demo on Fedora
--------------------------

Since SCLs aren't currently provided for Fedora, the scenarios can be
tested under docker:

    $ sudo docker run -it --name scl-pipsi-demo centos:7

And then inside the container:

    # yum install -y centos-release-scl
    [... output snipped ...]
    # yum install -y yum-utils
    [... output snipped ...]
    # yum-config-manager --enable centos-sclo-rh-testing
    [... output snipped ...]
    # yum install -y rh-python35
    [... output snipped ...]

To start the same container again later:

    $ sudo docker start -ai scl-pipsi-demo

Demonstrating the problem
-------------------------

We can then illustrate the problem by installing a local user script
that we expect to run using the SCL-provided Python runtime.

First, as root, we ensure pipsi is available in the SCL environment:

    # scl enable rh-python35 bash
    # pip install pipsi
    [... output snipped ...]

We then terminate that subshell and create and switch to a demo user:

    # adduser demo
    # su demo

As the demo user, we then use pipsi in the SCL to install a script to our
home directory, and adjust the PATH in the subshell to let us find and run it:

    $ export LANG=en_US.utf8
    $ scl enable rh-python35 bash
    $ which python
    $ pipsi install Pygments
    [... output snipped ...]
    $ which pygmentize
    $ export PATH=$PATH:~/.local/bin
    $ which pygmentize
    ~/.local/bin/pygmentize
    $ pygmentize -l python `which pygmentize`
    [... output snipped ...]

We can then terminate that subshell, check the SCL is disabled, and show our
installed utility no longer works:

    $ which python
    /usr/bin/python
    $ export PATH=$PATH:~/.local/bin
    $ which pygmentize
    ~/.local/bin/pygmentize
    $ pygmentize -l python `which pygmentize`
    /home/demo/.local/venvs/pygments/bin/python3: error while loading shared libraries: libpython3.5m.so.rh-python35-1.0: cannot open shared object file: No such file or directory

The fact it relies on an SCL runtime rather than the system Python installation
*should* be a hidden implementation detail of the `pipsi` installed script, but
the need for an explicit `scl enable` call to correctly find shared libraries
means that isn't the case.
