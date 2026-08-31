# Permissions, ACLs, and Access

Permissions with NFS4’s version of access control lists (ACLs), particularly in the Red Hat Enterprise Linux 8 (RHEL) version used currently on Discovery, Ndoli, and other local Linux machines, are a pain and deserve a page. The most painful, or common, things get tips.

There are times when DartFS permissions seem broken, and so it’s always a reasonable recourse to open a Research Computing ticket to let them look at what you’re doing if you are having trouble (any emails to Research.Computing@dartmouth.edu will generate a ticket). 

NOTE: there is a new storage system (Vast, at `/vast`) used by DBIC on a “temporary” basis starting in 2025. It, too, uses ACLs, but there are differences in how they are handled. A new section below covers system specific behaviors. 

# TL;DR Tips

- Do NOT use the POSIX ACL commands (`getfacl`/`setfacl`) — they do NOT do anything here (you need the `nfs4_` versions)
- DO use `listacl` (if you want) as a local substitute for `nfs4_getfacl`, with nicer output
- ACLs are evaluated top to bottom and so lower lines can take effect even if higher ones do not (*exception:* systems may act on an “Allow” before reaching a lower “Deny”, so early positioning is needed for denials!)
- When setting permissions there is no “recursive” flag, so you need to do your own recursion, e.g. `find My_Results/ -print0 | xargs -0 -n1 nfs4_setfacl -a "A::THATUSER:rxtc"` (substituting for `THATUSER` with a DartID or other entry) to give a user read/traverse permission throughout My_Results, and the ability to see the ACLs (`c`)
    - To cover *future* new files/dirs, you could also add `fd`, e.g. `"A:fd:THATUSER:rxtc"`, to set inheritance — though separate rules are better, so that you can make directories and files have different permissions
- If you copy a file folder (or move it between filesystems) new ACLs are set based on the destination, but if you move a file within a filesystem it keeps its old ACLs
- For DataLad specific advice, refer to https://dbic-handbook.readthedocs.io/en/latest/computing/discovery.html#installing-data
- “Classic” (`chmod`) POSIX permissions sometimes still work, though with limits — see the section on system specific behaviors

# Local Notes

ACLs depend somewhat on the specific implementation. Here, we have RHEL8 (used on Discovery and other local systems), which defines what commands are available and what they try to do, and we have specific storage systems (DartFS and Vast) which actually implement the steps those commands trigger (see separate section on their influence further below).

The system expects the full NFSv4 ACE spec format, which can look like the following (taken from man pages and example usages). Note that it’s important that, for example “OWNER@” is capitalized, and unlike in the older POSIX ACLs (for `setfacl`), there are no “shortcuts” like using `u:` for the owner.

```
A::OWNER@:rwaxtTnNcCy               
A:fdi:OWNER@:rwaDxtTnNcCoy       
A:fdg:groupname:rwaDxtTnNcCoy       
A::EVERYONE@:rxtncy
```

Those lines correspond to “all permissions for owner” (a default setting), the same with some inheritance set (applies to newly created children), something similar for a `groupname` (to be filled in with a real value), and “read access for all”.

See the following section for decoding specifics.

# NFSv4 ACL Format Notes

NFSv4 (or NFS4) — `nfs4_setfacl`, etc. — uses a different, more complex and fine grained, access control list setup than POSIX ACLs (`setfacl`). 

The ACL structure uses colon-separated columns for each ACE (Access Control Entry)

```
A:type:who:perms
D:type:who:perms
```

(’A’ and ‘D’ are actually not the only options, but they are the most common — especially, by far, ‘A’). The meaning of the columns is

- `A` = Allow (or `D` for Deny)
- `type` = flags (colon-separated flags like `fdi`, `fdg`, `f` etc.; see below)
- `who` = principal (OWNER@, groupname, etc.; see below)
- `perms` = permission letters (see below)

The order of the ACEs in the ACL matters. They are evaluated top down, until success, denial, or the end of the list. Note that “until success” means that you *cannot* reliably deny access using `D:` rules at the end (any `D:` should come first, and may terminate evaluation on a match).

## Principal

There are a few possibilities for this column:

- a specific userid (DartID)
- a specific group (e.g. `rc-DBIC`)
- the shorthand entry `OWNER@`, which refers to the uid that currently owns the file/folder
- the shorthand entry `GROUP@`, which similarly refers to the current gid for that file/folder
- an entry `EVERYONE@` that corresponds to the bits for “others” in classic POSIX permissions (but is NOT reading from those bits — this is setting whatever access directly)

## ACE Type

Here are the contents of the man page (`man nfs4_acl`), grouped by the flag category. Inheritance (`fd` flags) is also covered in more detail further below.

Note that `g` for “is a group” is not optional if the principal is a group — it must be present or you’ll get a rejection on an otherwise well formed ACE. 

| Flag | Flag Type | Description | Usage Notes |
| --- | --- | --- | --- |
| **g** | Group flag | Indicates that the principal is a **group** rather than a user. | Used in any ACE. |
| **d** | Directory-inherit | Newly created **subdirectories** will inherit this ACE. | Only valid in ACEs on directories. |
| **f** | File-inherit | Newly created **files** will inherit this ACE (but without inheritance flags). Newly created subdirectories will inherit this ACE; if `d` is not specified, `i` is added to the inherited ACE. | Only valid in ACEs on directories. |
| **n** | No-propagate-inherit | Newly created subdirectories will inherit the ACE **but without its inheritance flags**, so inheritance does not propagate further beyond that level. | Only valid in ACEs on directories. |
| **i** | Inherit-only | The ACE is **not considered when evaluating permissions** on the object itself, only inherited by children. | Only used on ACEs in directories, stripped from inherited ACEs. |
| **S** | Successful-access audit | Trigger an audit/alarm when the principal is **allowed** to perform an action covered by the ACE's permissions. | Only for Audit or Alarm ACEs. |
| **F** | Failed-access audit | Trigger an audit/alarm when the principal is **denied** an action covered by the ACE's permissions. | Only for Audit or Alarm ACEs. |

## Permission letters

Full list from the man page (`man nfs4_acl`):

| Letter  | Permission Name  | Description  |
| --- | --- | --- |
| r | read-data / list-directory | Read the file data (for files) or list the contents (for directories) |
| w | write-data / create-file | Write data to a file or create a file in a directory |
| a | append-data / create-subdirectory | Append data to a file or create a subdirectory |
| x | execute / change-directory | Execute a file or traverse (enter) a directory |
| d | delete | Delete the file/directory itself (some servers allow delete if set here or delete-child in parent) |
| D | delete-child | Delete files or subdirectories within this directory (only applicable to directories) |
| t | read-attributes | Read file/directory attributes |
| T | write-attributes | Write/change file/directory attributes |
| n | read-named-attributes | Read named (extended) attributes |
| N | write-named-attributes | Write named (extended) attributes |
| c | read-ACL | Read the NFSv4 ACL of the file/directory |
| C | write-ACL | Modify (write) the NFSv4 ACL |
| o | write-owner | Change the ownership of the file/directory |
| y | synchronize | Allow synchronous I/O operations with the server |

So `rwaxdDtTnNcCoy` is a full permission mask that could be used to flip every permission bit available. Note that the order doesn’t matter, so any permutation would do the same thing, but this order is the order you’ll see displayed with e.g. `listacl` or `nfs4_getfacl`.

The `n` for `read-named-attributes` is used for extended attributes, which are apparently used by SELinux but could also be defined by other tools (or by users). Setting this is probably harmless, though also probably not required. Not all filesystems support this, and since `setfattr` returns an error apparently user setting of extended attributes is not permitted on DartFS (e.g. `touch testfile; setfattr -n user.testattr -v "hello" testfile` fails). You should be able to see attributes, if any, on local Linux systems using `getfattr`, though as none may be set the output is likely going to be empty.

## Inheritance

There are two particular `Type` values that specify inheritance: 

- ACEs with the `f` flag are inherited by new files
- ACEs with the `d` flag are inherited by new directories

There are also other flags covered in ACE Type above, e.g. `n` to inherit once but not propagate (and note that any ACEs without these flags are also not inherited).

In general, when a new file or directory is created inside a directory, the new object's ACL is created by copying all ACEs marked with inheritance flags (`f`, `d`) from the parent directory.

If a new directory has ACEs with `d` (directory inherit) flags on them, then subsequent children created inside it will also inherit those ACEs — so inheritance can cascade down the directory tree if the ACEs have inheritance flags set.

| Example Action | Result |
| --- | --- |
| Set `A:fd:USER:rxt` on top dir | Current directory and future new files and dirs inherit this ACE |
| Create a subdirectory inside a dir | New directory inherits all ACEs with `d` flag from parent |
| Change ACL on a directory  | Does **not** affect existing children (must set explicitly) |
| Change permission bits on a dir | Controlled by `chmod`, mode bits, and `umask`, separate from NFSv4 ACLs, and may or may not matter (see “system specific behaviors”) |

## Indirect Permissions

These apply when there are “effective” permissions that aren’t explicitly stated in the ACLs. For example, if a directory has an `nfs4_getfacl` line

```bash
A:fd:GROUP@:rxtncy
```

that means that whatever group is set has access (read and execute/traverse). This might appear with `listacl` as for example

```bash
 A:fd:GROUP@               r----xt-n-c--y Allow           ([rc-DBIC])               (Read Traverse)
```

if, as here, the group is `rc-DBIC`. However, this can be more complicated to manage than a line that explicitly gives `rc-DBIC` access,

```bash
A:fd:rc-DBIC@KIEWIT.DARTMOUTH.EDU:rxtncy
```

even though it looks (and functions) about the same:

```bash
 A:fd:rc-DBIC             r----xt-n-c--y Allow             [rc-DBIC]               (Read Traverse)
```

For one thing, if you want to remove access for `rc-DBIC` here, it requires only removing the line in the second case, but it would require changing group owner and/or permissions in the first one. For just one “primary” group, though, `GROUP@` is a simple way to administer group access (it is used for incoming DBIC DICOMs for that reason, for example).

## Bulk Permissions

This is a pointer to a note in the topic “For future admin: permissions on discovery”. Basically, you can manage ACLs using bulk operations by dumping ACLs to a file, editing, and then applying them. 

<aside>
💡 **Changing permissions**

### 1. export permission to txt file.

OR If there is a directory that has the permission setting that you want - you can export this setting for future uses.

`nfs4_getfacl . > spacetop_acl.txt`

### 2. modify the txt file, the way you want permissions to be

`A:fdg:rc-CANlab-spacetop:rwadxtTnNcy`

### 3. run setfacl

```bash
# PROJECT_TOP_DIR is the directory where you want permissions modified
nfs4_setfacl -R -S ./${PERMISSION_ACL}.txt ${PROJECT_TOP_DIR} &

# EXAMPLE: 
DATA=/dartfs-hpc/rc/lab/C/CANlab/labdata/data
nfs4_setfacl -R -S /dartfs-hpc/rc/lab/C/CANlab/labdata/.permissions/data_scebl.txt ${DATA}/PainMem &
```

### 4. check job

`top -u $USER`

### 5. check nfs4_setfacl

`listacl -v *`

</aside>

This is because there are not actually any good tools for making sophisticated ACL changes in bulk — so, instead of modifying, what you end up doing is bulk-replacing existing ACLs with a new copy.

Note that a permissions file can have comments (all lines starting with “#” are ignored), allowing documentation of what the lines are doing:

```jsx
# Admin access to files (Read/Write/Admin, but no execute)
A:g:rc-DBIC-admin@kiewit.dartmouth.edu:rwadtTnNcCoy
A:g:rc-DartFSadmin@kiewit.dartmouth.edu:rwadtTnNcCoy
```

## DBIC Admin Scripts

There are DBIC admin scripts that might also be helpful for batch ACL operations. They default to DBIC groups but can be used for other projects by providing a target principal (i.e. a userid or group). The scripts are located at `/dartfs/rc/lab/D/DBIC/DBIC/code/sbin` and take a <PRINCIPAL> followed by a <TARGET> as arguments:

| `reset-nfs4-acl` | Set all ACEs for a <PRINCIPAL> on a <TARGET> to read only (default principal: rc-DBIC) | Use this to set read only access for a principal |
| --- | --- | --- |
| `reset-nfs4-admin-acl` | Set all ACEs for a <PRINCIPAL> on a <TARGET> to full admin rights (default principal: rc-DBIC-admin) | Use this to set full admin access for a principal (you will have to be the owner or otherwise have good access rights) |

Note that “set all ACEs …” here really means “remove all existing ACEs and set new ones”, since as noted above editing ACEs directly is not actually possible.

## POSIX Permissions

In some ways the “classic” access control for files and folders (owner, group, and those letters like `drwxr-xr-x` you see with `ls -l`) still function with NFS4 ACLs. The “user” and “group” determine who `OWNER@` and `GROUP@` lines apply to, and `EVERYONE@` does something like “other” (though being non specific it doesn’t depend on it). For example, a directory with permissions

```bash
drwxr-xr-x 2 f99xyz0 rc-DBIC-admin 152 Aug 20 13:42 code/hpc 
```

is a directory (`d`) owned by `f99xyz0`, who has permissions `rwx`; it has group `rc-DBIC-admin` and that group is granted `rx`, and `rx` is also granted to “other”. With no ACLs explicitly set, if you view this with `nfs4_acl` you will likely see the permissions translated into ACEs (assuming you yourself have permission `c` to see ACLs through one of the entries):

```bash
A::OWNER@:rwaDdxtTnNcCoy
A::GROUP@:rxtncy
A::EVERYONE@:rxtncy
```

Note that owner gets `rwx` but also every other permission bit; the others get certain bits in addition to `rx` too, including `c`. Inheritance is not enabled.

If you run `chmod g+w $DIR`, where `$DIR` is this directory (*and* have permission for this change, which you do if you are the owner in this case), the ACL may update, and the new line for `GROUP@` would change from having `rxtncy` to `rwxtncy`.

(Note: exact behavior may be somewhat storage system dependent — see section further below.)

## Move vs. Copy

As noted in the intro, if you copy a file folder (or move it between filesystems) new ACLs are set based on the destination, but if you move a file within a filesystem it keeps its old ACLs.

It is possible to override this if desired. With `cp` check out the `-a` option, for example, which preserves “everything”, including xattr (extended attributes), which is where ACLs are stored in Linux. You can also do something similar with `rsync`, though there `-a` isn’t enough and you need to include `-X`, aka `--xattrs`, to again preserve extended attributes. 

# System Specific Behaviors (DartFS, Vast)

The DartFS and Vast filesystems both use ACLs, but they have different behaviors in how they interpret and update them, particularly in relation to POSIX permissions (i.e., `chmod`). Here is a brief summary based on some tests (see `/dartfs/rc/lab/D/DBIC/DBIC/acl_tests/test-script` for the tests).

1. DartFS sometimes ignores `chmod` — specifically, it seems to do so when a custom ACL exists. If you *don't* have any rules set, though, it will attempt to translate `chmod` requests into ACL changes.
2. On Vast, all `chmod` requests seem to be incorporated into the ACL, in a way that preserves the existing rules.

So it seems DartFS's attitude is "once we have an ACL, we only use that", but before there are any custom rules it will create an initial ACL from POSIX changes. 

Vast's approach is more like "let's try to support both types of permission setting requests". This is arguably a more sophisticated and better approach, at least assuming it never messes up.

# References

- https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=88459 is a Dartmouth Research Computing tech note providing guidance on this. It also gets into related topics like adding members to a group.

# Examples

1. **Set permissions on everything below a folder:**

```bash
find My_Results/ -print0 | xargs -0 -n1 nfs4_setfacl -a "A::THIS_PRINCIPAL:rxtc" 
```

(substituting for `THIS_PRINCIPAL` with a principal, such as a DartID). 

This will **append** (`-a`) an **allow** ACE (`A`) with read (`r`), execute (`x`), and read attributes (`t`) permissions and the ability to see the ACLs (`c`) for a particular principal on **every file/directory** found under `My_Results/`. (The attributes cover things like timestamps, size, and classic permissions.)

The `-print0` used by `find` pairs with the `xargs` ”`-0`" (aka `--null`), and means that null characters are used instead of normal line endings, in the process dealing with whitespace. The `-n1` just tells `xargs` that it must run a set number (1) of commands at a time, instead of potentially stacking them up for potential efficiency as it does by default.

Note that there is no “recursive” flag so you have to do the recursion yourself using `find`. You could potentially add other arguments on `find` to be selective; e.g. `-maxdepth` and/or `-mindepth` for recursion depth, `-type l` for symlinks only (or `-not -type l` for no symlinks), `-not -path '**/.**'` to omit hidden “dot” files/folders starting with “.”. (Note on symlinks: `find` here will change their permission, but it never follows them unless you explicitly tell it to with `-L`.)

1. **Do the permission setting in a bash function and only apply `x` (execute/traverse) to directories, not files, and define this as a bash function.**

```bash
add_nfs4acl_recursive() {
  local target_dir="$1"
  local principal="$2"

  echo "Adding rxt to directories..."
  find "$target_dir" -type d -print0 | xargs -0 -n1 nfs4_setfacl -a "A::${principal}:rxtc"

  echo "Adding r to files..."
  find "$target_dir" -type f -print0 | xargs -0 -n1 nfs4_setfacl -a "A::${principal}:rc"
}
```

Once this is added to your `.bashrc` it provides a command `add_nfs4acl_recursive` that can be called as `add_nfs4acl_recursive <directory> <user>`. 

1. **Use a copy to fix permissions (works *if* inheritance is set).**

```bash
cd parentdir
mv baddir baddir.temp
cp -R baddir.temp baddir
listacl baddir
# assuming no issues have come up...
rm -rf baddir.temp
```

If inheritance on `parentdir` is set, then the newly created copy of `baddir` will inherit its permissions.

