#
# Phase: Build
#

ARG MARVIN_VERSION
ARG REGISTRY_NAME

FROM golang:1.12.9-alpine3.10 AS builder

# Necessary dependencies
RUN echo "https://mirror.csclub.uwaterloo.ca/alpine/v3.10/main" >/etc/apk/repositories
RUN echo "https://mirror.csclub.uwaterloo.ca/alpine/v3.10/community" >>/etc/apk/repositories
RUN apk update

RUN apk add --no-cache --update python3-dev gcc build-base bash curl git musl openssh

### Application ###

RUN mkdir /app /code

# Copy the code
COPY . /code
WORKDIR /code

# Compile the binary
RUN GOOS=linux GOARCH=amd64 go build -tags static_all -o /app/marvin-python .

### Toolbox ###

RUN mkdir /toolbox

# Copy assets
COPY doc_coverage.py /toolbox/doc_coverage.py
COPY run_pylint.py /toolbox/run_pylint.py
COPY dependency.py /toolbox/dependency.py
COPY pylint_deepsource /app/pylint_deepsource
COPY processors /app/processors
COPY requirements /toolbox/requirements
COPY config_manager.py /toolbox/config_manager.py

#
# Phase: Analyzer build
#

FROM us.gcr.io/$REGISTRY_NAME/marvin:$MARVIN_VERSION

RUN apk update
RUN apk add bash curl git python py-pip py-virtualenv python3-dev gcc build-base musl-dev

#
# Copy the builds
COPY --from=builder /app /app
COPY --from=builder /toolbox/doc_coverage.py /toolbox/doc_coverage.py
COPY --from=builder /toolbox/dependency.py /toolbox/dependency.py
COPY --from=builder /toolbox/run_pylint.py /toolbox/run_pylint.py
COPY --from=builder /toolbox/config_manager.py /toolbox/config_manager.py
COPY --from=builder /toolbox/requirements /toolbox/requirements

#
# Virtualenv
#
RUN virtualenv -p python2 /toolbox/venv2
RUN virtualenv -p python3 /toolbox/venv3

# Phase: Analyzer
#

# Python2
RUN /toolbox/venv2/bin/pip install -r /toolbox/requirements/requirements2.txt

# Python3
RUN /toolbox/venv3/bin/pip install -r /toolbox/requirements/requirements3.txt
