# Converting dist-git to source-git

This should be similar to [How to source-git?], except that it isn't :)

The main difference is that [How to source-git?] starts with the upstream
history and applies the patches from dist-git.

In this case though, the starting point is dist-git itself, so a few things
need to be done differently.

Here are the steps:

*Note:* things bellow were done on git.centos.org/rpms/rpm.

1. Fetch the sources from the lookaside cache.

   This will create some `tar.*` under `SOURCES`.

2. Unpack the `tar` to `src/rpm`.

3. Now applying the patches (from the `SOURCES` directory) can begin. In order
   to decide which patches to apply, look in the spec-file. Use
   [rebase-helper's `get_applied_patches()`] for this.

## Intermezzo: what should happen to all the patches?

Here is what we could do with all the patches:

- Patches which are used in the spec file and are applied to the sources will
  be deleted, a.k.a they wont show up in the source-git tree.

- Patches which are used in the spec file and cannot be applied - this is
  where tooling should err out, as something bad happened in dist-git.

- Patches which are not used in the spec file will end up being in the
  source-git tree under `centos-packaging/SOURCES/`. Maintainers might want to
  clean this up in the future. Or these might be other kinds of files stored
  for yet unknown reasons.

### For the future

Are there any expectations for how the history of a source-git repo will
evolve?

What should happen when sources in dist-git are updated? Currently we will
recreate the source-git branch.

What should happen when the spec file or the patches in dist-git are updated?


## Usage


### [pipenv](https://github.com/pypa/pipenv)
This project is using pipenv to manage dependencies.
```
$ dnf install pipenv
```

Create a new pipenv environment:
```
$ pipenv shell
```

Install all the dependencies:
```
$ pipenv install
```

You can use the tool now:
```
$ ./dist2src.py --help
Usage: dist2src.py [OPTIONS] COMMAND [ARGS]...

...
```

### via dnf

Alternatively to pipenv, you can get the dependencies from Fedora via dnf:
```
$ dnf install python3-{sh,click,GitPython,packit}
```


## The Process

When creating a source-git commit from dist-git, the process will be the
following:

1. Take the content of the lookaside cache from a dist-git commit.

2. Apply the patches from the same commit (more or less the way it's described
   above).

3. Create a source-git commit from whatever the above results in.

Simply put:

    $ cd git.centos.org
    $ dist2src convert rpms/rpm:c8s src/rpm:c8s

Or breaking it down:

    $ cd git.centos.org
    $ dist2src checkout rpms/rpm c8s
    $ dist2src checkout --orphan src/rpm c8s
    $ dist2src get-archive rpms/rpm
    $ dist2src extract-archive rpms/rpm src/rpm
    $ dist2src copy-spec rpms/rpm src/rpm
    $ dist2src add-packit-config src/rpm
    $ dist2src copy-patches rpms/rpm src/rpm
    $ dist2src apply-patches src/rpm

`dist2src get-archive` calls [`get_sources.sh`] or the script specified in
`DIST2SRC_GET_SOURCES`.

[How to source-git?]: https://packit.dev/docs/how-to-source-git
[`get_sources.sh`]: https://wiki.centos.org/Sources#get_sources.sh_script
[rebase-helper's `get_applied_patches()`]: https://github.com/rebase-helper/rebase-helper/blob/e98f4f6b14e2ca2e8cbb8a8fbeb6935e5d0cf289/rebasehelper/specfile.py#L351
