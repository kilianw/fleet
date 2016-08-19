# Kolide [![CircleCI](https://circleci.com/gh/kolide/kolide-ose.svg?style=svg&circle-token=2573c239b7f18967040d2dec95ca5f71cfc90693)](https://circleci.com/gh/kolide/kolide-ose)

### Contents

- [Development Environment](#development-environment)
  - [Installing build dependencies](#installing-build-dependencies)
  - [Building](#building)
    - [Generating the packaged JavaScript](#generating-the-packaged-javascript)
    - [Compiling the kolide binary](#compiling-the-kolide-binary)
    - [Managing Go dependencies with glide](#managing-go-dependencies-with-glide)
    - [Automatic recompilation](#automatic-recompilation)
  - [Testing](#testing)
    - [Application tests](#application-tests)
    - [Viewing test coverage](#viewing-test-coverage)
    - [Testing email](#testing-email)
  - [Development Infrastructure](#development-infrastructure)
    - [Starting the local development environment](#starting-the-local-development-environment)
    - [Stopping the local development environment](#stopping-the-local-development-environment)
    - [Setting up the database tables](#setting-up-the-database-tables)
  - [Running Kolide](#running-kolide)


## Development Environment

### Installing build dependencies

To setup a working local development environment, you must install the following
minimum toolset:

* [Docker](https://www.docker.com/products/overview#/install_the_platform)
* [Go](https://golang.org/dl/) (1.6 or greater)
* [Node.js](https://nodejs.org/en/download/current/) (and npm)
* [GNU Make](https://www.gnu.org/software/make/)

Once you have those minimum requirements, to install build dependencies, run the following:

```
make deps
```

### Building

#### Generating the packaged JavaScript

To generate all necessary code (bundling JavaScript into Go, etc), run the
following:

```
go generate
```

#### Compiling the kolide binary

On UNIX (OS X, Linux, etc), run the following to compile the application:

```
go build -o kolide
```

On Windows, run the following to compile the application:

```
go build -o kolide.exe
```

#### Managing Go Dependencies with Glide

[Glide](https://github.com/Masterminds/glide#glide-vendor-package-management-for-golang)
is a package manager for third party Go libraries. See the ["How It Works"](https://github.com/Masterminds/glide#how-it-works)
section in the Glide README for full details.

##### Installing the correct versions of dependencies

To install the correct versions of third package libraries, use `glide install`. 
`glide install` will  use the `glide.lock` file to pull vendored packages from 
remote vcs.  `make deps` takes care of this step, as well as downloading the
latest version of glide for you.

##### Adding new dependencies

To add a new dependency, use [`glide get [package name]`](https://github.com/Masterminds/glide#glide-get-package-name)  

##### Updating dependencies

To update, use [`glide up`](https://github.com/Masterminds/glide#glide-update-aliased-to-up) which will use VCS and `glide.yaml` to figure out the correct updates.   

##### Testing application code with glide

Finally, you can use [`go test $(glide novendor)`](https://github.com/Masterminds/glide#glide-novendor-aliased-to-nv) to skip tests in the vendor directory. This will run go test over all directories of your project except the vendor directory.

#### Automatic recompilation

If you're editing mostly frontend JavaScript, you may want the Go binary to be
automatically recompiled with a new JavaScript bundle and restarted whenever 
you save a JavaScript file. To do this, instead of running `kolide serve`, run
the following:

```
make watch
```

This is only supported on OS X and Linux.

### Testing

#### Application tests

To run the application's tests, run the following from the root of the
repository:

```
go test
```

From the root, `go test` will run a test launcher that executes `go test` and 
`go vet` in the appropriate subpackages, etc. If you're working in a specific
subpackage, it's likely that you'll just want to iteratively run `go test` in
that subpackage directly until you are ready to run the full test suite.

#### Viewing test coverage

When you run `go test` from the root of the repository, test coverage reports
are generated in every subpackage. For example, the `sessions` subpackage will
have a coverage report generated in `./sessions/sessions.cover`

To explore a test coverage report on a line-by-line basis in the browser, run 
the following:

```bash
# substitute ./errors/errors.cover, ./app/app.cover, etc
go tool cover -html=./sessions/sessions.cover
```

To view test a test coverage report in a terminal, run the following:

```bash
# substitute ./errors/errors.cover, app/app.cover, etc
go tool cover -func=./sessions/sessions.cover
```

#### Testing Email

To intercept sent emails while running a Kolide development environment, make
sure that you've set the SMTP address to `<docker host ip>:1025` and leave the
username and password blank. Then, visit `<docker host ip>:8025` in a web 
browser to view the [MailHog](https://github.com/mailhog/MailHog) UI.

For example, if docker is running natively on your `localhost`, then your mail
settings should look something like:

```json
{
  "mail": {
    "address": "localhost:1025",
    "pool_connections": 4
  }
}
```

### Development infrastructure

#### Starting the local development environment

To set up a canonical development environment via docker,
run the following from the root of the repository:

```
docker-compose up
```

This requires that you have docker installed. At this point in time,
automatic configuration tools are not included with this project.


#### Stopping the local development environment

If you'd like to shut down the virtual infrastructure created by docker, run
the following from the root of the repository:

```
docker-compose down
```

#### Setting up the database tables

Once you `docker-compose up` and are running the databases, you can build
the code and run the following command to create the database tables:

```
kolide prepare db
```

### Running Kolide

Now you are prepared to run a Kolide development environment. Run the following:

```
kolide serve
```

If you're running the binary from the root of the repository, where it is built
by default, then the binary will automatically use the provided example
configuration file, which assumes that you are running docker locally, on
`localhost` via a native engine.

You may have to edit the example configuration file to use the output of 
`docker-machine ip` instead of `localhost` if you're using Docker via 
[Docker Toolbox](https://www.docker.com/products/docker-toolbox).