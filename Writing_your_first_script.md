Writing your first script
=========================

## Introduction

This tutorial will guide you through discovering Quixote and its concepts, in order to allow you to write your own moulinettes easily and painlessly.

Its only pre-requisites are a working installation of Quixote (see [Installing]()), and basic knowledge of programming.

## What is a moulinette ?

A moulinette describes operations that should be applied to a particular piece of data, in order to produce a result. A typical example would be in the context of the validation of an exercise. The mou



With **Quixote**, a moulinette is a set of scripts and resources specifying the job's execution environment and behavior.

The scripts must specify the three following things:

- How the job's environment should be configured (installed applications, dependencies, ...)
- How the job should fetch the data to analyze.