Writing your first script
=========================

## Introduction

This tutorial will guide you through discovering Quixote and its concepts, in order to allow you to write your own moulinettes easily and painlessly.

Its only pre-requisites are a working installation of Quixote (see [Installing]()), and basic knowledge of programming.

## What is a moulinette ?

A moulinette describes operations that should be applied to a particular piece of data, in order to produce a result. A typical example would be in the context of the validation of an exercise.


With **Quixote**, a moulinette is a set of scripts and resources specifying the job's execution environment and behavior.
The file that compute all of the above is called a **blueprint**.

The scripts must specify the three following things:

- How the job's environment should be configured (installed applications, dependencies, ...)
- How the job should fetch the data to inspect.
- How the job should inspect the data

## How to write the blueprint

the blueprint is composed of two elements

the blueprint description which is done by instanciating a blueprint class

```python
import quixote

blueprint = quixote.Blueprint(
    name="my_is_even_demo_moulinette",
    author="don quixote",
)
```
and the diferent stages of the moulinette which you provide by defining a function and decorate it with one of the available quixote decorators

- `@quixote.builder`
- `@quixote.fetcher`
- `@quixote.inspector`

## Setup step

In this steps the function will describe the environment and configuration required in order to run the moulinette

in our example of a my_is_even moulinette we only need to install gcc to compile the student turn-in and our correction.

```python
import quixote.setup as setup

@quixote.builder
def install_env():
    setup.apt.update()
    setup.apt.install("gcc")
```

## Fetch step

This step's functions describe how to fetch data to inspect from the student turn-in, here our moulinette mark a gitlab project so we're gonna use the gitlab built-in

```python
import quixote.fetch as fetch

@quixote.fetcher
def fetch():
    fetch.gitlab()
```

### Inspection step

This step's function describe the core purpose of the moulinnette which are the tests to be perform on the student turn-in

we're gonna make two function one to compile the correction and the student turn-in, and one to perform the actual test.
for the compilation we're gonna check that both program compile, if they fail an error will be triggered, and the moulinette will stop and grade the excercice with a 0

```python
import quixote.inspection.build as build
import quixote.inspection.check as check

@quixote.inspector(critical = True)
def compile():
    resources_path = get_context()["resources_path"]
    rendu = get_context()["delivery_path"]
    
    build.gcc([f"{resources_path}/cor_my_is_even.c", f"{resources_path}/main.c"], output_file="correction").check(exception = check.InternalError, reason="Internal error, compilation of the correction failed")
    build.gcc([f"{rendu}/my_is_even.c", f"{resources_path}/main.c"], output_file="student").check(reason="Compile error, we cannot compile your turn-in")
```
**Note** : the critical argument in the decorator specify that if this function fail for whatever reason (the student program fail to compile for example) the inspection will not go further and the moulinette will stop its work  

for the test a simple assertion to check that the output of the program match the expected output from our correction is required.
if the assertion failed 0 will automatically ba attribued to the exercice otherwise 1 will be marked.

```python
@quixote.inspector
def test():
    res = inspection.check.diff_process("student", "correction")
    check.assert_true(res.return_code == 0, f"Your ouptut and ours differ :\n{res.stdout}")
```

And thats it, the blueprint is ready our moulinette can now be tested and runned

## blueprint

this is what our blueprint should look like once we took care of all those steps

```python
import quixote
import quixote.setup as setup
import quixote.fetch as fetch
import quixote.inspection.build as build
import quixote.inspection.check as check


@quixote.builder
def install_env():
    setup.apt.update()
    setup.apt.install("gcc")

@quixote.fetcher
def fetch():
    fetch.gitlab()

@quixote.inspector(critical = True)
def compile():
    resources_path = quixote.get_context()["resources_path"]
    rendu = quixote.get_context()["delivery_path"]
    
    build.gcc([f"{resources_path}/cor_my_is_even.c", f"{resources_path}/main.c"], output_file="correction").check(exception = check.InternalError, reason="Internal error, compilation of the correction failed")
    build.gcc([f"{rendu}/my_is_even.c", f"{resources_path}/main.c"], output_file="student").check(reason="Compile error, we cannot compile your turn-in")

@quixote.inspector
def test():
    res = inspection.check.diff_process("student", "correction")
    check.assert_true(res.return_code == 0, f"Your ouptut and ours differ :\n{res.stdout}")
    

blueprint = quixote.Blueprint(
    name="my_is_even_demo_moulinette",
    author="don quixote"
)
```
