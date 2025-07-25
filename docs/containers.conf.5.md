% containers.conf 5 Container engine configuration file

# NAME
containers.conf - The container engine configuration file specifies default
configuration options and command-line flags for container engines.

# DESCRIPTION
Container engines like Podman & Buildah read containers.conf file, if it exists
and modify the defaults for running containers on the host. containers.conf uses
a TOML format that can be easily modified and versioned.

Container engines read the __/usr/share/containers/containers.conf__,
__/etc/containers/containers.conf__, and __/etc/containers/containers.conf.d/\*.conf__
for global configuration that effects all users.
For user specific configuration it reads __\$XDG_CONFIG_HOME/containers/containers.conf__ and
__\$XDG_CONFIG_HOME/containers/containers.conf.d/\*.conf__ files. When `$XDG_CONFIG_HOME` is not set it falls back to using `$HOME/.config` instead.

Fields specified in containers conf override the default options, as well as
options in previously read containers.conf files.

Config files in the `.d` directories, are added in alpha numeric sorted order and must end in `.conf`.

Not all options are supported in all container engines.

Note, container engines also use other configuration files for configuring the environment.

* `storage.conf` for configuration of container and images storage.
* `registries.conf` for definition of container registries to search while pulling.
container images.
* `policy.conf` for controlling which images can be pulled to the system.

Note: If Podman is running in a virtual machine using `podman machine` (this
includes Mac and Windows hosts), ensure that the configuration files are edited in the
virtual machine by using `podman machine ssh`.

## ENVIRONMENT VARIABLES
If the `CONTAINERS_CONF` environment variable is set, all system and user
config files are ignored and only the specified config file will be loaded.

If the `CONTAINERS_CONF_OVERRIDE` path environment variable is set, the config
file will be loaded last even when `CONTAINERS_CONF` is set.

The values of both environment variables may be absolute or relative paths, for
instance, `CONTAINERS_CONF=/tmp/my_containers.conf`.

## MODULES
A module is a containers.conf file located directly in or a sub-directory of the following three directories:
 - __\$XDG_CONFIG_HOME/containers/containers.conf.modules__ or  __\$HOME/.config/containers/containers.conf.modules__ if `$XDG_CONFIG_HOME` is not set.
 - __/etc/containers/containers.conf.modules__
 - __/usr/share/containers/containers.conf.modules__

Files in those locations are not loaded by default but only on-demand.  They are loaded after all system and user configuration files but before `CONTAINERS_CONF_OVERRIDE` hence allowing for overriding system and user configs.

Modules are currently supported by podman(1).  The `podman --module` flag allows for loading a module and can be specified multiple times.  If the specified value is an absolute path, the config file will be loaded directly.  Relative paths are resolved relative to the three module directories mentioned above and in the specified order such that modules in `$XDG_CONFIG_HOME/$HOME` allow for overriding those in `/etc` and `/usr/share`.

## APPENDING TO STRING ARRAYS

The default behavior during the loading sequence of multiple containers.conf files is to override previous data.  To change the behavior from overriding to appending, you can set the `append` attribute as follows: `array=["item-1", "item=2", ..., {append=true}]`.  Setting the append attribute instructs to append to this specific string array for the current and also subsequent loading steps.  To change back to overriding, set `{append=false}`.

Consider the following example:
```
modules1.conf: env=["1=true"]
modules2.conf: env=["2=true"]
modules3.conf: env=["3=true", {append=true}]
modules4.conf: env=["4=true"]
```

After loading the files in the given order, the final contents are `env=["2=true", "3=true", "4=true"]`.  If modules4.conf would set `{append=false}`, the final contents would be `env=["4=true"]`.

# FORMAT
The [TOML format][toml] is used as the encoding of the configuration file.
Every option is nested under its table. No bare options are used. The format of
TOML can be simplified to:

    [table1]
    option = value

    [table2]
    option = value

    [table3]
    option = value

    [table3.subtable1]
    option = value

## CONTAINERS TABLE
The containers table contains settings to configure and manage the OCI runtime.

**annotations** = []

List of annotations. Specified as "key=value" pairs to be added to all containers.

Example: "run.oci.keep_original_groups=1"

**apparmor_profile**="container-default"

Used to change the name of the default AppArmor profile of container engines.
The default profile name is "container-default".

**base_hosts_file**=""

Base file to create the `/etc/hosts` file inside the container. This must either
be an absolute path to a file on the host system, or one of the following
special flags:
  ""      Use the host's `/etc/hosts` file (the default)
  `none`  Do not use a base file (i.e. start with an empty file)
  `image` Use the container image's `/etc/hosts` file as base file

**cgroup_conf**=[]

List of cgroup_conf entries specifying a list of cgroup files to write to and
their values. For example `memory.high=1073741824` sets the
memory.high limit to 1GB.

**cgroups**="enabled"

Determines  whether  the  container will create CGroups.
Options are:
  `enabled`   Enable cgroup support within container
  `disabled`  Disable cgroup support, will inherit cgroups from parent
  `no-conmon` Do not create a cgroup dedicated to conmon.

**cgroupns**="private"

Default way to create a cgroup namespace for the container.
Options are:
`private` Create private Cgroup Namespace for the container.
`host`    Share host Cgroup Namespace with the container.

**container_name_as_hostname**=true|false

When no hostname is set for a container, use the container's name, with
characters not valid for a hostname removed, as the hostname instead of
the first 12 characters of the container's ID. Containers not running
in a private UTS namespace will have their hostname set to the host's
hostname regardless of this setting.

Default is false.

**default_capabilities**=[]

List of default capabilities for containers.

The default list is:
```
default_capabilities = [
      "CHOWN",
      "DAC_OVERRIDE",
      "FOWNER",
      "FSETID",
      "KILL",
      "NET_BIND_SERVICE",
      "SETFCAP",
      "SETGID",
      "SETPCAP",
      "SETUID",
      "SYS_CHROOT",
]
```

Note, by default container engines using containers.conf, run with less
capabilities than Docker. Docker runs additionally with "AUDIT_WRITE", "MKNOD" and "NET_RAW". If you need to add one of these capabilities for a
particular container, you can use the --cap-add option or edit your system's containers.conf.

**default_sysctls**=[]

A list of sysctls to be set in containers by default,
specified as "name=value".

Example:"net.ipv4.ping_group_range=0 1000".

**default_ulimits**=[]

A list of ulimits to be set in containers by default,
specified as "name=soft-limit:hard-limit".

Example: "nofile=1024:2048".

**devices**=[]

List of devices.
Specified as 'device-on-host:device-on-container:permissions'.

Example: "/dev/sdc:/dev/xvdc:rwm".

**dns_options**=[]

List of default DNS options to be added to /etc/resolv.conf inside of the
container.

**dns_searches**=[]

List of default DNS search domains to be added to /etc/resolv.conf inside of
the container.

**dns_servers**=[]

A list of dns servers to override the DNS configuration passed to the
container. The special value “none” can be specified to disable creation of
/etc/resolv.conf in the container.

**env**=["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"]

Environment variable list for the container process, used for passing
environment variables to the container. If a variable is listed without a value,
the value is copied from the host environment.

Note that this is only used when a container is created, not with subsequent
commands like `podman exec`. This prevents variables in the config file from
overwriting values specified on the command line when the container was created.

**env_host**=false

Pass all host environment variables into the container.

**host_containers_internal_ip**=""

Set the IP address the container should expect to connect to the host. The IP
address is used by Podman to automatically add the `host.containers.internal`
and `host.docker.internal` hostnames to the container's `/etc/hosts` file. It
is also used for the *host-gateway* flag of Podman's `--add-host` CLI option.
If no IP address is configured (the default), Podman will try to determine it
automatically, but might fail to do so depending on the container's network
setup. Adding these internal hostnames to `/etc/hosts` is silently skipped then.
Set this config to `none` to never add the internal hostnames to `/etc/hosts`.

Note: If Podman is running in a virtual machine using `podman machine` (this
includes Mac and Windows hosts), Podman resolves the `host.containers.internal`
hostname via the podman machine (gvproxy) DNS resolver instead when it is empty.
Also because the name will be resolved by the DNS name in gvproxy setting this
to `none` has no effect. This option does not change the gvproxy behavior.

Note: This config doesn't affect the actual network setup, it just tells Podman
the IP address it should expect. Configuring an IP address here doesn't ensure
that the container can actually reach the host using this IP address.

**http_proxy**=true

Default proxy environment variables will be passed into the container.
The environment variables passed in include:
`http_proxy`, `https_proxy`, `ftp_proxy`, `no_proxy`, and the upper case
versions of these. The `no_proxy` option is needed when host system uses a proxy
but container should not use proxy. Proxy environment variables specified for
the container in any other way will override the values passed from the host.

**init**=false

Run an init inside the container that forwards signals and reaps processes.

**init_path**="/usr/libexec/podman/catatonit"

If this option is not set catatonit is searched in the directories listed under
the **helper_binaries_dir** option. It is recommended to just install catatonit
there instead of configuring this option here.

Path to the container-init binary, which forwards signals and reaps processes
within containers. Note that the container-init binary will only be used when
the `--init` for podman-create and podman-run is set.

**interface_name**=""

Default way to set interface names inside containers. Defaults to legacy pattern
of ethX, where X is an integer, when left undefined.
Options are:
  `device`   Uses the network_interface name from the network config as interface name. Falls back to the ethX pattern if the network_interface is not set.

**ipcns**="shareable"

Default way to create a IPC namespace for the container.
Options are:
  `host`     Share host IPC Namespace with the container.
  `none`     Create shareable IPC Namespace for the container without a private /dev/shm.
  `private`  Create private IPC Namespace for the container, other containers are not allowed to share it.
  `shareable` Create shareable IPC Namespace for the container.

**keyring**=true

Indicates whether the container engines create a kernel keyring for use within
the container.

**label**=true

Indicates whether the container engine uses MAC(SELinux) container separation via labeling. This option is ignored on disabled systems.

**label_users**=false

label_users indicates whether to enforce confined users in containers on
SELinux systems. This option causes containers to maintain the current user
and role field of the calling process. By default SELinux containers run with
the user system_u, and the role system_r.

**log_driver**=""

Logging driver for the container. Currently available options are k8s-file, journald, none and passthrough, with json-file aliased to k8s-file for scripting compatibility.  The journald driver is used by default if the systemd journal is readable and writable.  Otherwise, the k8s-file driver is used.

**log_size_max**=-1

Maximum size allowed for the container's log file. Negative numbers indicate
that no size limit is imposed. If it is positive, it must be >= 8192 to
match/exceed conmon's read buffer. The file is truncated and re-opened so the
limit is never exceeded.

**log_tag**=""

Default format tag for container log messages. This is useful for creating a specific tag for container log messages. Container log messages default to using the truncated container ID as a tag.

**mounts**=[]

List of mounts.
Specified as "type=TYPE,source=<directory-on-host>,destination=<directory-in-container>,<options>"

Example:  [ "type=bind,source=/var/lib/foobar,destination=/var/lib/foobar,ro", ]

**netns**=""

Default way to create a NET namespace for the container.
The option is mapped to the **--network** argument for the podman commands, it accepts the same values as that option.
For example it can be set to `bridge`, `host`, `none`, `pasta` and more, see the [podman-create(1)](https://docs.podman.io/en/latest/markdown/podman-create.1.html#network-mode-net)
manual for all available options.

**no_hosts**=false

Do not modify the `/etc/hosts` file in the container. Podman assumes control
over the container's `/etc/hosts` file by default; refer to the `--add-host`
CLI option for details. To disable this, either set this config to `true`, or
use the functionally identical `--no-hosts` CLI option.

**oom_score_adj**=0

Tune the host's OOM preferences for containers (accepts values from -1000 to 1000).

**pidns**="private"

Default way to create a PID namespace for the container.
Options are:
  `private` Create private PID Namespace for the container.
  `host`    Share host PID Namespace with the container.

**pids_limit**=1024

Maximum number of processes allowed in a container. 0 indicates that no limit
is imposed.

**prepare_volume_on_create**=false

Copy the content from the underlying image into the newly created volume when the container is created instead of when it is started. If `false`, the container engine will not copy the content until the container is started. Setting it to `true` may have negative performance implications.

**privileged**=false

Give extended privileges to all containers. A privileged container turns off the security features that isolate the container from the host. Dropped Capabilities, limited devices, read-only mount points, Apparmor/SELinux separation, and Seccomp filters are all disabled. Due to the disabled security features, the privileged field should almost never be set as containers can easily break out of confinment.

Containers running in a user namespace (e.g., rootless containers) cannot have more privileges than the user that launched them.

**read_only**=true|false

Run all containers with root file system mounted read-only. Set to false by default.

**seccomp_profile**="/usr/share/containers/seccomp.json"

Path to the seccomp.json profile which is used as the default seccomp profile
for the runtime.

**shm_size**="65536k"

Size of `/dev/shm`. The format is `<number><unit>`. `number` must be greater
than `0`.
Unit is optional and can be:
`b` (bytes), `k` (kilobytes), `m`(megabytes), or `g` (gigabytes).
If you omit the unit, the system uses bytes. If you omit the size entirely,
the system uses `65536k`.

**tz=**""

Set timezone in container. Takes IANA timezones as well as `local`, which sets the timezone in the container to match the host machine.
If not set, then containers will run with the time zone specified in the image.

Examples:
  `tz="local"`
  `tz="America/New_York"`

**umask**="0022"

Sets umask inside the container.

**userns**="host"

Default way to create a USER namespace for the container.
Options are:
  `private` Create private USER Namespace for the container.
  `host`    Share host USER Namespace with the container.

**utsns**="private"

Default way to create a UTS namespace for the container.
Options are:
  `private` Create private UTS Namespace for the container.
  `host`    Share host UTS Namespace with the container.

**volumes**=[]

List of volumes.
Specified as "directory-on-host:directory-in-container:options".

Example:  "/db:/var/lib/db:ro".

## NETWORK TABLE
The `network` table contains settings pertaining to the management of CNI
plugins.

**network_backend**=""

Network backend determines what network driver will be used to set up and tear down container networks.
Valid values are "cni" and "netavark".
The default value is empty which means that it will automatically choose CNI or netavark. If there are
already containers/images or CNI networks preset it will choose CNI.

Before changing this value all containers must be stopped otherwise it is likely that
iptables rules and network interfaces might leak on the host. A reboot will fix this.

**cni_plugin_dirs**=[]

List of paths to directories where CNI plugin binaries are located.

The default list is:
```
cni_plugin_dirs = [
  "/usr/local/libexec/cni",
  "/usr/libexec/cni",
  "/usr/local/lib/cni",
  "/usr/lib/cni",
  "/opt/cni/bin",
]
```

**netavark_plugin_dirs**=[]

List of directories that will be searched for netavark plugins.

The default list is:
```
netavark_plugin_dirs = [
  "/usr/local/libexec/netavark",
  "/usr/libexec/netavark",
  "/usr/local/lib/netavark",
  "/usr/lib/netavark",
]
```

**default_network**="podman"

The name of the default network as seen in `podman network ls`. This option only effects the network assignment when
the bridge network mode is selected, i.e. `--network bridge`. It is the default for rootful containers but not as
rootless. To change the default network mode use the **netns** option under the `[containers]` table.

Note: This should not be changed while you have any containers using this network.

**default_subnet**="10.88.0.0/16"

The subnet to use for the default network (named above in **default_network**).

Note: This should not be changed if any containers are currently running on the default network.

**default_subnet_pools**=[]

DefaultSubnetPools is a list of subnets and size which are used to
allocate subnets automatically for podman network create.
It will iterate through the list and will pick the first free subnet
with the given size. This is only used for ipv4 subnets, ipv6 subnets
are always assigned randomly.

The default list is (10.89.0.0-10.255.255.0/24):
```
default_subnet_pools = [
  {"base" = "10.89.0.0/16", "size" = 24},
  {"base" = "10.90.0.0/15", "size" = 24},
  {"base" = "10.92.0.0/14", "size" = 24},
  {"base" = "10.96.0.0/11", "size" = 24},
  {"base" = "10.128.0.0/9", "size" = 24},
]
```

**default_rootless_network_cmd**="pasta"

Configure which rootless network program to use by default. Valid options are
`slirp4netns` and `pasta` (default).

**network_config_dir**="/etc/cni/net.d/"

Path to the directory where network configuration files are located.
For the CNI backend the default is __/etc/cni/net.d__ as root
and __$HOME/.config/cni/net.d__ as rootless.
For the netavark backend "/etc/containers/networks" is used as root
and "$graphroot/networks" as rootless.

**firewall_driver**=""

The firewall driver to be used by netavark.
The default is empty which means netavark will pick one accordingly. Current supported
drivers are "iptables", "nftables", "none" (no firewall rules will be created) and "firewalld" (firewalld is
experimental at the moment and not recommend outside of testing).

**dns_bind_port**=53

Port to use for dns forwarding daemon with netavark in rootful bridge
mode and dns enabled.
Using an alternate port might be useful if other dns services should
run on the machine.

**pasta_options** = []

A list of default pasta options that should be used running pasta.
It accepts the pasta cli options, see pasta(1) for the full list of options.

## ENGINE TABLE
The `engine` table contains configuration options used to set up container engines such as Podman and Buildah.

**active_service**=""

Name of destination for accessing the Podman service. See SERVICE DESTINATION TABLE below.

**add_compression**=[]

List of compression algorithms. If set makes sure that requested compression variant
for each platform is added to the manifest list keeping original instance intact in
the same manifest list on every `manifest push`. Supported values are (`gzip`, `zstd` and `zstd:chunked`).
`zstd:chunked` is incompatible with encrypting images, and will be treated as `zstd` with a warning
in that case.


Note: This is different from `compression_format` which allows users to select a default
compression format for `push` and `manifest push`, while `add_compression` is limited to
`manifest push` and allows users to append new instances to manifest list with specified compression
algorithms in `add_compression` for each platform.

**cgroup_manager**="systemd"

The cgroup management implementation used for the runtime. Supports `cgroupfs`
and `systemd`.

**compat_api_enforce_docker_hub**=true

Enforce using docker.io for completing short names in Podman's compatibility
REST API. Note that this will ignore unqualified-search-registries and
short-name aliases defined in containers-registries.conf(5).

**compose_providers**=[]

Specify one or more external providers for the compose command.  The first
found provider is used for execution.  Can be an absolute and relative path or
a (file) name.

**compose_warning_logs**=true

Emit logs on each invocation of the compose command indicating that an external
compose provider is being executed.

**conmon_env_vars**=[]

Environment variables to pass into Conmon.

**conmon_path**=[]

Paths to search for the conmon container manager binary. If the paths are
empty or no valid path was found, then the `$PATH` environment variable will be
used as the fallback.

The default list is:
```
conmon_path=[
    "/usr/libexec/podman/conmon",
    "/usr/local/libexec/podman/conmon",
    "/usr/local/lib/podman/conmon",
    "/usr/bin/conmon",
    "/usr/sbin/conmon",
    "/usr/local/bin/conmon",
    "/usr/local/sbin/conmon",
    "/run/current-system/sw/bin/conmon",
]
```

**database_backend**=""

The database backend of Podman.  Supported values are "" (default), "boltdb"
and "sqlite". An empty value means it will check whenever a boltdb already
exists and use it when it does, otherwise it will use sqlite as default
(e.g. new installs). This allows for backwards compatibility with older versions.
Please run `podman-system-reset` prior to changing the database
backend of an existing deployment, to make sure Podman can operate correctly.

**detach_keys**="ctrl-p,ctrl-q"

Keys sequence used for detaching a container.
Specify the keys sequence used to detach a container.
Format is a single character `[a-Z]` or a comma separated sequence of
`ctrl-<value>`, where `<value>` is one of:
`a-z`, `@`, `^`, `[`, `\`, `]`, `^` or `_`
Specifying "" disables this feature.

**enable_port_reservation**=true

Determines whether the engine will reserve ports on the host when they are
forwarded to containers. When enabled, when ports are forwarded to containers,
they are held open by conmon as long as the container is running, ensuring that
they cannot be reused by other programs on the host. However, this can cause
significant memory usage if a container has many ports forwarded to it.
Disabling this can save memory.

**env**=[]

Environment variables to be used when running the container engine (e.g., Podman, Buildah). For example "http_proxy=internal.proxy.company.com".
Note these environment variables will not be used within the container. Set the env section under [containers] table,
if you want to set environment variables for the container.

**events_logfile_path**=""

Define where event logs will be stored, when events_logger is "file".

**events_logfile_max_size**="1m"

Sets the maximum size for events_logfile_path.
The unit can be b (bytes), k (kilobytes), m (megabytes) or g (gigabytes).
The format for the size is `<number><unit>`, e.g., `1b` or `3g`.
If no unit is included then the size will be in bytes.
When the limit is exceeded, the logfile will be rotated and the old one will be deleted.
If the maximum size is set to 0, then no limit will be applied,
and the logfile will not be rotated.

**events_logger**="journald"

The default method to use when logging events.

The default method is different based on the platform that
Podman is being run upon.  To determine the current value,
use this command:

`podman info --format {{.Host.EventLogger}}`

Valid values are: `file`, `journald`, and `none`.

**events_container_create_inspect_data**=true|false

Creates a more verbose container-create event which includes a JSON payload
with detailed information about the container.  Set to false by default.

**healthcheck_events**=true|false

Whenever Podman should log healthcheck events.
With many running healthcheck on short interval Podman will spam the event
log a lot as it generates a event for each single healthcheck run. Because
this event is optional and only useful to external consumers that may want
to know when a healthcheck is run or failed allow users to turn it off by
setting it to false.

Default is true.

**helper_binaries_dir**=["/usr/libexec/podman", ...]

A is a list of directories which are used to search for helper binaries.
The following binaries are searched in these directories:
 - aardvark-dns
 - catatonit
 - netavark
 - pasta
 - slirp4netns

Podman machine uses it for these binaries:
 - gvproxy
 - qemu
 - vfkit

The default paths on Linux are:

- `/usr/local/libexec/podman`
- `/usr/local/lib/podman`
- `/usr/libexec/podman`
- `/usr/lib/podman`

The default paths on macOS are:

- `/usr/local/opt/podman/libexec`
-	`/opt/homebrew/bin`
-	`/opt/homebrew/opt/podman/libexec`
- `/usr/local/bin`
-	`/usr/local/libexec/podman`
-	`/usr/local/lib/podman`
-	`/usr/libexec/podman`
-	`/usr/lib/podman`

The default path on Windows is:

- `C:\Program Files\RedHat\Podman`

**hooks_dir**=["/etc/containers/oci/hooks.d", ...]

Path to the OCI hooks directories for automatically executed hooks.

**cdi_spec_dirs**=["/etc/cdi", "/var/run/cdi", ...]

Directories to scan for CDI Spec files.

**image_default_format**="oci"|"v2s2"|"v2s1"

Manifest Type (oci, v2s2, or v2s1) to use when pulling, pushing, building
container images. By default images pulled and pushed match the format of the
source image. Building/committing defaults to OCI.
Note: **image_build_format** is deprecated.

**image_default_transport**="docker://"

Default transport method for pulling and pushing images.

**image_parallel_copies**=0

Maximum number of image layers to be copied (pulled/pushed) simultaneously.
Not setting this field will fall back to containers/image defaults. (6)

**image_volume_mode**="bind"

Tells container engines how to handle the built-in image volumes.

* bind: An anonymous named volume will be  created  and  mounted into the container.
* tmpfs: The volume is mounted onto the container as a tmpfs, which allows the users to create content that disappears when the container is stopped.
* ignore: All volumes are just ignored and no action is taken.

**infra_command**="/pause"

Infra (pause) container image command for pod infra containers. When running a
pod, we start a `/pause` process in a container to hold open the namespaces
associated with the pod. This container does nothing other than sleep,
reserving the pod's resources for the lifetime of the pod.

**infra_image**=""

Infra (pause) container image for pod infra containers. When running a
pod, we start a `pause` process in a container to hold open the namespaces
associated with the pod. This container does nothing other than sleep,
reserving the pod's resources for the lifetime of the pod. By default container
engines run a built-in container using the pause executable. If you want override
specify an image to pull.

**kube_generate_type**="pod"

Default Kubernetes kind/specification of the kubernetes yaml generated with the `podman kube generate` command. The possible options are `pod` and `deployment`.

**lock_type**="shm"

Specify the locking mechanism to use; valid values are "shm" and "file".
Change the default only if you are sure of what you are doing, in general
"file" is useful only on platforms where cgo is not available for using the
faster "shm" lock type. You may need to run "podman system renumber" after you
change the lock type.

**multi_image_archive**=false

Allows for creating archives (e.g., tarballs) with more than one image. Some container engines, such as Podman, interpret additional arguments as tags for one image and hence do not store more than one image. The default behavior can be altered with this option.

**namespace**=""

Default engine namespace. If the engine is joined to a namespace, it will see
only containers and pods that were created in the same namespace, and will
create new containers and pods in that namespace. The default namespace is "",
which corresponds to no namespace. When no namespace is set, all containers
and pods are visible.

**network_cmd_path**=""

Path to the slirp4netns binary.

**network_cmd_options**=[]

Default options to pass to the slirp4netns binary.

Valid options values are:

  - **allow_host_loopback=true|false**: Allow the slirp4netns to reach the host loopback IP (`10.0.2.2`). Default is false.
  - **mtu=MTU**: Specify the MTU to use for this network. (Default is `65520`).
  - **cidr=CIDR**: Specify ip range to use for this network. (Default is `10.0.2.0/24`).
  - **enable_ipv6=true|false**: Enable IPv6. Default is true. (Required for `outbound_addr6`).
  - **outbound_addr=INTERFACE**: Specify the outbound interface slirp should bind to (ipv4 traffic only).
  - **outbound_addr=IPv4**: Specify the outbound ipv4 address slirp should bind to.
  - **outbound_addr6=INTERFACE**: Specify the outbound interface slirp should bind to (ipv6 traffic only).
  - **outbound_addr6=IPv6**: Specify the outbound ipv6 address slirp should bind to.
  - **port_handler=rootlesskit**: Use rootlesskit for port forwarding. Default.
  Note: Rootlesskit changes the source IP address of incoming packets to a IP address in the container network namespace, usually `10.0.2.100`. If your application requires the real source IP address, e.g. web server logs, use the slirp4netns port handler. The rootlesskit port handler is also used for rootless containers when connected to user-defined networks.
  - **port_handler=slirp4netns**: Use the slirp4netns port forwarding, it is slower than rootlesskit but preserves the correct source IP address. This port handler cannot be used for user-defined networks.

**no_pivot_root**=false

Whether to use chroot instead of pivot_root in the runtime.

**num_locks**=2048

Number of locks available for containers, pods, and volumes.
Each created container, pod, or volume consumes one lock.
Locks are recycled and can be reused after the associated container, pod, or volume is removed.
The default number available is 2048.
If this is changed, a lock renumbering must be performed, using the `podman system renumber` command.

**pod_exit_policy**="continue"

Set the exit policy of the pod when the last container exits.  Supported policies are:

| Exit Policy        | Description                                                                 |
| ------------------ | --------------------------------------------------------------------------- |
| *continue*         | The pod continues running when the last container exits. Used by default.   |
| *stop*             | The pod is stopped when the last container exits. Used in `play kube`.      |

**pull_policy**="always"|"missing"|"never"

Pull image before running or creating a container. The default is **missing**.

- **missing**: attempt to pull the latest image from the registries listed in registries.conf if a local image does not exist. Raise an error if the image is not in any listed registry and is not present locally.
- **always**: pull the image from the first registry it is found in as listed in registries.conf. Raise an error if not found in the registries, even if the image is present locally.
- **never**: do not pull the image from the registry, use only the local version. Raise an error if the image is not present locally.

**remote** = false

Indicates whether the application should be running in remote mode. This flag modifies the
--remote option on container engines. Setting the flag to true will default `podman --remote=true` for access to the remote Podman service.

**retry** = 3

Number of times to retry pulling/pushing images in case of failure.

**retry_delay** = ""

Delay between retries in case pulling/pushing image fails. If set, container engines will retry at the set interval, otherwise they delay 2 seconds and then exponentially back off.

**runtime**=""

Default OCI specific runtime in runtimes that will be used by default. Must
refer to a member of the runtimes table. Default runtime will be searched for
on the system using the priority: "crun", "runc", "runj", "kata", "runsc", "ocijail"

**runtime_supports_json**=["crun", "crun-vm", "runc", "kata", "runsc", "youki", "krun"]

The list of the OCI runtimes that support `--format=json`.

**runtime_supports_kvm**=["kata", "krun"]

The list of OCI runtimes that support running containers with KVM separation.

**runtime_supports_nocgroups**=["crun", "crun-vm", "krun"]

The list of OCI runtimes that support running containers without CGroups.

**image_copy_tmp_dir**="/var/tmp"

Default location for storing temporary container image content. Can be
overridden with the TMPDIR environment variable. If you specify "storage", then
the location of the container/storage tmp directory will be used. If set then it
is the users responsibility to cleanup storage. Configure tmpfiles.d(5) to
cleanup storage.

**service_timeout**=**5**

Number of seconds to wait without a connection  before the
`podman system service` times out and exits

**static_dir**="/var/lib/containers/storage/libpod"

Directory for persistent libpod files (database, etc).
By default this will be configured relative to where containers/storage
stores containers.

**stop_timeout**=10

Number of seconds to wait for container to exit before sending kill signal.

**exit_command_delay**=300

Number of seconds to wait for the API process for the exec call before sending exit command mimicking the Docker behavior of 5 minutes (in seconds).

**tmp_dir**="/run/libpod"

The path to a temporary directory to store per-boot container.
Must be a tmpfs (wiped after reboot).

**volume_path**="/var/lib/containers/storage/volumes"

Directory where named volumes will be created in using the default volume
driver.
By default this will be configured relative to where containers/storage store
containers. This convention is followed by the default volume driver, but may
not be by other drivers.

**chown_copied_files**=true

Determines whether file copied into a container will have changed ownership to
the primary uid/gid of the container.

**compression_format**="gzip"

Specifies the compression format to use when pushing an image. Supported values
are: `gzip`, `zstd` and `zstd:chunked`. This field is ignored when pushing
images to the docker-daemon and docker-archive formats. It is also ignored
when the manifest format is set to v2s2.
`zstd:chunked` is incompatible with encrypting images, and will be treated as `zstd` with a warning
in that case.

**compression_level**="5"

The compression level to use when pushing an image. Valid options
depend on the compression format used. For gzip, valid options are
1-9, with a default of 5. For zstd, valid options are 1-20, with a
default of 3.

## SERVICE DESTINATION TABLE
The `engine.service_destinations` table contains configuration options used to set up remote connections to the podman service for the podman API.

**[engine.service_destinations.{name}]**
URI to access the Podman service
**uri="ssh://user@production.example.com/run/user/1001/podman/podman.sock"**

  Example URIs:

- **rootless local**  - unix:///run/user/1000/podman/podman.sock
- **rootless remote** - ssh://user@engineering.lab.company.com/run/user/1000/podman/podman.sock
- **rootful local**  - unix:///run/podman/podman.sock
- **rootful remote** - ssh://root@10.10.1.136:22/run/podman/podman.sock

**identity="~/.ssh/id_rsa**

Path to file containing ssh identity key

**[engine.volume_plugins]**

A table of all the enabled volume plugins on the system. Volume plugins can be
used as the backend for Podman named volumes. Individual plugins are specified
below, as a map of the plugin name (what the plugin will be called) to its path
(filepath of the plugin's unix socket).

**[engine.platform_to_oci_runtime]**

Allows end users to switch the OCI runtime on the bases of container image's platform string.
Following config field contains a map of `platform/string = oci_runtime`.

## SECRET TABLE
The `secret` table contains settings for the configuration of the secret subsystem.

**driver**=file

Name of the secret driver to be used.
Currently valid values are:
  * file
  * pass

**[secrets.opts]**

The driver specific options object.

## MACHINE TABLE
The `machine` table contains configurations for podman machine VMs

**cpus**=1
Number of CPU's a machine is created with.

**disk_size**=10

The size of the disk in GB created when init-ing a podman-machine VM

**image**=""

Image used when creating a new VM using `podman machine init`.
Can be specified as a registry with a bootable OCI artifact, download URL, or a local path.
Registry target must be in the form of `docker://registry/repo/image:version`.
Container engines translate URIs $OS and $ARCH to the native OS and ARCH.
URI "https://example.com/$OS/$ARCH/foobar.ami" would become
"https://example.com/linux/amd64/foobar.ami" on a Linux AMD machine.
If unspecified, the default Podman machine image will be used.

**memory**=2048

Memory in MB a machine is created with.

**user**=""

Username to use and create on the podman machine OS for rootless container
access. The default value is `user`. On Linux/Mac the default is`core`.

**volumes**=["$HOME:$HOME"]

Host directories to be mounted as volumes into the VM by default.
Environment variables like $HOME as well as complete paths are supported for
the source and destination. An optional third field `:ro` can be used to
tell the container engines to mount the volume readonly.

On Mac, the default volumes are:

   [ "/Users:/Users", "/private:/private", "/var/folders:/var/folders" ]

**provider**=""

Virtualization provider to be used for running a podman-machine VM. Empty value
is interpreted as the default provider for the current host OS.

| Platform | Default Virtualization provider         | Optional |
| -------- | --------------------------------------- | -------- |
| Linux    | "" (qemu)                               | None     |
| Windows  | "" ("wsl": Windows Subsystem for Linux) | "hyperv" (Windows Server Virtualization) |
| Mac      | "" ("applehv": Apple Hypervisor)        | "libkrun" (Launch machine via libkrun platform, optimized for sharing GPU with the machine) |


**rosetta**="true"

Rosetta supports running x86_64 Linux binaries on a Podman machine on Apple silicon.
The default value is `true`. Supported on AppleHV(arm64) machines only.

## FARMS TABLE
The `farms` table contains configuration options used to group up remote connections into farms that will be used when sending out builds to different machines in a farm via `podman buildfarm`.

**default**=""

The default farm to use when farming out builds.

**[farms.list]**

Map of farms created where the key is the farm name and the value is the list of system connections.

## PODMANSH TABLE
The `podmansh` table contains configuration options used by podmansh.

**shell**="/bin/sh"

The shell to spawn in the container.
The default value is `/bin/sh`.

**container**="podmansh"

Name of the container that podmansh joins.
The default value is `podmansh`.

**timeout**=0

Number of seconds to wait for podmansh logins. This value if favoured over the deprecated field `engine.podmansh_timeout` if set.
The default value is 30.


# FILES

**containers.conf**

Distributions often provide a __/usr/share/containers/containers.conf__ file to
provide a default configuration. Administrators can override fields in this
file by creating __/etc/containers/containers.conf__ to specify their own
configuration. They may also drop `.conf` files in
__/etc/containers/containers.conf.d__ which will be loaded in alphanumeric order.
For user specific configuration it reads __\$XDG_CONFIG_HOME/containers/containers.conf__ and
__\$XDG_CONFIG_HOME/containers/containers.conf.d/\*.conf__ files. When `$XDG_CONFIG_HOME` is not set it falls back to using `$HOME/.config` instead.

Fields specified in a containers.conf file override the default options, as
well as options in previously loaded containers.conf files.

**storage.conf**

The `/etc/containers/storage.conf` file is the default storage configuration file.
Rootless users can override fields in the storage config by creating
__$HOME/.config/containers/storage.conf__.

If the `CONTAINERS_STORAGE_CONF` path environment variable is set, this path
is used for the storage.conf file rather than the default.
This is primarily used for testing.

# SEE ALSO
containers-storage.conf(5), containers-policy.json(5), containers-registries.conf(5), tmpfiles.d(5)

[toml]: https://github.com/toml-lang/toml
