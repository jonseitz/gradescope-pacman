# GradeScope Autograder Submission notes

_The uploaded zip can have any name, but throughout this guide it will be called `autograder.zip`_

At minimum, `autograder.zip` must have two files:

`setup.sh` - An initial configuration script
`run_autograder` - A script that executes the autograder

When you upload `autograder.zip`, GradeScope will create a Docker image and expand your code into the following directory structure:

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

## setup.sh

This must be a shell script, and will execute as root (no need to `sudo`) only once, during the Docker image creation.

Default environment is Ubuntu 16.04, so use `apt-get` to install necessary packages, etc. The defuault `python`/`pip` commands use python 2.7. Python 3 is installed, so any p3 specifics should use `python3`/`pip3`.

`apt-get update` will be called before your script executes, so no need to call it before installing packages

## run_autograder

This can be any sort of script (bash, python, etc.) as long as it starts with a compatible shebang. 

It will be called any time a student submits the assignment so it should ideally be lightweight--any dependencies should be installed in `setup.sh` 

It will execute from `/autograder`, so any references to other files will need to be prefixed by the correct directory.

Whatever steps involved, your script should create a compatible JSON file in `/autograder/results/results.json`. See [Output Format](https://gradescope-autograders.readthedocs.io/en/latest/specs/#output-format) in GradeScope docs.

## Uploading autograder

Gradescope wants to receive a zip file with your `setup.sh` and `run_autograder` scripts in the root. I recommend adding over code in a separate directory. Be careful with how directories are handled when creating the zip file, as you want to make sure there isn't another directory above your scripts. In Linux, run:

```sh
$ zip -r autograder.zip setup.sh run_autograder code/*
```

## Uploading submissions

Students can submit either a zip of all files, or individual files.  

## Pacman AI

_As a test case I used the first "Search" exercise from the [Berkeley Pacman AI problem set](http://ai.berkeley.edu/search.html)_

The Pacman autograder can be called with `--gradescope-output`, which will create a file called `gradescope_response.json` in the working directory. The default output is very minimal, and only shows the student their score. If you want to make `stdout` from the autograder execution visible to students, you will need to add `"stdout_visibility": "visible"` to the top level of the JSON object (other values include `hidden`, `after_due_date`, `after_published`). In a bash script, this can be done with `sed`:

```sh
$ sed -e '0,/{/s/{/{"stdout\_visibility": "visible", /' ./gradescope\_response.json > /autograder/results/results.json
```

The autograder lives alongside the problemset code so students can test their own work along the way. However, I would recommend only having students upload the files they edit as a submission, and including the full code in the original autograder.zip upload. You can specify the student files that you want to run with the `--student-code` flag and a list of filenames, (**making sure to properly prepend `/autograder/submission`**). Thus a miminal run_autograder script for this problem would be:

```sh
#!/bin/sh
cd /autograder/source/code
python ./autograder.py --gradescope-output --student-code=/autograder/submission/search.py,/autograder/submission/searchAgents.py
sed -e '0,/{/s/{/{"stdout_visibility": "visible", /' ./gradescope_response.json > /autograder/results/results.json
```

Alternatively, you can have students submit the entire project directory as a zip file, but this will take longer to upload and does open up the possibility of students hardcoding answers or otherwise hacking the tests.

