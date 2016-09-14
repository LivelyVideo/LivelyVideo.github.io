# Upload

This is a simple NodeJS service to act as a backend service for browser-based
file uploads. It uses Restify. 

## Summary of Operation
This service accepts files sent by an upload client, either in chunked form 
or via the legacy multipart method. To use the service, the client is expected
to authenticate itself using the Authorization header (see curl examples below).
The service uses the Bearer token included in this header and calls a 
configured authorization service to verify the validity of the token. If valid,
the upload is allowed to proceed.

## Installation and Configuration
### Deployment Considerations
* Depending on your preference, this service can be made available as an 
installable tarfile, a npm package downloaded from LivelyVideo's repo, 
or a Docker container available from **TBD** repo.

#### Install via NPM
* If you are using the npm package, you will need to ensure that you are able
to connect to and use our repository: it is at `https://npm.livelyvideo.tv`.
You will need authentication credentials or a token along with other settings
that should be in the welcome email that you might have received earlier.
* Download the service: `npm install -g @livelyvideo/svcUpload`. If you are 
unable to install the module globally, you might need to follow different 
steps to run it locally.
* Create the configuration file as defined in the `Configuration` section
below.
* Run the service as `LIVELY_CFG=your_config_file.yaml svcUpload`.
* To use with a Node process manager such as `PM2` the following should work 
`pm2 start LIVELY_CFG=your_cfg_file.yaml svcUpload`.
* To download the project's dependencies, you will want to perform 
`npm install` in the project's working directory. To run unit tests, type
`npm test`.

### Dependencies
The service depends on a few Lively Video modules specified in the 
`package.json` file, which are published on https://npm.livelyvideo.tv. 
These include `@livelyvideo/config` and `@livelyvideo/db-mysql`.

### Configuration 
The application supports configuration specification by way of a YAML file
(default `cfg.yaml` in the local directory, or overridden by setting the 
environment var `LIVELY_CFG`) with individual settings overridden by environment
variables if desired. The settings and the corresponding override environment
variables in alphabetical order are as follows:
 
* `authEndpoint` (env var `AUTH_EP`): if defined, this allows the service to 
perform a GET on the specified url as follows: 
`{authEndPoint}/{BearerToken}`; expected response must be in JSON and should
contain a `token` and a `userId` field. If not defined, all calls to the service
will fail with `500/Internal Error` with the log hinting at this omission.
* `container` (env var `CONTAINER`): if defined, application assumes that it is
running in a container. If the value of this variable is `LC`, it assumes that
it is running as a container on localhost, which means that it has full access
to the configured DB server without any certificates. If value is `GC`, then it 
assumes that the DB instance is hosted by Google Cloud with access to the DB
managed by certs defined in `server-ca.pem`, `client-key.pem`, and 
`client-cert.pem` in directory `../mysql/`. NOTE: in a Kubernetes environment, 
the certs directory is mounted as a secrets volume to the container.
* `gcloudProj` (env var `GCLOUD_PROJECT`): define to be able to use Google
Cloud (GC) functionality such as Google Storage.
* `maxFileSize`: the maximum filesize that the end-user is allowed to upload.
* `sqlHost` (env var `SQL_HOST`): points to the address of the MariaDB/MySQL
service.
* `sqlPass` (env var `SQL_USER`): the DB user to use to connect to the DB
instance (default `root`).
* `sqlUser` (env var `SQL_PASS`): the DB password to use for the DB instance
(default `''`).
* `storeParm` (env var `STORE_PARM`): A parameter that is relevant for the 
`storeType` below. It could be the name of a directory or a GS bucket.
* `storeType` (env var `STORE_TYPE`): Valid values are `fs` for the filesystem
and `gs` for Google Storage (default `fs`).
* `svcPort` (env var `PORT`): The port on which the service will listen for
requests (default `3000`).
* `tmpfsDest` (env var `TMPFS_DST`): A temporary location on the local 
filesystem where the uploaded file is staged before being moved to the final
destination (default `/tmp`).
* `validExts`: A space separated list of extensions that are acceptable for the
input file. For example, if the value is `docx pdf`, then a file with name
`something.jpg` will be rejected by the server. Default value
is `gif jpg mp4 mpeg png`. 

## Running Locally
* Ensure that you have a local instance of MariaDB or MySQL installed with a 
user `root` with no password (or define the above environment variables to 
match your configuration).
* Run the scripts in the `config/dbscripts` directory on your DB instance in the
implied chronological order. For example, `mysql -u <username> -p < config/
dbscripts/db_20160714_svcupload_1.sql`.
* To use the mock authorization API embedded in this server, set the environment
variable `FAKE_AUTH_ENABLE` to 1. This service accepts almost any bearer token
sent to it and returns a valid response with a fixed `userID`. However, a bearer
token value of `evil` will cause it to return an Authorization error - this is 
to help you test the failed authentication case.
* Execute `node app.js` to run the service.

## End-points
* Server configuration: GET `/api/upload/config`
* Chunked: POST `/api/upload`
* Multi-part: POST `/api/upload/multipart`

## Server Response
* For successful requests, the service will return HTTP status code 200.
* When an upload completes successfully, the service will include a response header
`X-File-Retrieve-URL` that will contain a URL to be used to get the uploaded file.

## Example CURL Requests

### Chunked
* Discover file status to determine resume offset: `curl -ikL -XPOST -H 'Authorization: Bearer xxx' -H 'X-File-Name: something2.mpg'  http://localhost:3000/api/upload -d ''`
* Initiate upload and send chunks: `curl -ikL -XPOST -H 'Authorization: Bearer xxx' -H 'X-File-Name: something2.mpg'  http://localhost:3000/api/upload -d 'hello'`
* Last chunk: `curl -ikL -XPOST -H 'Authorization: Bearer xxx' -H 'X-Last: true' -H 'X-File-Name: something2.mpg'  http://localhost:3000/api/upload -d ' and bye'`

### Multi-part
* Request: `curl -ikL -XPOST -H 'Authorization: Bearer xxx' -H 'Content-Type: multipart/form-data' http://localhost:3000/api/upload/multipart -F "files=something2-mp.mp4" -F "fileBegin=@/tmp/something2"`

