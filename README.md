# What is it

Simple BOSH release [made from scratch](http://bosh.io/docs/create-release.html) for educational purposes.
It consists of 2 jobs starting primitive Golang Web servers and one job running the infinite loop written in bash script.

The code of the `main_job` was packed into `main.tar.gz` blob for testing.

```
while :;
do
    echo "Press [CTRL+C] to stop..";
    sleep 1;
done
```

`golang_app`'s sources were taken from [CF CAT assets](https://github.com/cloudfoundry/cf-acceptance-tests/tree/master/assets/golang)

## Not suitable for production

Unlike this release, production-ready releases also have their blobs uploaded
and the blobstore configured (see [docs](http://bosh.io/docs/create-release.html#config-blobstore) for details).

# Deploy release

## Prerequisites

You have to deploy [BOSH Lite](https://github.com/cloudfoundry/bosh-lite) first.

## Deploy to BOSH Lite

```
git clone https://github.com/ldmberman/test-bosh-release.git test-release
cd test-release
./update  # update git submodules
bosh upload release ./dev-releases/test-release/test-release-0+dev.1.yml
bosh deployment manifest.yml
```

Replace `director_uuid` in the `manifest.yml` with your director's uuid (`bosh status --uuid`), then

```
bosh -n deploy
```

Ping `golang_job`s:

```
$ curl <job VM IP>:8000
> go, world
```

Get `main_job`'s logs:

```
$ bosh logs main_job
$ tar xzf main_job...tgz
$ cat main_job/main_job.stdout.log
Press [CTRL+C] to stop..
Press [CTRL+C] to stop..
Press [CTRL+C] to stop..
Press [CTRL+C] to stop..
Press [CTRL+C] to stop..
Press [CTRL+C] to stop..
..
Press [CTRL+C] to stop..
```

# Create new release

To create new release make some changes, then:

```
bosh create release --force
bosh upload ./dev-releases/test-release/test-release-0+dev.<new version>.yml
bosh -n deploy
```

## Debugging packaging scripts

There is a [great tool](https://github.com/mmb/bosh_job_docker) that may be used for debugging BOSH releases. It builds a docker image from a BOSH release archive. The archive can be created using `--with-tarball` option of a `bosh create` command.
