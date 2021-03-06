quietemerge:
============

This program is used to perform the emerge in Gentoo Linux with pretty output.
Link to [Google code page](https://code.google.com/p/quietemerge/) and
[Gentoo Forums page](http://forums.gentoo.org/viewtopic-t-797019-highlight-.html)

The help:
---------

  * Usage:     `quietemerge [<options>]`
  * Options:
    ```
    -h | --help     Show this help text
    ```

    All other options are passed directly to emerge.
    If no arguments are provided to quietemerge, then this help text is shown.

  * If any arguments that are unsupported by this script is detected
    then the script will execute emerge directly, bypassing all the setup
    in the rest of the script.
    A few examples of such unsupported arguments are:
    `--changelog`, `--depclean`, `--search`, `--sync`, `--unmerge`, etc.
    See the function `has_unsupported_arg` of quietemerge for examples of
    more unsupported arguments.

  * Examples on how to call this script:
    ```
    quietemerge -1 glibc vim
    quietemerge --resume
    quietemerge -auDvN --jobs=3 --load-average=4 --keep-going world
    ```
  * You can set the environment variable `_ETA_ALL=1` to get the estimated
    time of emerge for each individual package. On running as root, you can
    simply set `ETA_ALL=1` in the config file.

    Note: This script creates a config file `/root/.config/quietemerge.config`
    only when run as root.

  * In order to temporarily override the settings present in the config
    file, prepend the variable with an underscore `_` and provide it as an
    environment variable.
    An example where I want to override the config file setting
    temporarily, and/or provide a temporary setting:
    ```
    _DELAY=2 _MOUNT_TMPFS=1 quietemerge -a eselect
    ```


Config variables:
-----------------

  * `DELAY`: Delay after which genlop will be called (in a loop). Default
    5 seconds.

  * `MOUNT_TMPFS`: Should we mount tmpfs on to /var/tmp/portage? This should
    be used only if you have enough RAM. Default 0 (no).

  * `TMPFS_SIZE`: Amount of RAM that will be mounted on to /var/tmp/portage
    if MOUNT_TMPFS is 1. Default 50%.

  * `TXT_UPDATER`: The console based program that is used to update /etc,/usr
    config files. Default empty.

  * `GUI_UPDATER`: The graphical program that is used to update /etc,/usr
    config files. Default `TXT_UPDATER`.

  * `SHOW_MERGE_TIME`: Whether we should display the latest merge time
    instead of "Done!" after  a package has been emerged. Default is
    0 (no).

  * `PRE_CMD`: A command which will be run just before the actual emerge is
    started. If the `MOUNT_TMPFS` option is enabled, then it is run
    just after the tmpfs is mounted. Default empty.

  * `POST_CMD`: A command which is run just before the tmpfs is
    unmounted and after any cleanups performed by the script itself is
    done.  This post-command will be run even if the script is killed by
    pressing for example `Ctrl-C`. Default empty.

  * `ETA_ALL`: f you want to show the estimated emerge time for all the
    individual packages, before the actual emerge (i.e. during the
    "Pretended emerge" stage), then set `ETA_ALL` to 1. Default 0.


More information about `PRE_CMD` and `POST_CMD`:
------------------------------------------------

The commands in `PRE_CMD` and `POST_CMD` can be one line bash commands or can
be a path name to another script or binary. Multiple bash commands can be
provided if you write them enclosed in the `{ }` construct.

Examples:

  * `PRE_CMD='{ mount /usr/portage/distfiles; echo distfiles mounted; }'`
  * `POST_CMD='{ umount /usr/portage/distfiles; echo distfiles unmounted; }'`
  * `PRE_CMD='/usr/local/bin/my_custom_pre_command'`
  * `POST_CMD='/usr/local/bin/my_custom_post_command'`

Since the commands in `PRE_CMD` and `POST_CMD` are run from inside quietemerge,
they have access to all the variables in quietemerge! You can do
interesting things if you make use of the information in the internal
variables of quietemerge.

For example, `POST_CMD` will have access to the `exit_status` array variable.
`$exit_status` (or `${exit_status[0]}`) contains the number of packages that
have failed and `${exit_status[1]}` contains the number of packages that were
skipped during emerge. Packages might be skipped if `--keep-going` is used
and some package failed to emerge. So, something like the following can be
used if you want to run a custom command only if the emerge process has
succeeded:

```
    POST_CMD='if [[ $exit_status -eq 0 ]]; then my_custom_post_command; else echo some packages failed; fi'
```

Note that it is enclosed in single quotes so that `$exit_status` doesn't get
evaluated. Otherwise you will get an error. `$exit_status` will get evaluated
later when the `POST_CMD` is actually run.

An alternative would be write this into a script and make `POST_CMD` point to
that script. Then you can do more elaborate things.  All the important
variables in the script are commented so that you know which variable is
doing what.
