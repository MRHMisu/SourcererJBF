# SourcererJBF:  A Java Build Framework For Large Scale Compilation

SourcererJBF or JBF is a build framework that is capable of building thousand of Java projects at scale.
JBF first takes a vast collection of Java projects as input and scrapes all the required external dependencies from
those projects or the web. Then it indexes these dependencies and compiles the projects in multiple stages. During the
compilation, JBF
fixes errors and resolves external dependencies.

<img src="doc/jbf-overview.png" alt="JBF High Level Architecture"/>


### Environment Setup & Requirements

- Java Version: JDK-8+ [Preferable Latest Java [OpenJDK17](https://openjdk.java.net/projects/jdk/17/)]
- Ant Version: Ant 1.10 works with ``javac`` from JDK-8+ [Ant](https://ant.apache.org/manual/install.html)
- Python Version: 3.9+
- JBF uses three python packages [subprocess32](https://pypi.org/project/subprocess32/), [chardet](https://pypi.org/project/chardet/) and [simplejson](https://pypi.org/project/simplejson/). The following python packages are
  required to install before running JBF:

```
pip install subprocess32
pip install chardet
pip install simplejson
```

### Directories and Files Structure

```
📦 SourcererJBF
   ┃ 📂 doc                           // Resource for project documentation
   ┃ 📂 env-test                     // Resource for test the JBF workflow
   ┣ 📂 sourcererjbf                // The python package with scripts for building the projects
   ┃ 📂 utils                      // The utlity package with scripts for analyzing Jars and projects
   ┃ 📂 xml-templates             // The templates for creating normalized build scripts
   📜 clean-up.sh                // Script for cleaning all generated files & folders       
   📜 jbf.config                // Contains the configuration of JBF execution
   📜 jbf-cmd-compile.py       // Main Entry point of JBF execution with Command Line Arguments
   📜 jbf-config-compile.py   // Main Entry point of JBF execution with jbf.config
   📜 README.md              // JBF documentation
```

### Executing JBF
JBF can be run with a configuration file or with the command line arguments

- #### Run JBF With Configuration File
The easiest option is to edit the ``jbf.config`` configuration file and execute the ``jbf-config-compile.py`` script.
The file is self-explanatory, and it just requires to update according to host machine physical paths.

- #### ``jbf.config``
``` yml
[DEFAULT]
# The directory under which all the java projects to be compiled exist.
root =./env-test/projects
# Rebuild the projects from scratch. Dependency rematching implies that all projects might not recompile successfully.
rebuild_from_scratch = True
# The file with project paths to be build. Paths in file are considered relative to the root directory.
file = AUTOGEN
# The directory under which all the output build directories will be put.
outfolder = ./env-test/builds/
# An output file that will contain all the output information consolidated.
output = ./env-test/project_details.json
# The root of the jar collection repository
jars =./env-test/jars
# The file that represents the mapping of fqn to jar in repository.
fqn_to_jar = ./env-test/fqn-to-jars.shelve
# The number of base threads to be run.
threads = 1
try_project_build = False
verbose = True
only_project_build = False
```

```bash 
python3 jbf-config-compile.py
```
- #### Run JBF With Command Line Arguments
If you prefer to run JBF with command line arguments, you can get the details of these arguments with help option.
```bash
python3 jbf-cmd-compile.py -h
```
```bash
usage: jbf-cmd-compile.py [-h] [-r ROOT] [-b] [-f FILE] [-d OUTFOLDER] [-o OUTPUT] [-j JARS] [-ftj FQN_TO_JAR] [-t THREADS] [-tpb] [-v] [-opb]

optional arguments:
  -h, --help            show this help message and exit
  -r ROOT, --root ROOT  The directory under which all the java projects to be compiled exist.
  -b, --rebuild_from_scratch
                        Rebuild the projects from scratch. Dependency rematching implies that all projects might not recompile successfully.
  -f FILE, --file FILE  The file with project paths to be build. Paths in file are considered relative to the root directory.
  -d OUTFOLDER, --outfolder OUTFOLDER
                        The directory under which all the output build directories will be put.
  -o OUTPUT, --output OUTPUT
                        An output file that will contain all the output information consolidated.
  -j JARS, --jars JARS  The root of the java repository
  -ftj FQN_TO_JAR, --fqn_to_jar FQN_TO_JAR
                        The file that represents the mapping of fqn to jar in repository.
  -t THREADS, --threads THREADS
                        The number of base threads to be run.
  -tpb, --try_project_build
                        Use project build files first if it exists.
  -v, --verbose         Forces javac output to be verbose. Default False
  -opb, --only_project_build
                        Only use project build files.
```

```bash
python3 jbf-cmd-compile.py [-h] [-r ROOT] [-b] [-f FILE] [-d OUTFOLDER] [-o OUTPUT]
[-j JARS] [-ftj FQN_TO_JAR] [-t THREADS]
```

### The Test Environment and JBF Generated Directories, Files 
```
📦 SourcererJBF
  ┣ 📂 env-test                       // Resource for test the JBF workflow
      ┣ 📂 projects                          // All the projects that can be build. There are at most 1000 projects in each folder in projects
      ┣ 📂 jars                             // Collection of jars representing external dependencies of the porjects
      ┣ 📂 builds                          // The output of the build process. Generated following the same heirarchy that is similar to ┣📂 projects/
      📜 fqn-to-jars.shelve           // (Will appear after JBF execution) The global mapping of FQNs to jars, from the central ┣ 📂 jars/ collection
      📜 project_details.json        // (Will appear after JBF execution) Bookkeeping files, details for the projects with all the detias of build process     
  ┣ 📂 TBUILD                    // (Will appear after JBF execution) JBF genetered temporary build folders
  ┃ 📂 Uncompress               // (Will appear after JBF execution) JBF genetered temporary folder used to unzip the project files from their zip archives
  📜 *.log                     // (Will appear after JBF execution) Log files generated by worker threads in case of failures
  📜 save_*.shelve            // (Will appear after JBF execution) JBF genetered temporary mapping of FQNs to jars
  📜 badjars_*               // (Will appear after JBF execution) JBF genetered temporary list of invalid jars files
```

#### Note:
Please delete all these generated files & folders before each new execution of JBF.
```bash
./clean-up.sh
```


### JBF Works in Action

https://user-images.githubusercontent.com/6449333/170950780-6e1f03a2-832f-416e-a531-92da32c6a33b.mp4



### Build as a Service (BaaS)
Following the JBF methodology, we also designed a Build as a Service (BaaS), that can instantly build a Java project hosted on GitHub.
To learn more about it, please check out this repository [BaaS](https://github.com/Mondego/baas).

