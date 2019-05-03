<img align="right"  src="https://i.imgur.com/tpVSrBr.png" width="45%" />  

# PFT

This is a unit test library and tester for ft\_printf.  

By default, it can check if your *completed* printf is pretty good or not pretty good.   

It's **more** useful as a production tool while you're developing ft\_prinf, because it lets you enable and disable entire blocks of tests at once, search and run tests by name and category, and in general perform quick regression testing. It's quick and easy to add your own tests, which I recommend on principle. It's built to be flexible, so you can use it how you wish.   

<p align="center">
  <img src="https://i.imgur.com/Iwsvc2Y.png" width="50%" />
</p>

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
  - [Test Naming Conventions (for looking up tests)](#test-naming-conventions)
  - [Adding Tests](#adding-tests)
  - [Using PFT with LLDB or other debuggers](#using-pft-with-lldb-or-other-debuggers) 
  - [Enabling and Disabling Tests](#enabling-and-disabling-tests)
- [How It Works](#how-it-works-in-brief)
- [Advanced Options](#advanced-options)
  - [Leaks Test (BETA)](#leaks-test-beta)
  - [Filtering by recent test history](#additional-options-when-test-history-logging-is-enabled)
- [Troubleshooting](#troubleshooting)

## Requirements

You have to have a Makefile in your project directory that will compile libftprintf.a as the default make option, and your libftprintf.a has to have ft\_printf inside.

Other than this, it should be completely general to all ft\_printf projects.  

# Installation

In the root of your repo, run this command:

***DEV BRANCH INSTALLATION***

```
git clone --branch dev https://github.com/gavinfielder/pft.git testing && echo "testing/" >> .gitignore
```

***If your libft.a is separate from libftprintf.a:***   
If you include all required .o files (including your libft) in libftprintf.a, this is not necessary. If you do NOT, and require your libft separate, you must set `USE_SEPARATE_LIBFT=1` in PFT's Makefile. See the PFT Makefile, and it should be self-explanatory.  

# Usage
 - `./test help` shows some help text and usage examples  
 - `./test help all` shows more help text  

The executable accepts the following as queries:
 - `./test moul` runs all the enabled tests whose name starts with a string, in this case 'moul'
 - `./test "d_*prec"` is a wildcard search; this one runs all the enabled tests that have start with 'd\_' and have 'prec' in the name.
 - `./test 42 84` runs (enabled) test #42 through test #84
 - `./test 42` runs test #42 (also turns on debugger compatibility mode.)
 - `./test` runs all the enabled tests
 - `./test -d [any of the above queries]` runs the selected test(s) in debugger compatibility mode.

Wildcard-based searches have an implict '\*' at the end. For example, `./test "*zeropad"` runs all the tests that have 'zeropad' anywhere in the name.

## Test Naming Conventions
These are the naming conventions currently used in the included unit\_tests.c.   
 - d, i, o, u, x, X, c, s, p, f tests start with `d_`, `i_`, `o_`, etc.
 - `%%` tests start with `pct_`
 - hh, h, l, ll tests usually have '`size`' in the name
 - L (long double) tests start with `f_L_`
 - `#` tests usually have '`af`' in the name (or '`altform`')
 - 0 (zero padding) tests usually have '`zp`' in the name
 - `-` (left justify) tests usually have '`lj`' in the name
 - `+` tests usually have '`as`' or '`allsign`' in the name
 - ' ' (space padding) tests usually have '`sp`' in the name
 - Precision tests usually have '`prec`' in the name
 - Field width tests usually have '`width`' or just '`w`' in the name
 - Simple tests usually have '`basic`' in the name
 - Tests taken from moulinette files start with `moul_`
 - The `moul_` block has subgroups `moul_d_`, `moul_i_`, `moul_o_`, etc.
 - Tests adapated from 42FileChecker have '`ftfc`' in the name   

These naming conventions are ***usually*** followed. There's a lot of tests, so making it more consistent is a huge task for another day. The first rule plus the `moul_` block are the two that most users care about, and they're both consistent.  

## Disclaimer about the `moul_` tests

They are outdated. At the very least, they were taken from before the subject recently changed. ***IN ADDITION*** more tests were added to moulinette to more sections than just `%f`. Do not think that by simply passing the `moul_` block, you will pass moulinette. Multiple people have already tried this and failed.  

## Workflow with PFT

unit\_tests.c shows you all the tests that are available. Failing a test means that your output and/or return value was not the same as the libc printf. When this happens, there will be a new file, 'results.txt', that holds information about the failed test, the first line of code for the test (most of them are one line anyway), what printf printed, and what ft\_printf printed.  

### Adding Tests   
<p align="center">
  <img src="https://i.imgur.com/CGBRUTf.png" width="70%" />
</p>

You can add your own tests to unit\_tests.c, following the same format as the existing tests. You do not need to do anything except write the function in this file and re-make. The new tests will be included in the test index and can be queried the same way.    

Valid unit tests have the prototype `int  foo(void)` and return the value returned by a call to the `test` function, which has the same prototype as printf.

### Using PFT with LLDB or other debuggers
PFT compiles with debugging symbols by default, and also by default, running a single test e.g. `./test 42` turns on [debugger compatibility mode](#debugger-compatibility-mode). You can force all tests to run in debugger compatibility mode by using the `-d` command line option e.g. `./test -d nospec`. You can read more about command line options and debugger compatibility mode under [Advanced Options](#advanced-options).   

tl;dr: To use PFT with lldb, use it like this: `lldb ./test 42`.  

Of course, in order for lldb to read your own libftprintf.a, you must also use the `-g` flag in your own Makefile...

## Enabling and Disabling tests

I have provided scripts that make it easy to enable and disable tests by a search pattern. Example:

```bash
Simple prefix-based search:
 ./disable-test s                         # All the tests that start with 's' are disabled
 ./enable-test s_null_                    # All the tests that start with 's_null_' are enabled
 ./disable-test && ./enable-test s        # Disables all tests except tests that start with 's'

Wildcard search:
 ./disable-test "*zeropad"      # Disables all the tests that have 'zeropad' anywhere in the name
 ./enable-test "*null*prec"     # Enables all the tests that have a 'null' followed by a 'prec'
 ./enable-test "s_*prec"        # Enables all tests that start with 's_' and have a 'prec' in the name
```

You **can** call `./enable-test` (with no arguments) to enable all tests, but keep in mind that some tests are disabled by default because if you have not implemented certain bonuses, your ft\_printf will segfault.  

## Disabling Return Value Check

The PFT Makefile includes an option to ignore return value checking. I included this because at the time of writing this, moulinette does not check return value, and from what I've seen, the return value is the #1 reason people fail a lot of PFT tests. I don't encourage ignoring the return value, but it is an option if you would like to.  

# How it works, in Brief

The Makefile creates two versions of each unit test function, one that uses ft\_printf, and one that uses printf. For each test, it redirects stdout to a file, then compares their return value. If the return value is identical, it opens both files and reads each one byte by byte until *both* reach EOF. If any single byte differs, the test fails.

# Advanced Options
- [Run Options (also command line options)](#run-options)
  - [Test History Logging](#test-history-logging)
  - [Debugger Compatibility Mode](#debugger-compatibility-mode)
  - [Timeout](#timeout)
  - [Leaks Test (BETA)](#leaks-test-beta)
  - [Fork Mode](#fork-mode)
  - [Signal Handling](#handle-signals)
- [Options in the PFT Makefile](#options-in-the-pft-makefile)
  - [Default run options](#options-in-the-pft-makefile)
  - [Test outdated time](#options-in-the-pft-makefile)
- [Other Options](#other-options)

## Run Options
These options are selected either in the PFT Makefile (to set the default run options) and/or as command line options. 

In both cases, options are processed left to right, and can override previous selections.  

| Option                                                       |  On  |  Off  | Note                                                                                          |
|--------------------------------------------------------------|------|-------|-----------------------------------------------------------------------------------------------|
| [Debugger compatibility Mode](#debugger-compatibility-mode)  | `-d` |       | default on for single tests, off otherwise                                                    |
| [Timeout](#timeout)                                          | `-t` | `-T`  | default on                                                                                    |
| [Test history logging](#test-history-logging)                | `-l` | `-L`  | default on; enables [extra options](#additional-options-when-test-history-logging-is-enabled) |
| Include disabled tests                                       | `-a` | `-A`  | default off                                                                                   |
| [Leaks test (BETA)](#leaks-test-beta)                        | `-k` | `-K`  | default off; turns off fork mode                                                              |
| [Fork mode](#fork-mode)                                      | `-x` | `-X`  | default on                                                                                    |
| [Handle Signals](#handle-signals)                            | `-s` | `-S`  | default on (non-fork mode only)                                                               |
| Print run info before tests                                  | `-i` | `-I`  | default on                                                                                    |

### Additional options when test history logging is enabled

|                                     |     | |  Historical category       |       |
|-------------------------------------|-----|-|----------------------------|-------|
| Select only the following           | `=` | |  Recently failed tests     |  `f`  |
| Include the following               | `+` | |  Recently passed tests     |  `p`  |
| Exclude the following               | `-` | |  Outdated tests            |  `o`  |
|                                     |     | |  Tests with no history     |  `n`  |

-  `-W` : do not write new test history.

**examples**
 - `./test --p x` runs tests that start with 'x', but excludes tests that recently passed.
 - `./test -d=po nocrash` runs tests in debug mode that start with 'nocrash' that are either passing or outdated.
 - `./test -=f s` runs only recently failed tests that start with 's'.

### Test History Logging
By default, PFT will track the last time each test passes and fails, and print information about the most recent run of each test. Tests' history will also become 'outdated' if their age exceeds the value set in the PFT Makefile. This feature also allows selecting tests to run by whether they have recently passed or failed. See [Additional options when test history logging is enabled](#additional-options-when-test-history-logging-is-enabled).

By default, the PFT Makefile removes the test history whenever unit\_tests.c is strictly newer. This prevents the test history from becoming corrupted when you add your own tests (potentially changing the test numbers). By default the enable-test and disable-test scripts, as they modify unit\_tests.c, will trigger such removal of history.csv. There is an option in the makefile to disable the history removal behavior, as well as an option in each of these scripts to touch history.csv to prevent the removal trigger. Use these options at your own discretion.  

### Debugger Compatibility Mode
Debuggers tend to only work well on single-threaded single processes, so "debugger compatibility mode" means "disable forking and multithreading". It also disables signal handling. `-d` is identical to `-XTS`. By default, debugger compatibility mode is turned on when running a single test e.g. `./test 42`.
### Timeout
Fails tests after a specified time interval. The timeout duration can be set in the PFT Makefile. Only available in fork mode.

### Leaks Test (BETA)
Specify leaks test with `-k`. When this option is on, a leaks test command will run after all tests are completed. This disables fork mode. Leaks test will not run when any test segfaulted or otherwise terminated abnormally (as in this case leaks can come from PFT).

This feature is still being tested. Do not rely on it completely.
### Fork Mode
By default, PFT calls ft\_printf only on forked child processes to improve stability. Turning this option off means that tests will run in a single process and a single thread. Timeout is not available when fork mode is off.
### Handle Signals
(only applies to non-fork mode) When this option is off, signals will not be caught by PFT and PFT will stop immediately when a test aborts abnormally. When this option is on, abnormal terminations by certain signals will be caught by PFT and tests will continue running.    

## Options in the PFT Makefile
| Option                                  | Description                                                                                                                                                                                                                                                                                |
|-----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DEFAULT_RUN_OPTIONS`                   | (string) Determines default program behavior. These are the options that are selected by default at the start of the program; they are overridden by any command line options given at execution time.                                                                                     |
| `TIMEOUT_SECONDS`                       | (float) time in seconds for a test to time out, when timeouts are enabled                                                                                                                                                                                                                  |
| `TEST_OUTDATED_TIME`                    | (long) time in seconds for a test result history to become outdated. The default is 900 (15 minutes). You can set this value to 0 to always consider tests outdated, or to 9999999 to never consider test results outdated, or anything in between.                                        |
| `MAKE_RE_ALSO_REBUILDS_LIBFTPRINTF`     | (1 or 0) if 1, `make re` will also recompile your project                                                                                                                                                                                                                                  |
| `IGNORE_RETURN_VALUE`                   | (1 or 0) When this option is 0, tests fail if ft\_printf and printf have different return values. If 1, the return value is ignored.                                                                                                                                                       | 
| `REMOVE_HISTORY_WHEN_TESTS_NEW`         | (1 or 0) If this option is 1, the makefile will remove history.csv whenever unit\_tests.c is strictly newer.                                                                                                                                                                               |
| `LEAKS_TEST_CMD`                        | (C macro) The command to run at the end of the test run when the leaks test (`-k`) is selected. The default is to invoke `leaks(1)` with `system(3)`.                                                                                                                                      |
| `TEST_FAIL_LOGGING_MAXBYTES`                        | (int) The maximum size of output strings to show in results.txt                                                                                                                                                                                                                |
| `SINGLE_NUMBER_SINGLE_TEST`             | (1 or 0) When this option is 1, single numeric arguments run a single test. If 0, single numeric arguments runs all tests starting at the given number. The latter is a legacy feature, but could still be useful if you are using only your own tests and are writing them as you develop |
| `SINGLE_TEST_TURNS_ON_LLDB_COMPAT_MODE` | (1 or 0) When this option is 1 (and `SINGLE_NUMBER_SINGLE_TEST` is also 1), single numeric arguments run the requested test in [debugger compatibility mode](#debugger-compatibility-mode) unless explicitly overridden with command line options.                                         |
| `LIBFTPRINTF_DIR` | (path)  If you do not wish to have PFT inside your local repo, you can specify a different path for your project.                                                                                                                                                                                                |

## Other Options
The enable-test and disable-test scripts both have the option `TOUCH_TEST_HISTORY`. When this option is 1, running the script will also touch history.csv, which prevents a makefile removal of the test history when these scripts are used.

# Troubleshooting

**I tried making and got a bunch of "undefined symbol" errors**  
Are these your libft functions? If so, you probably need to follow the installation directions under 'Installation' in order to include your libft.a separately from libftprintf.a  

**Help! Wildcard search isn't working!**  
For almost all shell terminals, the `*` needs to be escaped--usually, putting a string in double quotes is sufficient, but some terminals still treat it as a shell `*` even then. You can either escape it manually '`\*`', or, to make this feature compatible with all shells, I've made **any character not valid for a C function name (alphanumeric + underscore) is now considered a wildcard**. This means instead of `\*`, you can also use `@`, or anything else your terminal doesn't recognize as a special character. The same is true for the enable-test and disable-test scripts.

### Other Issues

If something goes wrong--slack me @gfielder. I like testing, like people using good testing, and want to make this easier to use, so don't hesitate to contact me.    

# Contributing

I encourage everyone to contribute to this, even if it's just adding tests to the library. To do this, fork and make pull requests.   

Before making pull requests, please:

```bash
./enable-test && ./disable-test argnum && ./disable-test moul_notmandatory \
&& ./disable-test nocrash && ./disable-test moul_D && ./disable-test moul_F \
&& ./disable-test f_reserved_values_ && ./disable-test f_L_reserved_values_
```
*and if you add non-mandatory test cases or tests that can segfault, modify this block in the readme*

# Possible Future Features

I have a few ideas how to improve this:

- The unit test library could stand to be a little more consistent and concise with naming conventions. 
- There are very few tests for the `*` bonus.  
- Could add tests for the thousands separator optional format flag.
- Could add tests for the `n` specifier.

Feel free to give me suggestions, or code them yourself and make a pull request.  

# Credits

The test method itself was adapted from outdated moulinette test files a buddy gave me, from which the author was ly@42.fr. The vast majority of code was written by me. The tests prefixed moul\_ were adapted from the moulinette test files, the tests with \_ftfc\_ were adapted from 42FileChecker, and all other tests (so far) were written by me.

Also thanks to:
- [rwright](https://github.com/wright08)
- phtruong
- [osfally](https://github.com/shaparder)
- [dfonarev](https://github.com/ruv1nce)  
for various suggestions and feature motivations.


