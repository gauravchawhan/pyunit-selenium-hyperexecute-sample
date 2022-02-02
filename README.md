# How to run Selenium automation tests on HyperTest (using PyUnit framework)

Download the concierge binary corresponding to the host operating system. It is recommended to download the binary in the project's parent directory.

* Mac: https://downloads.lambdatest.com/concierge/darwin/concierge
* Linux: https://downloads.lambdatest.com/concierge/linux/concierge
* Windows: https://downloads.lambdatest.com/concierge/windows/concierge.exe

[Note - The current project has concierge for macOS. Irrespective of the host OS, the concierge will auto-update whenever there is an updated version available on the server.]

Before the tests are run, please set the environment variables LT_USERNAME & LT_ACCESS_KEY from the terminal. The account details are available in your [LambdaTest Profile](https://accounts.lambdatest.com/detail/profile) page.

For Windows:

```bash
set LT_USERNAME=LT_USERNAME
set LT_ACCESS_KEY=LT_ACCESS_KEY
```

For macOS:

```bash
export LT_USERNAME=LT_USERNAME
export LT_ACCESS_KEY=LT_ACCESS_KEY
```

## Running tests in PyUnit using the Matrix strategy

Matrix YAML file (*yaml/pyunit_hypertest_matrix_sample.yaml*) in the repo contains the following configuration:

```yaml
globalTimeout: 90
testSuiteTimeout: 90
testSuiteStep: 90
```

Global timeout, testSuite timeout, and testSuite timeout are set to 90 minutes.
 
The target platform is set to Windows. Please set the *[os]* key to *[mac]* in case the tests have to be executed on the macOS platform. 

```yaml
os: [win]
```

Python files in the 'tests' folder contain the test suites that will be run on the HyperTest grid. In the example, the tests in the files *tests/lt_test_suite_runner_1.py* and *tests/lt_test_suite_runner_2.py* run in parallel using the specified input combinations.

```yaml
files: ["tests/lt_test_suite_runner_1.py", "tests/lt_test_suite_runner_2.py"]
```

### Matrix Execution: Pre, Post, and Dependency Caching for faster package download & installation

To ensure that the package dependencies are not downloaded in subsequent runs, dependency caching is enabled in the YAML file. The first step is to set the Key which is used to cache directories.

```yaml
cacheKey: '{{ checksum "requirements.txt" }}'
```

Set the array of files & directories to be cached. In the example, all the packages will be cached in the *CacheDir* directory.

```yaml
cacheDirectories:
  - CacheDir
```

Steps (or commands) that need to be run before the test execution are listed in the *pre* run step. In the example, the packages listed in *requirements.txt* are installed using *pip3* command. The *--cache-dir* option is used for specifying the location of the directory used for caching the packages (i.e. *CacheDir*). It is important to note that downloaded packages that are cached are securely uploaded to a secure clooud, before the execution environment is auto-purged after build completion. Please modify *requirements.txt* as per the project requirements.

```yaml
pre:
  - pip3 install -r requirements.txt --cache-dir CacheDir
```

Steps (or commands) that need to run after the test executiona are listed in the *post* step. In the example, we cat the contents of *yaml/pyunit_hypertest_matrix_sample.yaml*

```yaml
post:
  - cat yaml/pyunit_hypertest_matrix_sample.yaml
```

The *upload* directive informs HyperTest to upload Artefacts [files, reports, etc.] that are generated after task completion. In the example, the *reports* folder that contains the test reports will be uploaded by HyperTest. 

```yaml
upload:
  - reports/
```

The *testSuites* object contains a list of commands (that can be presented in an array). In the current YAML file, commands to be run for executing the tests are put in an array (with a '-' preceding each item). In the current YAML file, Python command is used to run tests in *.py* files. The files are mentioned as array to *files* key that is a part of the matrix.

```yaml
testSuites:
  - python3 -s  $files
```

The CLI option *--config* is used for providing the custom HyperTest YAML file (i.e. yaml/pyunit_hypertest_matrix_sample.yaml). Run the following command on the terminal to trigger the tests in Python files on the HyperTest grid. The *--download-artifacts* option is used to inform HyperTest to downoad the artifacts for the job.

```bash
./concierge --download-artifacts --config yaml/pyunit_hypertest_autosplit_sample.yaml --verbose
```

Visit [HyperTest Automation Dashboard](https://automation.lambdatest.com/hypertest) to check the status of execution:



## Running tests using PyUnit using the Auto-Split strategy

Auto-Split YAML file (pyunit_hypertest_autosplit_sample.yaml) in the repo contains the following configuration:

```yaml
globalTimeout: 90
testSuiteTimeout: 90
testSuiteStep: 90
```

Global timeout, testSuite timeout, and testSuite timeout are set to 90 minutes.
 
The *runson* key determines the platform (or operating system) on which the tests would be executed. Here we have set the target OS as Windows.

```yaml
runson: win
```

Auto-split is set to true in the YAML file.

```yaml
 autosplit: true
``` 

*retryOnFailure* is set to true which instructs HyperTest to retry failed command(s). The retry operation is carried out till the number of retries mentioned in *maxRetries* are exhausted or the command execution results in a pass. The concurrency (i.e. number of parallel sessions) is set to 1.

```yaml
  retryOnFailure: true
  maxRetries: 5
  concurrency: 1
```

Environment variables *LT_USERNAME* and *LT_ACCESS_KEY* are added under *env* directive. The user_name and access_key to access the LambdaTest platform can be found in your [LambdaTest Profile](https://accounts.lambdatest.com/detail/profile) page. Any more environment variables can be added in this section.

```yaml
env:
  LT_USERNAME: LT_USERNAME
  LT_ACCESS_KEY: LT_ACCESS_KEY
```

### Auto-Split Execution: Pre, Post, and Dependency Caching for faster package download & installation

To leverage the advantage offered by *Dependency Caching* in HyperTest, we first check the integrity of *requirements.txt* using checksum functionality

```yaml
cacheKey: '{{ checksum "requirements.txt" }}'
```

By default, *pip* in Python saves the downloaded packages in the cache so that next time, the package download request can be serviced from the cache (rather than re-downloading it again). The caching advantage offered by *pip* can be leveraged in HyperTest whereby the downloaded packages can be stored (or cached) in a secure server for future executions. The packages available in the cache will only be used if the checksum stage results in a Pass.

The *cacheDirectories* directive is used for specifying the directory where the packages have to be cached. The mentioned directory will override the default directory where Python packages are normally cached, further information about Caching in pip is available [here](https://pip.pypa.io/en/stable/cli/pip_cache/). The packages downloaded using pip will be cached in the directory (or location) mentioned under the *cacheDirectories* directive.

In our case, the downloaded packages are cached in *CacheDir* folder in the project's root. The folder is automatically created when the packages mentioned in *requirements.txt* are downloaded.  

```yaml
cacheDirectories:
  - CacheDir
```

Content under the *pre* directive is the pre-condition that will be run before the tests are executed on HyperTest grid. The *--cache-dir* option in *pip3* is used for specifying the cache directory. It is important to note that downloaded packages that are cached are securely uploaded to a secure upload, before the execution environment is auto-purged after build completion. Please modify *requirements.txt* as per the project requirements.

```yaml
pip3 install -r requirements.txt  --cache-dir CacheDir
```

The *post* directive contains a list of commands that run as a part of post-test execution. Here, the contents of *yaml/pyunit_hypertest_autosplit_sample.yaml* are read using cat as a part of post step. 

```yaml
post:
  - cat yaml/pyunit_hypertest_autosplit_sample.yaml
```

The *testDiscoverer* directive contains the command that gives details of the tests that are a part of the project. Here, we are fetching the list of Python files that would be further executed using the *value* passed in the *testRunnerCommand*

```bash
grep -nri 'HTML_TestRunner_TestSuite' tests -ir --include=\*.py | sed 's/:.*//'
```

Running the above command on the terminal will give a list of Python files that are located in the Project folder:

* tests/lt_test_suite_runner.py

The *testRunnerCommand* contains the command that is used for triggering the test. The output fetched from the *testDiscoverer* command acts as an input to the *testRunner* command.

```yaml
testRunnerCommand: python3 -s $test
```

Run the following command on the terminal to trigger the tests in Python files on the HyperTest grid.

```bash
./concierge --user LT_USERNAME --key LT_ACCESS_KEY --config yaml/pyunit_hypertest_autosplit_sample.yaml --verbose
```

Visit [HyperTest Automation Dashboard](https://automation.lambdatest.com/hypertest) to check the status of execution
