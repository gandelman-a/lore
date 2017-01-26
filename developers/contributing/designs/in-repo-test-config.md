# In-repo test configuration

## Problem Statement

Historically, Zuul uses a configuration file to define which jobs run against which projects.  Each defined job can contain a snippet of shell code to run any arbitrary command or script on the test node to invoke specific test actions against the change under test.  Jobs can also rely on macros to invoke known test suite entry points (ie, tox).

In order to add new jobs to a project being tested by BonnyCI, the Zuul configuration needs to be updated on the Zuul server by BonnyCI admins.  Users of BonnyCI would need to file issues or tickets to get the main Zuul config updated to suit their needs.  This would be a slow process that does not scale, and would make it difficult for new users of BonnyCI to experiment and iterate quickly as they migrate their projects to using BonnyCI.

Other tools such as Travis allow project repositories to carry a .travis.yml file which defines the entire test matrix for the project.  Similar functionality in Zuul is in the works but will not be available until Zuul v3.  Until then, we should provide a flexible way for project maintainers to freely define tests that run against changes.

Additionally, the way in which BonnyCI's Zuul collects test output is dependent on hard-coded config on the server-side.  We need to provide users who are defining their own custom test execution with a standard way of exporting test output so that it may be collected and stored on a log server for post-run viewing.

## Overview

A simple shell interface is provided to users that allows them to store a shell script at a known location in the project repo.  This script is responsible for running the tests shipped within that repository against the change under test. Users are free to define exactly what that script executes so long as it exits with a return code of  0 for a successful test run. Any other return code will be considered a failed run.  Information will be provided to the script, via command-line arguments and environment variables, that describe which stage of testing is running the script (gate, check, etc) and where logs may be stored for publishing by the BonnyCI infrastructure.

On the Zuul side, a single job template defines the necessary builders for executing the test script and publishers for reporting the test output and logs. This job template can be used to generically add jobs to new projects' pipelines. This will help standardize project configuration and streamline the onboarding of new projects into BonnyCI.

### In-repo testing script

#### Location

Projects will create a ``.bonnyci`` subdirectory at the root of the repository tree.  This directory may contain anything useful for executing tests (ie, shell libraries or helper scripts), but if tests are to be run it will need to contain an executable file ``run.sh`` that serves as an entry point to the test run.  This script is user-defined and may call out to any other scripts stored in the repo, or to binaries and scripts available elsewhere on the test slave (ie, virtualenv or pip).

#### Interface

The ``run.sh`` script is free to define any steps necessary to run the projects' tests.  It is required that its permissions allow it to be executed and it contains the required ``#!/bin/bash`` shebang.  Additionally, the script's return code is interpretted by BonnyCI and should be one of either:

* 0 - This signifies the test run executed successfully and without errors and that the test job has passed.
* non-zero - This signifies a test failure or error and that the test job has failed.

The ``run.sh`` script will be used to invoke tests across both gate and check pipelines, and possibly other pipelines in the future.  Users may wish to differentiate their test runs between pipelines.  For example, a project may only run linting and unit tests during the initial check, but additionally run functional or integration test as part of gating.  To allow this flexibility, the pipeline in which the test is being run will be exported to the script via environment variable.  The ``BONNYCI_TEST_PIPELINE`` variable will be set to the pipeline currently running, currently either ``check`` or ``gate``.  Note that these pipeline names are user-facing and distinct from the names of the internal pipelines configured in Zuul's configuration.

Regardless of a job's passing or failing status, BonnyCI will publish console output and, optionally, any test logs made available by the project's test suite.  User's are responsible for ensuring their ``run.sh`` places any log files in the correct location on the file system for post-run archiving.  The location of this directory will be made available to the script via a ``BONNYCI_TEST_LOG_DIR`` environment variable and it can be assumed that this directory exists and is writable by the user executing tests.

### Zuul Configuration

Our Zuul job configuration will contain a single bonnyci job template that defines:

* A publisher that publishes console logs and test log files
* Call any builders required for prepraring source tree or any other test dependency, ie ``zuul-git-prep``
* A builder that executes the ``.bonnyci/run.sh`` script if it exists in the repository.  If it does not exist, the builder will exit 1 and the test will fail.  In addition to executing the script, it will also export the ``BONNYCI_TEST_LOG_DIR`` and ``BONNYCI_TEST_PIPELINE`` with the correct settings.

New projects added to BonnyCI can then use a standard block of yaml to configure the project itself, with generic references to the job template for both the check and gate pipelines.
