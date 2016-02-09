# SmartContainers Client API

SmartContainers uses the Docker docker-py module [API](https://github.com/docker/docker-py/blob/master/docs/api.md) specification. This document defines how SmartContainers extends the docker-py client API for the capture of provenance information and additional methods that allow other types of metadata to be stored in the SmartContainer. For full documentation of the docker-py client, see the original [API](https://github.com/docker/docker-py/blob/master/docs/api.md). Only methods extended by the SmartContainers Client API are documented here as well as some additional functions for access to data and metadata. 

To instantiate a SmartContainer `Client` class that will allow you to communicate with a
Docker daemon, simply do:

```python
>>> from docker import Client
>>> cli = Client(base_url='unix://var/run/docker.sock')
```

**Params**:

* base_url (str): Refers to the protocol+hostname+port where the Docker server
is hosted.
* version (str): The version of the API the client will use. Specify `'auto'`
  to use the API version provided by the server.
* timeout (int): The HTTP request timeout, in seconds.
* tls (bool or [TLSConfig](tls.md#TLSConfig)): Equivalent CLI options: `docker --tls ...`

****

The Original API Modified by SmartContainer:

## build

Similar to the `docker build` command. Either `path` or `fileobj` needs to be
set. `path` can be a local path (to a directory containing a Dockerfile) or a
remote URL. `fileobj` must be a readable file-like object to a Dockerfile.

If you have a tar file for the Docker build context (including a Dockerfile)
already, pass a readable file-like object to `fileobj` and also pass
`custom_context=True`. If the stream is compressed also, set `encoding` to the
correct value (e.g `gzip`).

**Params**:

* path (str): Path to the directory containing the Dockerfile
* tag (str): A tag to add to the final image
* quiet (bool): Whether to return the status
* fileobj: A file object to use as the Dockerfile. (Or a file-like object)
* nocache (bool): Don't use the cache when set to `True`
* rm (bool): Remove intermediate containers. The `docker build` command now
  defaults to ``--rm=true``, but we have kept the old default of `False`
  to preserve backward compatibility
* stream (bool): *Deprecated for API version > 1.8 (always True)*.
  Return a blocking generator you can iterate over to retrieve build output as
  it happens
* timeout (int): HTTP timeout
* custom_context (bool): Optional if using `fileobj`
* encoding (str): The encoding for a stream. Set to `gzip` for compressing
* pull (bool): Downloads any updates to the FROM image in Dockerfiles
* forcerm (bool): Always remove intermediate containers, even after unsuccessful builds
* dockerfile (str): path within the build context to the Dockerfile
* container_limits (dict): A dictionary of limits applied to each container
  created by the build process. Valid keys:
    - memory (int): set memory limit for build
    - memswap (int): Total memory (memory + swap), -1 to disable swap
    - cpushares (int): CPU shares (relative weight)
    - cpusetcpus (str): CPUs in which to allow execution, e.g., `"0-3"`, `"0,1"`
* decode (bool): If set to `True`, the returned stream will be decoded into
  dicts on the fly. Default `False`.

**Returns** (generator): A generator for the build output

```python
>>> from io import BytesIO
>>> from docker import Client
>>> dockerfile = '''
... # Shared Volume
... FROM busybox:buildroot-2014.02
... MAINTAINER first last, first.last@yourdomain.com
... VOLUME /data
... CMD ["/bin/sh"]
... '''
>>> f = BytesIO(dockerfile.encode('utf-8'))
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> response = [line for line in cli.build(
...     fileobj=f, rm=True, tag='yourname/volume'
... )]
>>> response
['{"stream":" ---\\u003e a9eb17255234\\n"}',
'{"stream":"Step 1 : MAINTAINER first last, first.last@yourdomain.com\\n"}',
'{"stream":" ---\\u003e Running in 08787d0ee8b1\\n"}',
'{"stream":" ---\\u003e 23e5e66a4494\\n"}',
'{"stream":"Removing intermediate container 08787d0ee8b1\\n"}',
'{"stream":"Step 2 : VOLUME /data\\n"}',
'{"stream":" ---\\u003e Running in abdc1e6896c6\\n"}',
'{"stream":" ---\\u003e 713bca62012e\\n"}',
'{"stream":"Removing intermediate container abdc1e6896c6\\n"}',
'{"stream":"Step 3 : CMD [\\"/bin/sh\\"]\\n"}',
'{"stream":" ---\\u003e Running in dba30f2a1a7e\\n"}',
'{"stream":" ---\\u003e 032b8b2855fc\\n"}',
'{"stream":"Removing intermediate container dba30f2a1a7e\\n"}',
'{"stream":"Successfully built 032b8b2855fc\\n"}']
```

**Raises:** [TypeError](
https://docs.python.org/3.4/library/exceptions.html#TypeError) if `path` nor
`fileobj` are specified

## commit

Identical to the `docker commit` command.

**Params**:

* container (str): The image hash of the container
* repository (str): The repository to push the image to
* tag (str): The tag to push
* message (str): A commit message
* author (str): The name of the author
* conf (dict): The configuration for the container. See the [Docker remote api](
https://docs.docker.com/reference/api/docker_remote_api/) for full details.

## copy
Identical to the `docker cp` command. Get files/folders from the container.
**Deprecated for API version >= 1.20** &ndash; Consider using
[`get_archive`](#get_archive) **instead.**

**Params**:

* container (str): The container to copy from
* resource (str): The path within the container

**Returns** (str): The contents of the file as a string

## create_container

Creates a container that can then be `.start()` ed. Parameters are similar to
those for the `docker run` command except it doesn't support the attach
options (`-a`).

See [Port bindings](port-bindings.md) and [Using volumes](volumes.md) for more
information on how to create port bindings and volume mappings.

The `mem_limit` variable accepts float values (which represent the memory limit
of the created container in bytes) or a string with a units identification char
('100000b', '1000k', '128m', '1g'). If a string is specified without a units
character, bytes are assumed as an intended unit.

`volumes_from` and `dns` arguments raise [TypeError](
https://docs.python.org/3.4/library/exceptions.html#TypeError) exception if
they are used against v1.10 and above of the Docker remote API. Those
arguments should be passed as part of the `host_config` dictionary.

**Params**:

* image (str): The image to run
* command (str or list): The command to be run in the container
* hostname (str): Optional hostname for the container
* user (str or int): Username or UID
* detach (bool): Detached mode: run container in the background and print new
container Id
* stdin_open (bool): Keep STDIN open even if not attached
* tty (bool): Allocate a pseudo-TTY
* mem_limit (float or str): Memory limit (format: [number][optional unit],
where unit = b, k, m, or g)
* ports (list of ints): A list of port numbers
* environment (dict or list): A dictionary or a list of strings in the
following format `["PASSWORD=xxx"]` or `{"PASSWORD": "xxx"}`.
* dns (list): DNS name servers
* volumes (str or list):
* volumes_from (str or list): List of container names or Ids to get volumes
from. Optionally a single string joining container id's with commas
* network_disabled (bool): Disable networking
* name (str): A name for the container
* entrypoint (str or list): An entrypoint
* cpu_shares (int): CPU shares (relative weight)
* working_dir (str): Path to the working directory
* domainname (str or list): Set custom DNS search domains
* memswap_limit (int):
* host_config (dict): A [HostConfig](hostconfig.md) dictionary
* mac_address (str): The Mac Address to assign the container
* labels (dict or list): A dictionary of name-value labels (e.g. `{"label1": "value1", "label2": "value2"}`) or a list of names of labels to set with empty values (e.g. `["label1", "label2"]`)
* volume_driver (str): The name of a volume driver/plugin.
* stop_signal (str): The stop signal to use to stop the container (e.g. `SIGINT`).

**Returns** (dict): A dictionary with an image 'Id' key and a 'Warnings' key.

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> container = cli.create_container(image='busybox:latest', command='/bin/sleep 30')
>>> print(container)
{'Id': '8a61192da2b3bb2d922875585e29b74ec0dc4e0117fcbf84c962204e97564cd7',
 'Warnings': None}
```

### docker.utils.parse_env_file

A utility for parsing an environment file.

The expected format of the file is as follows:

```
USERNAME=jdoe
PASSWORD=secret
```

The utility can be used as follows:

```python
>>> import docker.utils
>>> my_envs = docker.utils.parse_env_file('/path/to/file')
>>> docker.utils.create_container_config('1.18', '_mongodb', 'foobar',  environment=my_envs)
```

You can now use this with 'environment' for `create_container`.


## create_volume

Create and register a named volume

**Params**:

* name (str): Name of the volume
* driver (str): Name of the driver used to create the volume
* driver_opts (dict): Driver options as a key-value dictionary

**Returns** (dict): The created volume reference object

```python
>>> from docker import Client
>>> cli = Client()
>>> volume = cli.create_volume(
  name='foobar', driver='local', driver_opts={'foo': 'bar', 'baz': 'false'}
)
>>> print(volume)
{u'Mountpoint': u'/var/lib/docker/volumes/foobar/_data', u'Driver': u'local', u'Name': u'foobar'}
```

## exec_create

Sets up an exec instance in a running container.

**Params**:

* container (str): Target container where exec instance will be created
* cmd (str or list): Command to be executed
* stdout (bool): Attach to stdout of the exec command if true. Default: True
* stderr (bool): Attach to stderr of the exec command if true. Default: True
* since (UTC datetime or int): Output logs from this timestamp. Default: `None` (all logs are given)
* tty (bool): Allocate a pseudo-TTY. Default: False
* user (str): User to execute command as. Default: root

**Returns** (dict): A dictionary with an exec 'Id' key.


## exec_start

Start a previously set up exec instance.

**Params**:

* exec_id (str): ID of the exec instance
* detach (bool): If true, detach from the exec command. Default: False
* tty (bool): Allocate a pseudo-TTY. Default: False
* stream (bool): Stream response data. Default: False

**Returns** (generator or str): If `stream=True`, a generator yielding response
chunks. A string containing response data otherwise.

## export

Export the contents of a filesystem as a tar archive to STDOUT.

**Params**:

* container (str): The container to export

**Returns** (str): The filesystem tar archive as a str

## get_archive

Retrieve a file or folder from a container in the form of a tar archive.

**Params**:

* container (str): The container where the file is located
* path (str): Path to the file or folder to retrieve

**Returns** (tuple): First element is a raw tar data stream. Second element is
a dict containing `stat` information on the specified `path`.

```python
>>> import docker
>>> cli = docker.Client()
>>> ctnr = cli.create_container('busybox', 'true')
>>> strm, stat = cli.get_archive(ctnr, '/bin/sh')
>>> print(stat)
{u'linkTarget': u'', u'mode': 493, u'mtime': u'2015-09-16T12:34:23-07:00', u'name': u'sh', u'size': 962860}
```

## get_image

Get an image from the docker daemon. Similar to the `docker save` command.

**Params**:

* image (str): Image name to get

**Returns** (urllib3.response.HTTPResponse object): The response from the docker daemon

An example of how to get (save) an image to a file.
```python
>>> from docker import Client
>>> cli = Client(base_url='unix://var/run/docker.sock')
>>> image = cli.get_image(“fedora:latest”)
>>> image_tar = open(‘/tmp/fedora-latest.tar’,’w’)
>>> image_tar.write(image.data)
>>> image_tar.close()
```

## import_image

Similar to the `docker import` command.

If `src` is a string or unicode string, it will first be treated as a path to
a tarball on the local system. If there is an error reading from that file,
src will be treated as a URL instead to fetch the image from. You can also pass
an open file handle as 'src', in which case the data will be read from that
file.

If `src` is unset but `image` is set, the `image` parameter will be taken as
the name of an existing image to import from.

**Params**:

* src (str or file): Path to tarfile, URL, or file-like object
* repository (str): The repository to create
* tag (str): The tag to apply
* image (str): Use another image like the `FROM` Dockerfile parameter

## import_image_from_data

Like `.import_image()`, but allows importing in-memory bytes data.

**Params**:

* data (bytes collection): Bytes collection containing valid tar data
* repository (str): The repository to create
* tag (str): The tag to apply

## import_image_from_file

Like `.import_image()`, but only supports importing from a tar file on
disk. If the file doesn't exist it will raise `IOError`.

**Params**:

* filename (str): Full path to a tar file.
* repository (str): The repository to create
* tag (str): The tag to apply

## import_image_from_url

Like `.import_image()`, but only supports importing from a URL.

**Params**:

* url (str): A URL pointing to a tar file.
* repository (str): The repository to create
* tag (str): The tag to apply

## import_image_from_image

Like `.import_image()`, but only supports importing from another image,
like the `FROM` Dockerfile parameter.

**Params**:

* image (str): Image name to import from
* repository (str): The repository to create
* tag (str): The tag to apply

## kill

Kill a container or send a signal to a container.

**Params**:

* container (str): The container to kill
* signal (str or int): The signal to send. Defaults to `SIGKILL`

## load_image

Load an image that was previously saved using `Client.get_image`
(or `docker save`). Similar to `docker load`.

**Params**:

* data (binary): Image data to be loaded

## login

Nearly identical to the `docker login` command, but non-interactive.

**Params**:

* username (str): The registry username
* password (str): The plaintext password
* email (str): The email for the registry account
* registry (str): URL to the registry. Ex:`https://index.docker.io/v1/`
* reauth (bool): Whether refresh existing authentication on the docker server.
* dockercfg_path (str): Use a custom path for the .dockercfg file
  (default `$HOME/.dockercfg`)

**Returns** (dict): The response from the login request

## pull

Identical to the `docker pull` command.

**Params**:

* repository (str): The repository to pull
* tag (str): The tag to pull
* stream (bool): Stream the output as a generator
* insecure_registry (bool): Use an insecure registry
* auth_config (dict): Override the credentials that Client.login has set for this request
  `auth_config` should contain the `username` and `password` keys to be valid.

**Returns** (generator or str): The output

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> for line in cli.pull('busybox', stream=True):
...     print(json.dumps(json.loads(line), indent=4))
{
    "status": "Pulling image (latest) from busybox",
    "progressDetail": {},
    "id": "e72ac664f4f0"
}
{
    "status": "Pulling image (latest) from busybox, endpoint: ...",
    "progressDetail": {},
    "id": "e72ac664f4f0"
}
```

## push

Push an image or a repository to the registry. Identical to the `docker push`
command.

**Params**:

* repository (str): The repository to push to
* tag (str): An optional tag to push
* stream (bool): Stream the output as a blocking generator
* insecure_registry (bool): Use `http://` to connect to the registry

**Returns** (generator or str): The output of the upload

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> response = [line for line in cli.push('yourname/app', stream=True)]
>>> response
['{"status":"Pushing repository yourname/app (1 tags)"}\\n',
 '{"status":"Pushing","progressDetail":{},"id":"511136ea3c5a"}\\n',
 '{"status":"Image already pushed, skipping","progressDetail":{},
    "id":"511136ea3c5a"}\\n',
 ...
 '{"status":"Pushing tag for rev [918af568e6e5] on {
    https://cdn-registry-1.docker.io/v1/repositories/
    yourname/app/tags/latest}"}\\n']
```

## put_archive

Insert a file or folder in an existing container using a tar archive as source.

**Params**:

* container (str): The container where the file(s) will be extracted
* path (str): Path inside the container where the file(s) will be extracted.
  Must exist.
* data (bytes): tar data to be extracted

**Returns** (bool): True if the call succeeds. `docker.errors.APIError` will
be raised if an error occurs.

## remove_container

Remove a container. Similar to the `docker rm` command.

**Params**:

* container (str): The container to remove
* v (bool): Remove the volumes associated with the container
* link (bool): Remove the specified link and not the underlying container
* force (bool): Force the removal of a running container (uses SIGKILL)

## remove_image

Remove an image. Similar to the `docker rmi` command.

**Params**:

* image (str): The image to remove
* force (bool): Force removal of the image
* noprune (bool): Do not delete untagged parents

## remove_volume

Remove a volume. Similar to the `docker volume rm` command.

**Params**:

* name (str): The volume's name

Failure to remove will raise a `docker.errors.APIError` exception.

## rename

Rename a container. Similar to the `docker rename` command.

**Params**:

* container (str): ID of the container to rename
* name (str): New name for the container

## restart

Restart a container. Similar to the `docker restart` command.

If `container` a dict, the `Id` key is used.

**Params**:

* container (str or dict): The container to restart
* timeout (int): Number of seconds to try to stop for before killing the
container. Once killed it will then be restarted. Default is 10 seconds.

## search
Identical to the `docker search` command.

**Params**:

* term (str): A term to search for

**Returns** (list of dicts): The response of the search

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> response = cli.search('nginx')
>>> response[:2]
[{'description': 'Official build of Nginx.',
  'is_official': True,
  'is_trusted': False,
  'name': 'nginx',
  'star_count': 266},
 {'description': 'Trusted automated Nginx (http://nginx.org/) ...',
  'is_official': False,
  'is_trusted': True,
  'name': 'dockerfile/nginx',
  'star_count': 60},
  ...
```

## start

Similar to the `docker start` command, but doesn't support attach options. Use
`.logs()` to recover `stdout`/`stderr`.

**Params**:

* container (str): The container to start

**Deprecation warning:** For API version > 1.15, it is highly recommended to
  provide host config options in the
  [`host_config` parameter of `create_container`](#create_container)

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> container = cli.create_container(
...     image='busybox:latest',
...     command='/bin/sleep 30')
>>> response = cli.start(container=container.get('Id'))
>>> print(response)
None
```

## stop

Stops a container. Similar to the `docker stop` command.

**Params**:

* container (str): The container to stop
* timeout (int): Timeout in seconds to wait for the container to stop before
sending a `SIGKILL`

## tag

Tag an image into a repository. Identical to the `docker tag` command.

**Params**:

* image (str): The image to tag
* repository (str): The repository to set for the tag
* tag (str): The tag name
* force (bool): Force

**Returns** (bool): True if successful

****
## New API functions defined by SmartContainers.

## get_entry


Returns http or directory entry point inside container as a json-ld document using the hydra core vocabulary.

## sc_version

Nearly identical to the `docker version` command.

**Returns** (dict): The server version information

```python
>>> from docker import Client
>>> cli = Client(base_url='tcp://127.0.0.1:2375')
>>> cli.version()
{
    "KernelVersion": "3.16.4-tinycore64",
    "Arch": "amd64",
    "ApiVersion": "1.15",
    "Version": "1.3.0",
    "GitCommit": "c78088f",
    "Os": "linux",
    "GoVersion": "go1.3.3"
}
```

## get_label
Get json_ld string as a docker label to an docker image.

## put_label
Put json_ld string to a docker conta

## create_sc
Infect container with SmartContainer API and Linked Data Platform. In short, make a "Smart Container"

## put_config
Send configuration file to SmartContainers for use in a operation. Normally, the configuration is stored in the .sc directory, but for cloud operations the file needs to be provided. The Configuration stores persistent data such as ORCID id, Prov:Agent information.

## get_config
Retrieve current configuration file.
