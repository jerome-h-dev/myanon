# Myanon

Myanon is a MySQL dump anonymizer, reading a dump from stdin, and producing an anonymized version to stdout.

Anonymization is done through a deterministic hmac processing based on sha-256. When used on fields acting as foreign keys, constraints are kept.

A configuration file is used to store the hmac secret and to select which fields need to be anonymized. A self-commented sample is provided (main/myanon-sample.conf)

This tool is in alpha stage. Please report any issue.

## Simple use case

Example to create both a real crypted (sensitive) backup and an anonymized (non-sentitive) backup from a single mysqldump command:

```
mysqldump mydb | tee >(myanon -f myanon.cfg | gzip > mydb_anon.sql.gz) | gpg -e -r me@domain.com > mydb.sql.gz.gpg
```

## Installation from sources

### Build Requirements

- autoconf 
- automake 
- make
- a C compiler (gcc or clang)
- flex 
- bison

Example on a Fedora system: 

```shell
$ sudo dnf install autoconf automake gcc make flex bison
[...]
```
Example on a Debian/Ubuntu system:

```shell
$sudo apt-get install autoconf automake flex bison build-essential
[...]
```
On macOS, you need to install Xcode and homebrew, and then:
```shell
$ brew install autoconf automake flex bison m4
[...]
```

(Please ensure binaries installed by brew are in your $PATH)

If your using zsh, you may need to add the following to your .zshrc file:

```shell
export PATH="/usr/local/opt/m4/bin:$PATH"
export PATH="/usr/local/opt/flex/bin:$PATH"
export PATH="/usr/local/opt/bison/bin:$PATH"
```

### Build/Install

```
./autogen.sh
./configure
make
make install
```

### Compilation/link flags

Flags are controlled by using CFLAGS/LDFLAGS when invoking make.
To create a debug build:
```
make CFLAGS="-O0 -g"
```

To create a static build on Linux:
```
make LDFLAGS="-static"
```


### Run/Tests
```
main/myanon -f tests/test1.conf < tests/test1.sql
zcat tests/test2.sql.gz | main/myanon -f tests/test2.conf
```

## Installation from packages (Ubuntu)

A PPA is available at: https://launchpad.net/~pierrepomes/+archive/ubuntu/myanon

## Docker Build / Run

### tl;dr: 

```shell
docker build --tag myanon .
docker run -it --rm -v ${PWD}:/app myanon sh -c '/bin/myanon -f /app/myanon.conf < /app/dump.sql | gzip > /app/dump-anon.sql.gz'
```

### Why Docker?
An alternative to the above build or run options is to use the provided Dockerfile to build inside an isolated environment, and run `myanon` from a container. 

It's useful when:

* you can't or don't want to install a full C development environment on your host
* you want to quickly build for or run on a different architecture (e.g.: `amd64` or `arm64`)
* you want to easily distribute a self-contained `myanon` (e.g.: for remote execution & processing on a Kubernetes cluster)

The provided multistage build `Dockerfile` is using the official [`gcc` Docker image](https://hub.docker.com/_/gcc) for the *build* phase and the [`alpine` Docker image](https://hub.docker.com/_/alpine/) for runtime (some `myanon` use-cases need a shell, so a *distroless* base image would not work here). 

### Build using Docker

Build a static binary using the provided `Dockerfile`: 

```shell
# recommended, to start from a clean state 
make clean
# build using your default architecture
docker build --tag myanon .
```

For Apple Silicon users who want to build for `amd64`:

```shell
# recommended, to start from a clean state 
make clean
# build using the amd64 architecture
docker build --tag myanon --platform=linux/amd64 .
```

### Run using Docker

In this example we will:

* use a `myanon` configuration file (`myanon.conf`)
* use a MySQL dump (`dump.sql`)
* generate an anonymized dump (`dump-anon.sql`) based on the configuration and the full dump.

Sharing the local folder as `/app` on the Docker host: 

```shell
docker run -it --rm -v ${PWD}:/app myanon sh -c '/bin/myanon -f /app/myanon.conf < /app/dump.sql > /app/dump-anon.sql'
```

For Apple Silicon users who want to run as `amd64`: 

```shell
docker run -it --rm --platform linux/amd64 -v ${PWD}:/app myanon sh -c '/bin/myanon -f /app/myanon.conf < /app/dump.sql > /app/dump-anon.sql' 
```

Refer to the different options from [the documentation above](https://github.com/ppomes/myanon#simple-use-case) for detailed usage options.

