# sla - Simple Little Automator 🥗

Tired of Make's moronic syntax and rules? Not working on a project that
requires incremental compilation? 😭 when you need to install 2.4
GiB of Ruby dependencies for a simple build system? Want to be able to execute
simple build rules from anywhere in your project tree? Then `sla` is for you!

sla is the Simple Little Automator. It's a tiny shell script that invokes
shell functions found in a `build.sla` script in your project's root dir. It
is ideal for smaller projects and projects that don't require compilation,
such as scripting languages.

![](https://raw.githubusercontent.com/fboender/sla/master/screenshot.png)

## Usage

### Example usage

Example usage:

    ~/Projects/my_project/src/llt/ $ sla test
    ./src/tools.py:25:80: E501 line too long (111 > 79 characters)
    Exection of rule 'test' failed with exitcode 2. Duration: 0h 0m 1s

`sla` searches up the current path until it finds a `build.sla` file, changes
to that directory, sources it in the shell, and executes the requested shell
function. Here's what the `build.sla` file for the above 'test' rule might
look like:

    ~/Projects/my_project/src/llt/ $ cat ../../build.sla
    #
    # This is a script containing functions that are used as build rules. You can
    # use the Simple Little Automator (https://github.com/fboender/sla) to run
    # these rules, or you can run them directly in your shell:
    #
    #   $ bash -c ". build.sla && test"
    #

    clean () {
        # Clean artifacts and trash from repo
        find ./ -name "*.pyc" -delete
        find ./ -name "*.tmp" -delete
    }

    test () {
        # Run some code tests
        clean  # Depend on 'clean' rule
        flake8 --exclude src/llt --ignore=E501 src/*.py
    }

As you can see, build rules are just plain old shell functions. Creating
dependencies on other rules is as simple as calling it as a normal shell
function.

You can list available rules by simply omitting a rule name:

    $ sla
    Available rules:
      - doc: Generate documentation
      - clean: Clean artifacts and trash from repo
      - test: Run some code tests
      - install: Install the program

Functions that start with an underscore ('`_`') are not shown.

If a comment is present right after the function definition, it will be used
as a description for the rule. Only one line is supported.

`sla` will stop running as soon as a command returns a non-zero exit code.

### Running rules without `sla`

Since `sla` rules are just plain old shell functions, you don't even need
`sla` installed to run them!:

    ~/Projects/my_project $ bash -c ". build.sla && test"
    ./src/tools.py:25:80: E501 line too long (111 > 79 characters)

That's useful so that people don't need to install the build system you prefer
just to do a `make install`.

### Full usage

Here's the output of `--help`, although there's not much to see:

    Usage: sla [-hdv] [--version] <rule> [param [param...]]
    Simple Little Automator. Find and execute tasks from a shell script.

    Arguments:
      -h, --help    Show this help message
      -d, --debug   Show debug information
      -v, --verbose Verbose output
      --version     Show sla version

    If no <rule> is specified, lists the available rules

You can use the `--verbose` option to show what your rules are executing.
Basically the same as doing `set -x` in your script.

### Passing values

You can pass values as environment variables:

    $ export TARGET=/usr/local
    $ sla install  # installs in /usr/local

or

    TARGET=/usr/local sla install

You can also pass command-line arguments:

    sla install /usr/local

These become available as normal positional arguments in a rule / shell
function:

    install () {
        TARGET=$1
    }

### Tips and tricks

#### Automatic execution of rules

You can use a tool like [entr](http://entrproject.org/) to automatically
execute rules when files change:

    $ find ./ -name "*.1.md" | entr sla doc
    Converting sec-diff.1.md
    Execution of rule 'doc' succeeded with exitcode 0. Duration: 0h 0m 0s

The `doc` rule converts a Markdown file to a manual page. `entr` watches all
changes to `*.1.md` files in the project, and if any of them change, it
executes `sla doc` to regenerate all man pages.

#### 'install' rule

If you have a rule called "install", and you also want to use the `install`
unix tool to install software, you'll end up with an endless loop, unless you
specify that you want to use the system's `install` and not the shell function
you just wrote. You can use `env` for that:

    install () {
        env install -m 755 ./sla "$DEST"
    }


## Installation

You can install `sla` with itself from the repo.

System-wide:

    git clone https://github.com/fboender/sla.git
    cd sla
    sudo ./sla install

Just for your user:

    git clone https://github.com/fboender/sla.git
    cd sla
    PREFIX=~/.local ./sla install
    
For the above to work, you should have `~/.local/bin` in your PATH.

## License

sla is released under the GPLv3 license:

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.

    For the full license, see the LICENSE file.
