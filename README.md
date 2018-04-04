rundoc
==================================================
[![PyPI version](https://badge.fury.io/py/rundoc.svg)](https://badge.fury.io/py/rundoc)
[![Build Status](https://travis-ci.org/EclecticIQ/rundoc.png)](https://travis-ci.org/EclecticIQ/rundoc)
[![Requirements Status](https://requires.io/github/EclecticIQ/rundoc/requirements.svg?branch=dev)](https://requires.io/github/EclecticIQ/rundoc/requirements/?branch=dev)
[![Documentation Status](https://readthedocs.org/projects/rundoc/badge/?version=latest)](http://rundoc.readthedocs.io/en/latest/?badge=latest)
[![Code Health](https://landscape.io/github/EclecticIQ/rundoc/master/landscape.svg?style=flat)](https://landscape.io/github/EclecticIQ/rundoc/master)

Run code blocks from documentation written in markdown.

Installation
-------------------------

### install from pypi
`pip3 install rundoc`

### install from github
`pip3 install -U git+https://github.com/EclecticIQ/rundoc.git`

Usage
-------------------------

Rundoc collects fenced code blocks from input markdown file and executes them in same order as they appear in the file.

Example of fenced code block in markdown file:

~~~markdown
 ```bash
 for x in `seq 0 10`; do
     echo $x
     sleep 1
 done
 ```
~~~

Interpreter will be automatically selected using the highlight tag of the code block (in our example `bash`). If highlight tag is not specified, bash will be used by default.

You can also replay all the actions using the output from running a markdown file. Replay will execute all commands as found in the output file.

### Run markdown file

Execute code blocks in *input.md* file:

```bash
rundoc run input.md
```

- You will be prompted before executing each code block with ability to modify the input.
- When done reviewing/modifying the code block, press *Return* to execute it and move to the next one.
- Program will exit when last code block is finished executing or when you press **ctrl+c**.

#### Skip prompts

You can use `-y` option to skip prompts and execute all code blocks without user interaction:

```bash
rundoc run -y input.md
```

If you need to add a delay between codeblocks, you can add `-p` or `--pause` option to specify number of seconds for the puase. This works only in conjunction with `-y`:

```bash
rundoc run -y -p 2 input.md
```

Some step fails first couple of times but it's normal? That may happen and you would just want to retry that step a couple of times. To do so use `-r` or `--retry` option followed by max number of retries and rundoc will run the same step again until it succeeds or reaches max retries in which case it will exit:

```bash
rundoc run -y -r 10 input.md
```

But you don't want it to retry right away, correct? You can specify a delay between each try with `-P` (capital P) or `--retry-pause` option followed by number of seconds:

```bash
rundoc run -y -r 10 -P 2 input.md
```

#### Start from specific step

You can start at specified step using `-s` or `--step` option:

```bash
rundoc run -s5 input.md
```

This is useful when your N-th code block fails and rundoc exits and you want to continue from that step.

#### Save output

Output can be saved as a json file containing these fields:

- `env` (dict): dictionary of set environment variables for the session
- `code_blocks` (list): list of code blocks that contains the following
    - `code` (str): original code
    - `interpreter` (str): interpreter used for this code block
    - `runs` (list): list of run attempts
        - `output` (str): merged stdout and stderr of the code block execution
        - `retcode` (int): exit code of the code block
        - `time_start` (float): timestamp when execution started (seconds from epoch)
        - `time_stop` (float): timestamp when execution finished (seconds from epoch)
        - `user_code` (str): code that user actually executed with prompt

To save output use `-o` or `--output` option when running rundoc:

```bash
rundoc run -y input.md -o output.json
```

#### Tags

By default, rundoc executes all fenced code blocks. If you want to limit execution to subset of the code blocks, use tags. Tags can be specified with `-t` or `--tags` option followed by hash (#) separated list of tags:

```bash
rundoc run -t bash#python3 input.md
```

This will execute only those code blocks that have specified highlight tag.

If you want to further isolate code blocks of the same highlight tag, you can use rundoc tag syntax, e.g.:

~~~markdown
 ```bash#custom-branch#v2#test
 echo "custom-tagged code block"
 ```
~~~

In this syntax, multiple tags are applied to same code block and are separated with hash symbol `#`. In the example above there are 4 tags: `bash`, `custom-branch`, `v2` and `test`. First tag always defines the interpreter. If any of it's tags is specified by `--tags` option, it will be executed. Code blocks that do not contain any of the specified tags will be skipped.

#### Environment variables

You can define required environment variables anywhere in the documentaion as a special code block tagged as `env` or `environment` at the beginning:

~~~markdown
 ```env#version5
 var1=
 var2=
 var3=default_value_3
 var4=default_value_4
 ```
~~~

- As in example above, define variables one on each line.
- When you run the docs you will be prompted for those.
- Empty values (e.g. `var1` and `var2` in example) will try to collect actual values from your system environment, so if `var1` was exported before you ran the docs, it will collect it's value as the default value.
- If you used `-y` option, you will be prompted only for variables that have empty values and are not exported in your current system environment.
- All variables will be passed to env for every code block that's being executed.
- If you use rundoc with tag option `-t`, environment blocks will be filtered in the same way as code blocks.

#### Secrets

You can define required credentials or other secrets anywhere in the documentaion as a special code block tagged as `secret` or `secrets` at the beginning:

~~~markdown
 ```secrets#production
 username=
 password=
 ```
~~~

Secrets behave just as `env` blocks with one single difference: they are never saved in the output file and are **expected to be empty in markdown file** so that user must provide them during execution. If you want to use rundoc as part of automation and can't input secrets by hand, you can always export them beforehand and use `-i` option (see next section).

#### Force variable collection

You can force rundoc to check if any of the variables defined with `env` tag is already exported in your current system environment and use it's value instead of the one defined in markdown file. To do this use `-i` or `--inherit-env` option when running rundoc. The list of variables that is presented to you when you run rundoc and that will be used in the session will now contain values defined in the system environment.

```bash
export var2=system_value_2
export var3=system_value_3
rundoc run input.md -i
```

### Replay

To replay all code blocks found in output of `run` command, just use `replay` command like so:

```bash
rundoc replay output.json
```

The above command will just turn last runs of each code block into a new code block and run them without prompting you about anything (like having `-y` option). It will ignore all run tries that did not succeed. Last run may have original command or user modified one and replay does not think about that, it just runs the last command it finds in each code block.

You can still use `-p`, `-s`, `-o`, `-r`, and `-P` options with `replay` command:

```bash
rundoc replay -s 2 -p 1 -r 20 -P 5 output.json -o replay_output.json
```

Tips and tricks
-------------------------

### Color output

Are you using light terminal background and can't see sh\*t? Use rundoc with `--light` option and save your eyesight!


