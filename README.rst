======================
 Cram: It's test time
======================

Cram is a functional testing framework for command line applications.
Cram tests look like snippets of interactive shell sessions. Cram runs
each command and compares the command output in the test with the
command's actual output.

Here's a snippet from `Cram's own test suite`_::

    The $PYTHON environment variable should be set when running this test
    from Python.

      $ [ -n "$PYTHON" ] || PYTHON="`which python`"
      $ [ -n "$PYTHONPATH" ] || PYTHONPATH="$TESTDIR/.." && export PYTHONPATH
      $ if [ -n "$COVERAGE" ]; then
      >   coverage erase
      >   alias cram="`which coverage` run --branch -a $TESTDIR/../scripts/cram"
      > else
      >   alias cram="$PYTHON $TESTDIR/../scripts/cram"
      > fi
      $ command -v md5 > /dev/null || alias md5=md5sum

    Usage:

      $ cram -h
      [Uu]sage: cram \[OPTIONS\] TESTS\.\.\. (re)

      [Oo]ptions: (re)
        -h, --help          show this help message and exit
        -V, --version       show version information and exit
        -q, --quiet         don't print diffs
        -v, --verbose       show filenames and test status
        -i, --interactive   interactively merge changed test output
        -d, --debug         write script output directly to the terminal
        -y, --yes           answer yes to all questions
        -n, --no            answer no to all questions
        -E, --preserve-env  don't reset common environment variables
        --keep-tmpdir       keep temporary directories
        --shell=PATH        shell to use for running tests (default: /bin/sh)
        --shell-opts=OPTS   arguments to invoke shell with
        --indent=NUM        number of spaces to use for indentation (default: 2)
        --xunit-file=PATH   path to write xUnit XML output
        --no-escape         don't escape test output
        --debug-script      write script directly to the terminal

The format in a nutshell:

* Cram tests use the ``.t`` file extension.

* Lines beginning with two spaces, a dollar sign, and a space are run
  in the shell.

* Lines beginning with two spaces, a greater than sign, and a space
  allow multi-line commands.

* All other lines beginning with two spaces are considered command
  output.

* Output lines ending with a space and the keyword ``(re)`` are
  matched as `Perl-compatible regular expressions`_.

* Lines ending with a space and the keyword ``(glob)`` are matched
  with a glob-like syntax. The only special characters supported are
  ``*`` and ``?``. Both characters can be escaped using ``\``, and the
  backslash can be escaped itself.

* Output lines ending with either of the above keywords are always
  first matched literally with actual command output.

* Lines ending with a space and the keyword ``(no-eol)`` will match
  actual output that doesn't end in a newline.

* Actual output lines containing unprintable characters are escaped
  and suffixed with a space and the keyword ``(esc)``. Lines matching
  unprintable output must also contain the keyword.

* Anything else is a comment.

.. _Cram's own test suite: https://bitbucket.org/brodie/cram/src/0.6/tests/cram.t
.. _Perl-compatible regular expressions: https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions


Download
--------

* `cram-0.7.tar.gz`_ (32 KB, requires Python 2.4-2.7 or Python 3.1 or newer)

.. _cram-0.7.tar.gz: https://bitheap.org/cram/cram-0.7.tar.gz


Installation
------------

Install Cram using make::

    $ wget https://bitheap.org/cram/cram-0.7.tar.gz
    $ tar zxvf cram-0.7.tar.gz
    $ cd cram-0.7
    $ make install


Usage
-----

Cram will print a dot for each passing test. If a test fails, a
`unified context diff`_ is printed showing the test's expected output
and the actual output. Skipped tests (empty tests and tests that exit
with return code ``80``) are marked with ``s`` instead of a dot.

For example, if we run Cram on `its own example tests`_::

    .s.!
    --- examples/fail.t
    +++ examples/fail.t.err
    @@ -3,21 +3,22 @@
       $ echo 1
       1
       $ echo 1
    -  2
    +  1
       $ echo 1
       1

     Invalid regex:

       $ echo 1
    -  +++ (re)
    +  1

     Offset regular expression:

       $ printf 'foo\nbar\nbaz\n\n1\nA\n@\n'
       foo
    +  bar
       baz

       \d (re)
       [A-Z] (re)
    -  #
    +  @
    s.
    # Ran 6 tests, 2 skipped, 1 failed.

Cram will also write the test with its actual output to
``examples/fail.t.err``, allowing you to use other diff tools. This
file is automatically removed the next time the test passes.

When you're first writing a test, you might just write the commands
and run the test to see what happens. If you run Cram with ``-i`` or
``--interactive``, you'll be prompted to merge the actual output back
into the test. This makes it easy to quickly prototype new tests.

You can specify a default set of options by creating a ``.cramrc``
file. For example::

    [cram]
    verbose = True
    indent = 4

Is the same as invoking Cram with ``--verbose`` and ``--indent=4``.

To change what configuration file Cram loads, you can set the
``CRAMRC`` environment variable. You can also specify command line
options in the ``CRAM`` environment variable.

Note that the following environment variables are reset before tests
are run:

* ``TMPDIR``, ``TEMP``, and ``TMP`` are set to the test runner's
  ``tmp`` directory.

* ``LANG``, ``LC_ALL``, and ``LANGUAGE`` are set to ``C``.

* ``TZ`` is set to ``GMT``.

* ``COLUMNS`` is set to ``80``. (Note: When using ``--shell=zsh``,
  this cannot be reset. It will reflect the actual terminal's width.)

* ``CDPATH`` and ``GREP_OPTIONS`` are set to an empty string.

Cram also provides the following environment variables to tests:

* ``CRAMTMP``, set to the test runner's temporary directory.

* ``TESTDIR``, set to the directory containing the test file.

* ``TESTFILE``, set to the basename of the current test file.

* ``TESTSHELL``, set to the value specified by ``--shell``.

Also note that care should be taken with commands that close the test
shell's ``stdin``. For example, if you're trying to invoke ``ssh`` in
a test, try adding the ``-n`` option to prevent it from closing
``stdin``. Similarly, if you invoke a daemon process that inherits
``stdout`` and fails to close it, it may cause Cram to hang while
waiting for the test shell's ``stdout`` to be fully closed.

.. _unified context diff: https://en.wikipedia.org/wiki/Diff#Unified_format
.. _its own example tests: https://bitbucket.org/brodie/cram/src/default/examples/


Development
-----------

Download the official development repository using Mercurial_::

    hg clone https://bitbucket.org/brodie/cram

Or Git_::

    git clone https://github.com/brodie/cram.git

Test Cram using Cram::

    pip install -r requirements.txt
    make test

Visit Bitbucket_ or GitHub_ if you'd like to fork the project, watch
for new changes, or report issues.

.. _Mercurial: http://mercurial.selenic.com/
.. _Git: http://git-scm.com/
.. _coverage.py: http://nedbatchelder.com/code/coverage/
.. _Bitbucket: https://bitbucket.org/brodie/cram
.. _GitHub: https://github.com/brodie/cram
