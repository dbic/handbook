## The Great and Evil ACL

TODO what is an ACL Link
TODO tldr ask yarik to (TODO give acl?)

If you drag&drop a file from your Windows OS machine into the mounted POSIX environment, its permissions now will be ACLed and you will not be able to do anything with that file until you use `nfs4_setfacl -e filename` to interactive remove "ace"s from the file permissions. (TODO: JH example)
TODO https://github.com/con/opfvta-replication-2023/issues/33
TODO https://rc.dartmouth.edu/wp-content/uploads/2019/04/Intro_to_Cluster.pdf

## TODO permissions

nfs4_setfacl -a 'A::EVERYONE@:rwaDdxtTnNcCoy' 1 -R broke-old


