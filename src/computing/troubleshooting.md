# Troubleshooting

## The great and evil ACL

TODO: what is an ACL (link).
TODO: tl;dr ask Yarik to (TODO: give ACL?).

If you drag and drop a file from your Windows machine into the mounted POSIX environment, its permissions will now be ACLed and you will not be able to do anything with that file until you use `nfs4_setfacl -e filename` to interactively remove the "ACE"s from the file permissions.
(TODO: JH example.)

TODO: <https://github.com/con/opfvta-replication-2023/issues/33>
TODO: <https://rc.dartmouth.edu/wp-content/uploads/2019/04/Intro_to_Cluster.pdf>

## TODO permissions

```shell
nfs4_setfacl -a 'A::EVERYONE@:rwaDdxtTnNcCoy' 1 -R broke-old
```
