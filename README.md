# Gradescope Autograder Submission notes

## Overview

Gradescope's Autograding system uses [Docker](https://www.docker.com/) to run student tests against student code in a sandboxed environment and report back on their grades.

You do not need to understand how Docker works to use autograder, though an understanding of the basics may make the process easier. To explain it at the highest level, Gradescope uses the files you upload to create a reusable execution environment with all of the code necessary to grade student work (i.e., building a Docker Image). Every time a student uploads code to Gradescope, it will start up that environment (i.e., creating a Docker Container) and runs a script you provide in that temporary environment.  

## Details

Setting up an autograder requires you to upload a zip file. It can have any name, but throughout this guide I will refer to it as `autograder.zip`.

At minimum, `autograder.zip` must have two files:

`setup.sh` - An initial configuration script
`run_autograder` - A script that executes the autograder

After uploading this file, GradeScope will create a Docker image and expand your code into the following directory structure:

```
/autograder
    /source
        ...uploaded autograder code
    /submission
        ...student code
    /results
        results.json    # Needs to be created by run_autograder
        stdout          # Captures all output from run_autograder
    run_autograder
```

### setup.sh

This must be a shell script, and will execute as root (no need to `sudo`) only once, during the Docker image creation.

Default environment is Ubuntu 16.04, so use `apt-get` to install necessary packages, etc. The defuault `python`/`pip` commands use python 2.7. Python 3 is also installed, and can be run with `python3`/`pip3`.

`apt-get update` will be called before your script executes, so no need to call it before installing packages

### run_autograder

This can be any sort of script (bash, python, etc.) as long as it starts with a compatible shebang (e.g. `#!/bin/sh`). 

`run_autograder` will be called any time a student submits their assignment. It will execute from `/autograder`, so any references to other files will need to be prefixed by the correct directory.

Your script should create a compatible JSON file in `/autograder/results/results.json`. See [Output Format](https://gradescope-autograders.readthedocs.io/en/latest/specs/#output-format) in GradeScope docs for more details about how it should be configured.

## Uploading Files

### Autograder

Gradescope wants to receive a zip file with your `setup.sh` and `run_autograder` scripts in the root. I recommend adding any code in a separate directory. Be careful with how directories are handled when creating the zip file so that you don't create an additional container folder inside the zip file. To create a zip from this git repository in Linux, run:

```sh
$ zip -r autograder.zip setup.sh run_autograder code/*
```

### Submissions

Students can submit either a zip of all their files, or upload individual files from the gradescope assignment page.  

# Pacman

_As a test case I used the first "Search" exercise from the [Berkeley Pacman AI problem set](http://ai.berkeley.edu/search.html). That code is in the `code/` directory._

The Pacman autograder can be called with the `--gradescope-output` flag, which will create a file called `gradescope_response.json` in the working directory. The default output fields only show the students their score. If you want to make the `stdout` from the autograder execution visible to students, you will need to add `"stdout_visibility": "visible"` to the top level of the JSON object (other values include `hidden`, `after_due_date`, `after_published`). In a bash script, this can be done with `sed`:

```sh
$ sed -e '0,/{/s/{/{"stdout\_visibility": "visible", /' ./gradescope\_response.json > /autograder/results/results.json
```

> NOTE: this also redirects the output to the final results file. Alternatively you can run `sed` with the `-i` flag to change the file in place, then copy to the results to the proper location.

The autograder code lives alongside the problemset code so students can test their own work along the way. However, I would recommend only having students upload the files they edit as a submission while you upload 0the full code in the original `autograder.zip` upload. You can then specify the student files that you want to run with the `--student-code` flag and a list of filenames, **making sure to properly prepend `/autograder/submission`**. This wil also ensure that only the correct files will be tested. 

Alternatively, you can have students submit the entire project directory as a zip file, but this will take longer to upload and does open up the possibility of students hardcoding answers or otherwise hacking the tests.

Thus a miminal run_autograder script for this problem set would be:

```sh
#!/bin/sh
cd /autograder/source/code
python ./autograder.py --gradescope-output --student-code=/autograder/submission/search.py,/autograder/submission/searchAgents.py
sed -e '0,/{/s/{/{"stdout_visibility": "visible", /' ./gradescope_response.json > /autograder/results/results.json
```

