---
layout: post
title: docker-compose and free CI on GitLab
date: 2019-07-15 08:29
categories: [docker, CI, docker-compose]
---


## CI needed

I was putting together a simple project based of Docker and Flask and wanted to run syntax check (flake8) and tests in continuous matter at some stage.

## Many CI choices

My first choice was Travis-CI as I have had a nice experience with it while working on cppzmq project but as I did not want to make public my code yet Travis was out of scope because of pricing.
Then I found this website that list a lot of CI services: https://github.com/ligurio/awesome-ci.
As my code was hosted on GitLab I decided to give a GitLab CI a try with their shared and free CI runners. GitLab offers 2,000 CI pipeline minutes per group per month on our shared runners. Really nice!

## Working solution

I needed docker-compose to run flake8 and tests.

After some back and forth I finally got working `.gitlab-ci.yml`:

```yaml
image: docker
services:
  - docker:dind

before_script:
    - apk add --no-cache py-pip python-dev libffi-dev openssl-dev gcc libc-dev make
    - pip install docker-compose
    - docker --version
    - docker-compose --version

build:
  stage: build
  script:
    - docker-compose down
    - docker-compose up -d --build
    - docker-compose exec -T users flake8 project
    - docker-compose exec -T users python manage.py test
```

## Failed iterations

Without option `-T` after `docker-compose exec` I was getting this error:

```
$ docker-compose exec users flake8 project
the input device is not a TTY
```

Tried to add `export COMPOSE_INTERACTIVE_NO_CLI=1` but that blew up with:


```
$ docker-compose exec users flake8 project
Traceback (most recent call last):
  File "/usr/bin/docker-compose", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python2.7/site-packages/compose/cli/main.py", line 71, in main
    command()
  File "/usr/lib/python2.7/site-packages/compose/cli/main.py", line 127, in perform_command
    handler(command, command_options)
  File "/usr/lib/python2.7/site-packages/compose/cli/main.py", line 535, in exec_command
    pty.start()
  File "/usr/lib/python2.7/site-packages/dockerpty/pty.py", line 338, in start
    io.set_blocking(pump, flag)
  File "/usr/lib/python2.7/site-packages/dockerpty/io.py", line 32, in set_blocking
    old_flag = fcntl.fcntl(fd, fcntl.F_GETFL)
ValueError: file descriptor cannot be a negative integer (-1)
```

## Further improvements

It would be nice to avoid installing docker-compose on every CI run...


