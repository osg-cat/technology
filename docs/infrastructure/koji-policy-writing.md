Koji permissions and policy
---------------------------

These are some notes I wrote on Koji ACLs/policy after doing some source diving in the Koji code.
The version of Koji was 1.6.0.
I later found some documentation at <https://docs.pagure.org/koji/defining_hub_policies/>. Go read it, that page has better examples.

### Default policies (defined in hub/kojixmlrpc.py)

    build_from_srpm =
           has_perm admin :: allow
           all :: deny
    build_from_repo_id =
           has_perm admin :: allow
           all :: deny
    channel =
           has req_channel :: req
           is_child_task :: parent
           all :: use default
    package_list =
           has_perm admin :: allow
           all :: deny
    vm =
           has_perm admin win-admin :: allow
           all :: deny

If `MissingPolicyOk` is true (default), then policies that do not exist default to "allow".

### Policy syntax

Policies are set in the `[policy]` section of `/etc/koji-hub/hub.conf`

Each policy definition starts with

    policy_name =

The rest of the definition is indented. The lines in the policy definition have the following format:

Simple tests:

    test [params] [&& test [params] ...] :: action-if-true
    test [params] [&& test [params] ...] !! action-if-false

Complex tests:

    test [params [&& ...]] :: {
       test [params [&& ...]] :: action
       test [params [&& ...]] :: {
          ...
          }
    }

The following generic tests are defined in `koji/policy.py`:

-   `true` / `all`   
    always true

-   `false` / `none`   
    always false

-   `has FIELD`   
    true if policy data contains a field called FIELD

-   `bool FIELD`   
    true if FIELD is true

-   `match FIELD PATTERN1 [PATTERN2 ...]`   
    true if FIELD matches any of the patterns (globs)

-   `compare FIELD OP NUMBER`   
    compare FIELD against a number. OP can be `<, >, <=, >=, =, !=`

the following koji-specific tests are defined in `hub/kojihub.py`:

-   `buildtag PATTERN1 [PATTERN2 ...]`   
    true if the build tag of a build matches a pattern

-   `fromtag PATTERN1 [PATTERN2 ...]`   
    true if the tag we're moving a package from matches a pattern

-   `has_perm PATTERN1 [PATTERN2 ...]`   
    true if user has any matching permission

-   `hastag TAG`   
    true if the build has the tag TAG

-   `imported`   
    true if the build was imported

-   `is_build_owner`   
    true if the user doing this task owns the build

-   `is_child_task`   
    true if the task is a child of some other task

-   `is_new_package`   
    true if the package being looked at is new (i.e. doesn't have an 'id' yet)

-   `method PATTERN1 [PATTERN2 ...]`   
    true if the method matches a pattern

-   `operation PATTERN1 [PATTERN2 ...]`   
    true if current operation matches any of the patterns

-   `package PATTERN1 [PATTERN2 ...]`   
    true if the package name matches any of the patterns

-   `policy POLICY`   
    true if the named policy is true

-   `skip_tag`   
    true if the skip\_tag option is true

-   `source PATTERN1 [PATTERN2 ...]`   
    true if source matches patterns

-   `tag PATTERN1 [PATTERN2 ...]`   
    true if the tag name matches any of the patterns

-   `user PATTERN1 [PATTERN2 ...]`   
    true if username matches a pattern

-   `user_in_group PATTERN1 [PATTERN2 ...]`   
    true if the user is in any matching group

-   `vm_name PATTERN1 [PATTERN2 ...]`   
    true if vm name matches a pattern

The actions are:

-   `allow`   
    allow the action

-   `deny`   
    deny the action

-   `req`   
    ?

-   `parent`   
    ?

-   `use default`   
    ?

### Default permissions

These are the permissions that people can be given in koji:

-   `admin`
-   `build`
-   `repo`
-   `livecd`
-   `maven-import`
-   `win-import`
-   `win-admin`
-   `appliance`

Additional permissions can be added with `grant-permission --new`, see below.

The following permissions are checked by name in the koji command-line utility (i.e. policies are not used):

-   `admin`:   
    `add-group`, `add-tag`, `add-target`, `clone-tag`, `edit-target`, `remove-tag`, `remove-target`, `wrapper-rpm`

-   `maven-import`:   
    `import-archive` with the `--type=maven` option

-   `win-import`:   
    `import-archive` with the `--type=win` option

-   `repo`:   
    `regen-repo`

I haven't found out where some of the other permissions are used.

### Adding permissions

You can give someone a permission that doesn't exist by doing:

    osg-koji grant-permission --new <PERMISSION> "<USER>"

Doing so creates the permission as a side effect.
I'm not aware of a way to create the permission without granting it, other than with database hackery.

### Where policies are used and what policy data is passed on:

#### build\_from\_srpm

**source:**

-   `builder/kojid:BuildTask.handler`   
    used when source url points to an SRPM (as opposed to an scm) and the build is not a scratch build.

**policy data:**

-   `user_id`   
    the owner of the task

-   `source`   
    the url of the source file

-   `task_id`   
    the id of the task

-   `build_tag`   
    the id of the build tag

-   `skip_tag`   
    true if we're not tagging this build (`--scratch` or `--skip-tag` passed on the command line)

-   `target`   
    the build target (only if we have one?)

-   `tag`   
    the destination tag (only if `skip_tag` is false)

#### build\_from\_repo\_id

**source:**

-   `builder/kojid:BuildTask.handler`   
    used when the `--repo-id` option is passed to `koji build`

**policy data:** same as `build_from_srpm`

#### package\_list

**source:**

-   `hub/kojihub.py:pkglist_add`   
    `add-pkg`, `block-pkg`, `set-pkg-arches`, `set-pkg-owner` commands

**policy data:**

-   `action`   
    'add', 'update', 'block' depending on what is being done

-   `force`   
    true if `--force` is passed on the command line

-   `package`   
    package info (the id I think?)

-   `tag`   
    the id of the tag we're trying to add the package to/package is in

**source:**

-   `hub/kojihub.py:pkglist_remove`   
    used internally by the `koji clone-tag` command?

**policy data:** same as above, except `action` is 'remove'

**source:**

-   `hub/kojihub.py:pkglist_unblock`   
    `unblock-pkg` command

**policy data:** same as above, except `action` is 'unblock'

#### tag

!!! note
    RootExports is the class containing functions exported via XMLRPC. In general, each function corresponds to a koji task.

**source:**

-   `hub/kojihub.py:RootExports.tagBuild`   
    tagging builds

**policy data:**

-   `build`   
    the id of the build

-   `fromtag`   
    the id of the tag we're moving the build from, if there is one

-   `operation`   
    'tag' or 'move'

-   `tag`   
    the id of the tag

**source:**

-   `hub/kojihub.py:RootExports.untagBuild`   
    untagging builds

**policy data:** same as above, except `operation` is 'untag', and `tag` is None

**source:**

-   `hub/kojihub.py:RootExports.moveAllBuilds`   
    moving all builds of a package from tag1 to tag2

**policy data:** same as for `tagBuild`, except `operation` is 'move'. The policy is checked once for each build being moved.

**source:**

-   `hub/kojihub.py:HostExports.tagBuild`   
    tagging builds ("host version" ?)

**policy data:** same as for `tagBuild`, plus `user_id`

#### vm

**source:**

-   `hub/kojihub.py:RootExports.winBuild`   
    windows builds in a vm (`win-build` command)

**policy data:**

-   `tag`   
    the destination tag

-   `vm_name`   
    the name of the vm

### Examples

#### Let people with the "build" permission also add packages and build SRPMs

    package_list = 
        has_perm admin :: allow
        has_perm build && match action add update :: allow
        all :: deny

    build_from_srpm =
        has_perm admin build :: allow
        all :: deny

#### Promotion policy for different teams

-   Software team members can tag any package as testing/release.
-   Operations team members can tag vo-clients as testing/release.
-   Security team members can tag CA packages as testing/release.

<!-- this comment actually affects the formatting -->

    promotion =
       has_perm software-team :: allow
       has_perm operations-team && package vo-client :: allow
       has_perm security-team && package *-ca-certs* :: allow
       all :: deny

    tag =
        has_perm admin :: allow
        operation tag :: {
            tag *testing *release* && policy promotion :: allow
            tag *testing *release* !! allow
        }
        operation untag :: {
            fromtag *testing *release* && policy promotion :: allow
            fromtag *testing *release* !! allow
        }
        operation move :: {
            tag *testing *release* && policy promotion :: allow
            fromtag *testing *release* && policy promotion :: allow
            tag *testing *release* !! {
                fromtag *testing *release* !! allow
            }
        }
        all :: deny

