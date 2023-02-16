---
title: "Files in containers"
teaching: 15
exercises: 15
questions:
- "How do I make data available in a container?"
- "What data is made available by default in a container?"
objectives:
- "Understand that some data from the host system is usually made available by default within a container"
- "Learn more about how handles users and binds directories from the host filesystem."
keypoints:
- "Your current directory and home directory are usually available by default in a container."
- "You have the same username and permissions in a container as on the host system."
- "You can specify additional host system directories to be available in the container."
---

The way in which user accounts and access permissions are handled in {{ site.software.name }} containers is very different from that in Docker (where you effectively always have superuser/root access). When running a {{ site.software.name }} container, you only have the same permissions to access files as the user you are running as on the host system.

In this episode we'll look at working with files in the context of {{ site.software.name }} containers and how this links with {{ site.software.name }}'s approach to users and permissions within containers.

## Users within a {{ site.software.name }} container

The first thing to note is that when you ran `whoami` within the container shell you started at the end of the previous episode, you should have seen the username that you were signed in as on the host system when you ran the container.

For example, if my username were `jc1000`, I'd expect to see the following:

```
{{ site.machine.prompt }} {{ site.software.cmd }} shell lolcow_latest.sif
{{ site.software.prompt }} whoami
```
{: .language-bash}

```
jc1000
```
{: .output}

But hang on! I downloaded a version of the `lolcow_latest.sif` image from a public container repo. I haven't customised it in any way. How is it configured with my own user details?!

If you have any familiarity with Linux system administration, you may be aware that in Linux, users and their Unix groups are configured in the `/etc/passwd` and `/etc/group` files respectively. In order for the shell within the container to know of my user, the relevant user information needs to be available within these files within the container.

Assuming this feature is enabled within the installation of {{ site.software.name }} on your system, when the container is started, {{ site.software.name }}  appends the relevant user and group lines from the host system to the `/etc/passwd` and `/etc/group` files within the container [\[1\]](https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf).

This means that the host system can effectively ensure that you cannot access/modify/delete any data you should not be able to on the host system and you cannot run anything that you would not have permission to run on the host system since you are restricted to the same user permissions within the container as you are on the host system.

## Files and directories within a {{ site.software.name }} container

{{ site.software.name }}  also _binds_ some _directories_ from the host system where you are running the `{{ site.software.cmd }}` command into the container that you're starting. Note that this bind process is not copying files into the running container, it is making an existing directory on the host system visible and accessible within the container environment. If you write files to this directory within the running container, when the container shuts down, those changes will persist in the relevant location on the host system.

There is a default configuration of which files and directories are bound into the container but ultimate control of how things are set up on the system where you are running {{ site.software.name }} is determined by the system administrator. As a result, this section provides an overview but you may find that things are a little different on the system that you're running on.

One directory that is likely to be accessible within a container that you start is your _home directory_.  You may also find that the directory from which you issued the `{{ site.software.cmd }}` command (the _current working directory_) is also mapped.

The mapping of file content and directories from a host system into a {{ site.software.name }} container is illustrated in the example below showing a subset of the directories on the host Linux system and in a {{ site.software.name }}  container:

```
Host system:                                                      {{ site.software.name}}  container:
-------------                                                     ----------------------
/                                                                 /
├── bin                                                           ├── bin
├── etc                                                           ├── etc
│   ├── ...                                                       │   ├── ...
│   ├── group  ─> user's group added to group file in container ─>│   ├── group
│   └── passwd ──> user info added to passwd file in container ──>│   └── passwd
├── home                                                          ├── usr
│   └── jc1000 ───> user home directory made available ──> ─┐     ├── sbin
├── usr                 in container via bind mount         │     ├── home
├── sbin                                                    └────────>└── jc1000
└── ...                                                           └── ...

```
{: .output}


Now lets have a look at the permissions inside the containers root directory with the command

 ```
{{ site.software.prompt }} ls -l /
```
{: .language-bash}

```
total 8
lrwxrwxrwx   1 root    root       7 Jul 23  2021 bin -> usr/bin
drwxr-xr-x   2 root    root       3 Apr 15  2020 boot
drwxr-xr-x  23 root    root    9020 Dec  9 10:32 dev
lrwxrwxrwx   1 root    root      36 Feb 16 23:26 environment -> .singularity.d/env/90-environment.sh
drwxr-xr-x   1 cwal219 cwal219   60 Feb 16 23:35 etc
drwxr-xr-x   1 cwal219 cwal219   60 Feb 16 23:35 home
lrwxrwxrwx   1 root    root       7 Jul 23  2021 lib -> usr/lib
lrwxrwxrwx   1 root    root       9 Jul 23  2021 lib32 -> usr/lib32
lrwxrwxrwx   1 root    root       9 Jul 23  2021 lib64 -> usr/lib64
lrwxrwxrwx   1 root    root      10 Jul 23  2021 libx32 -> usr/libx32
drwxr-xr-x   2 root    root       3 Jul 23  2021 media
drwxr-xr-x   2 root    root       3 Jul 23  2021 mnt
drwxr-xr-x   2 root    root       3 Jul 23  2021 opt
dr-xr-xr-x 954 root    root       0 Nov 20 19:48 proc
drwx------   2 root    root      46 Jul 23  2021 root
drwxr-xr-x   5 root    root      67 Jul 23  2021 run
lrwxrwxrwx   1 root    root       8 Jul 23  2021 sbin -> usr/sbin
lrwxrwxrwx   1 root    root      24 Feb 16 23:26 singularity -> .singularity.d/runscript
drwxr-xr-x   2 root    root       3 Jul 23  2021 srv
dr-xr-xr-x  13 root    root       0 Nov 20 19:49 sys
drwxrwxrwt  28 root    root    4096 Feb 16 23:35 tmp
drwxr-xr-x  13 root    root     178 Jul 23  2021 usr
drwxr-xr-x  11 root    root     160 Jul 23  2021 var
```
{: .output}

This tells us quite a lot about how the container is operating.

> ## Files in {{ site.software.name }} containers
>
> 1. Try to create a file in the root directory, `touch /bin/somefile`. Is that what you expected would happen?
> 
> 2. In in your home directory, run the same command `touch ~/somefile`. Why does it work here? What happens to it when
> you exit the container?
>
> 3. Some of the files in the root directory are owned by you. Why might this be?
>
> 4. Why are we using `touch` to create files. What happens when you try to run `nano`?
>
> > ## Solution
> >
> > 1. We will have received the error `touch: cannot touch '/bin/somefile': Read-only file system`.
> > This tells us something else about the filesystem. It's not just that we don't have permission to delete the file, the filesystem
> > itself is read-only so even the `root` user wouldn't be able to edit/delete this file.
> >
> > 2. Within your home directory, you _should_ be able to successfully create a file. Since you're seeing your home directory on the host system which has been bound into the container, when you exit and the container shuts down, the file that you created within the container should still be present when you look at your home directory on the host system.
> >
> > 3. Elaborate on other default binds? `/etc/groups` etc?
> >
> > 4. If you try to run the command `nano` you will get the error `bash: nano: command not found`. This is because nano is not
> > installed in the container, the `touch` command however is a core util so will almost always be available.
> {: .solution}
{: .challenge}

## Binding additional host system directories to the container

You will sometimes need to bind additional host system directories into a container you are using over and above those bound by default. For example:

- There may be a shared dataset in a shard location that you need access to in the container
- You may require executables and software libraries in the container

The `-B` or `--bind` option to the `{{ site.software.cmd }}` command is used to specify additonal binds. Lets try binding the `{{ site.machine.working_dir }}/shared` directory.

```
{{ site.machine.prompt }} {{ site.software.cmd }} shell -B {{ site.machine.working_dir }}/shared lolcow_latest.sif
{{ site.software.prompt }} ls {{ site.machine.working_dir }}/shared
```
{: .language-bash}

```
some stuff in here
```
{: .output}

Note that, by default, a bind is mounted at the same path in the container as on the host system. You can also specify where a host directory is mounted in the container by separating the host path from the container path by a colon (`:`) in the option:

```
{{ site.machine.prompt }} {{ site.software.cmd }}  shell -B {{ site.machine.working_dir }}/shared:/nesi99991 lolcow_latest.sif
{{ site.software.prompt }} ls /nesi99991
```
{: .language-bash}

```
some stuff in here
```
{: .output}

If you need to mount multiple directories, you can either repeat the `-B` flag multiple times, or use a comma-separated list of paths, _i.e._

```
{{ site.machine.prompt }} {{ site.software.cmd }} -B dir1,dir2,dir3 ...
```
{: .language-bash}

Directories to be bind mounted can be also specified using the environment variable `{{ site.software.name | upcase }}_BINDPATH`:

```
{{ site.machine.prompt }} export {{ site.software.name | upcase }}_BINDPATH="dir1,dir2,dir3"
```
{: .language-bash}

> ## Mounting `$HOME`
>
> Depending on the site configuration of {{ site.software.name }}, user home directories might
> or might not be mounted into containers by default.  
> We do recommend that you **avoid mounting home** whenever possible, to avoid
> sharing potentially sensitive data, such as SSH keys, with the container, especially if exposing it to the public through a web service.
>
> If you need to share data inside the container home, you might just mount that specific file/directory, _e.g._
>
> ```
> -B $HOME/.local
> ```
> {: .language-bash}
>
> Or, if you want a full fledged home, you might define an alternative host directory to act as your container home, as in
>
> ```
> -B /path/to/fake/home:$HOME
> ```
> {: .language-bash}
>
> Finally, you should also **avoid running a container from your host home**,
otherwise this will be bind mounted as it is the current working directory.
{: .callout}

### How about sharing environment variables with the host?

By default, shell variables are inherited in the container from the host:

```
{{ site.machine.prompt }} export HELLO=world
{{ site.machine.prompt }} {{ site.software.cmd }} exec lolcow_latest.sif bash -c 'echo $HELLO'
```
{: .language-bash}

```
world
```
{: .output}

There might be situations where you want to isolate the shell environment of the container; to this end you can use the flag `-C`, or `--containall`:  
(Note that this will also isolate system directories such as `/tmp`, `/dev` and `/run`)

```
{{ site.machine.prompt }} export HELLO=world
{{ site.machine.prompt }} {{ site.software.cmd }} exec -C lolcow_latest.sif bash -c 'echo $HELLO'
```
{: .language-bash}

```
HELLO
```
{: .output}

If you need to pass only specific variables to the container, that might or might
not be defined in the host, you can define variables that start with `{{ site.software.name | upcase }}ENV_`;
this prefix will be automatically trimmed in the container:

```
{{ site.machine.prompt }} export {{ site.software.name | upcase }}ENV_CIAO=mondo
{{ site.machine.prompt }} {{ site.software.cmd }} exec -C lolcow_latest.sif bash -c 'echo $CIAO'
```
{: .language-bash}

```
mondo
```
{: .output}

<!-- An alternative way to define variables is to use the flag `--env`:

```
{{ site.machine.prompt }} {{ site.software.cmd }} exec --env CIAO=mondo lolcow_latest.sif bash -c 'echo $CIAO'
```
{: .language-bash}

```
mondo
```
{: .output} -->

> ## Consistency in your containers
>
> If your container is not behaving as expected, a good place to start is adding the `--containall` flag, as an unexpected
> environment variable or bind mount may be the cause.
{: .callout}