# logpaste

[![CircleCI](https://circleci.com/gh/mtlynch/logpaste.svg?style=svg)](https://circleci.com/gh/mtlynch/logpaste)
[![Docker Pulls](https://img.shields.io/docker/pulls/mtlynch/logpaste.svg?maxAge=604800)](https://hub.docker.com/r/mtlynch/logpaste/)
[![License](http://img.shields.io/:license-mit-blue.svg?style=flat-square)](LICENSE)

A minimalist web service for uploading and sharing log files.

## Features

* Accept text uploads from command-line, JavaScript, and web UI
* Simple to deploy
  * Runs in a single Docker container
  * Fits in the free tier of Heroku
* Easy database management
  * Replicates to and restores datastore to S3-compatible interfaces
* Customizable UI without changing source code

## Demo

* <http://logpaste.com>

## Run LogPaste

### From source

```bash
go run main.go
```

### From Docker

This is the simplest way to run LogPaste, but you will lose all data when you shut down the container.

```bash
docker run \
  -p 3001:3001/tcp \
  --name logpaste \
  mtlynch/logpaste
```

### From Docker + persistent data

To run LogPaste with persistent data, mount a volume from your local system to store the LogPaste sqlite database.

```bash
docker run \
  -p 3001:3001/tcp \
  --volume "${PWD}/data:/app/data" \
  --name logpaste \
  mtlynch/logpaste
```

### From Docker + cloud data replication

If you specify settings for an S3 bucket, LogPaste will use [Litestream](https://litestream.io/) to automatically replicate your data to S3.

You can kill the container and start it later, and it will restore your data from the S3 bucket and continue as if there was no no interruption.

```bash
AWS_ACCESS_KEY_ID=YOUR-ACCESS-ID
AWS_SECRET_ACCESS_KEY=YOUR-SECRET-ACCESS-KEY
AWS_REGION=YOUR-REGION
DB_REPLICA_URL=s3://your-bucket-name/db

docker run \
  -e "AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}" \
  -e "AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}" \
  -e "AWS_REGION=${AWS_REGION}" \
  -e "DB_REPLICA_URL=${DB_REPLICA_URL}" \
  -e "CREATE_NEW_DB='true'" `# change to false after first run` \
  -p 3001:3001/tcp \
  --name logpaste \
  mtlynch/logpaste
```

Some notes:

* After you run your container for the first time, remove the `CREATE_NEW_DB` line.
* Only run one Docker container for each S3 location
  * LogPaste can't sync writes across multiple instances.

### With custom site settings

LogPaste offers some options to customize the text for your site. Here's an example that uses a custom title and subtitle:

```bash
SITE_TITLE="My Cool Log Pasting Service"
SITE_SUBTITLE="Upload all your logs for FooBar here"
SITE_SHOW_DOCUMENTATION="false" # Hide usage information from homepage

docker run \
  -e "SITE_TITLE=${SITE_TITLE}" \
  -e "SITE_SUBTITLE=${SITE_SUBTITLE}" \
  -e "SITE_SHOW_DOCUMENTATION=${SITE_SHOW_DOCUMENTATION}" \
  -p 3001:3001/tcp \
  --name logpaste \
  mtlynch/logpaste
```

## Deployment

LogPaste is easy to deploy to cloud services. Here are some places it works well:

* [Heroku](docs/deployment/heroku.md) (recommended)
* [Amazon LightSail](docs/deployment/lightsail.md)

## Parameters

### Command-line flags

| Flag | Meaning | Default Value |
|------|---------|---------------|
| `-title` | Title to display on homepage | `"LogPaste"` |
| `-subtitle` | Subtitle to display on homepage | `"A minimalist, open-source debug log upload service"` |
| `-showDocs` | Whether to display usage documentation on homepage | `true` |

### Docker environment variables

You can adjust behavior of the Docker container by passing these parameters with `docker run -e`:

| Environment Variable | Meaning | Default Value |
|----------------------|---------|---------------|
| `SITE_TITLE`         | Value to set the `-title` command-line flag | |
| `SITE_SUBTITLE`      | Value to set the `-subtitle`  command-line flag | |
| `SITE_SHOW_DOCUMENTATION` | Value to set the `-showDocs` command-line flag | |
| `DB_REPLICA_URL`     | S3 URL where you want to replicate the LogPaste datastore (e.g., `s3://mybucket.mydomain.com/db`) | |
| `AWS_REGION`         | AWS region where your S3 bucket is located | |
| `AWS_ACCESS_KEY_ID`  | AWS access key ID for an IAM role with access to the bucket where you want to replicate data. |  |
| `AWS_SECRET_ACCESS_KEY` | AWS secret access key for an IAM role with access to the bucket where you want to replicate data. | |

### Docker build args

If you rebuild the Docker image from source, you can adjust the build behavior with `docker build --build-arg`:

| Build Arg | Meaning | Default Value |
| --------- | ------- | ------------- |
| `litestream_version` | Version of [Litestream](https://litestream.io/) to use for data replication | `0.3.3` |
