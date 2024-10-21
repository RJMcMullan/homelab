# Upgrade process
1. git switch -c release-2.26
1. git fetch upstream release-2.26
1. git pull upstream release-2.26 --force
1. git merge upstream/release-2.26
1. change k8s version in vars and run:
   git commit -m "updating cluster to kubespray release 2.26"