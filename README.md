# debain-build-repo                                                                                                                                                                                                                         

This is a shell script that can build a functional Debian archive
repository out of a directory of Debian package files.

## Usage

```
$ KEY_ID=<...> debian-build-repo path/to/pkgs/ path/to/repo/
```

Where `pkgs/` is a directory of packages in the form
`<dist>/<component>/package.db`. A very simple example:

```
$ tree pkgs/
pkgs/
└── stable
    └── main
        ├── loki-canary_1.5.0-0_amd64.deb
        ├── loki-logs_1.5.0-0_amd64.deb
        └── promtail_1.5.0-0_amd64.deb

2 directories, 3 files
```

Packages files can be in any subpath under the component directory;
this portion of the path will be ignored except for specifying the
path to the package file. The key thing is that packages have to be
grouped into directories of named dists, and within each dist
directory packages must be grouped into directories of named
components.

The repository will be generated in the specified repository
directory, with the package directory symlinked as `<repo dir>/pool`
and the signing key exported as `Key.gpg`. The intent is that you can
just point a server at the generated repository directory and have
everything clients need to pull packages.

**WARNING: the script will delete all contents of the repo
directory**. The idea is to re-generate the entire repository whenever
necessary, so the repo dir is removed and recreated when the script is
run. If this is a problem, consider generating the repo in a secondary
location and moving/copying it to the final location This is a smart
thing to do if you can spare the disk space -- it will minimize or
prevent the file server from serving a partially-complete repository
while it's being generated.

## Environment variables

The script's operation can be altered by setting the following
environment variables:

- `KEY_ID`: the ID of the key you will use to sign the release. This
  must be on your gpg keychain. This must be set when running the
  script; you'll get an error if it isn't.
- `WORK_DIR`: the directory in which the script store temporary
  data. If you specify this, the script will not delete it
  afterwards. Useful for debugging.
- `SKIP_SIGN`: set this to any non-empty value in order to skip
  signing the release information. Repositories must be signed, but if
  you need to run this a lot the gpg key prompts can get
  irritating. Also mostly useful for debugging.

## Q & A

### Will this package support compressed index files?

Probably not (try turning compression on in your webserver
instead). Doing that requires you to do funny things like include
hashes for both the compressed and uncompressed files, even if you're
not serving the uncompressed ones. Storing an uncompressed and
compressed version takes up even more space than just storing the
uncompressed indexes. In short, it's possible, but it's a headache,
and has dubious benefits.

### How about source packages?

Nope. Maybe if I ever figure out what their deal is.

### by-hash index retrieval?

Uh-uh.

### ... index differencing/history....?

Buddy, this is a _shell script_.

### How about just translations, then?

Maybe, if it were clear how to do that. The Debian repository wiki
seems to indicate localized descriptions have to belong to a separate
locale-specific package, in which case it's not clear where the
translated description could be stored that wasn't in the package
itself.

### I want to put a package in more than one folder... er... dist... er... component?

Hardlinks, my dude. Hardlinks. Or symlinks, really. The thing is, the
script needs the directory structure to know how you, the operator,
would like packages to be grouped, but how that's accomplished doesn't
really matter. Do note that if you _do_ hardlink package files, they
will not be hardlinked in the output repo: package files are simply
copied to their output destination.
