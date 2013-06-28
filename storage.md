rhos-qe-guide
=============

Adding nfs server to cinder: 
=============================

We already have nfs servers installed but check the following configuration on the storage server: 

*  make sure the nfs-utils package exists: 'rpm -qa |grep nfs-utils'
*  make sure the nfs service is up: 'service nfs status' 
*  make sure the nfs service is on in config: ' chkconfig --list |grep nfs' 
*  make sure that /share exists: 'ls -l / |grep share'
*  make sure the share exists in exports: 'cat /etc/exports |grep share'
*  Make sure that the share dir has the following: (rw,sync,no_root_squash)
*  make sure that mount is exposed by running: 'showmount -e'

OpenStack compute side: 

1. install openstack using packstack on a clean rhel6 host
2. Add nfs_shares_config to a new share.conf file and add cinder.volume.nfs.NfsDriver and volume_driver params to 
the cinder conf: 

  'openstack-config --set /etc/cinder/cinder.conf DEFAULT \nfs_shares_config /etc/cinder/shares.conf'
  'openstack-config --set /etc/cinder/cinder.conf DEFAULT \volume_driver cinder.volume.nfs.NfsDriver'
  
3. add the server share to the share.conf on openstack: 
 
  'echo "<storage_server>:/share" > /etc/cinder/shares.conf

4. create new mnt dir:

  'mkdir /var/lib/cinder/mnt'
  
5. change mnt dir permissions to cinder user/group: 

  'chown -v cinder.cinder /var/lib/cinder/mnt'
  
6. verify the changes: 
 
  'cat /etc/cinder/shares.conf'
  'grep -i nfs /etc/cinder/cinder.conf | grep -v \#'

7. change selinux rule to allow nfs: 

  'setsebool virt_use_nfs on'
  
8. check that rpcbind is running: 

  'service rpcbind status'
  'service rpcbing start'

9. restart cinder services: 

  'for i in $(chkconfig --list | awk ' /cinder/ { print $1 }'); do service $i restart; done'
  
10. make sure the services are up: 

'for i in $(chkconfig --list | awk ' /cinder/ { print $1 }'); do service $i status; done

11. delete cinder-volumes vg:

  'vgs'
  'vgremove cinder-volumes'
  'vgs'
  
12. you can now create a persistent volume: 

  'cinder create --display-name <name> <size>'
  
13. make sure that the volume is ACTIVE:

  'cinder list'
  
14. attach the volume to a list:

  'nova list'
  'nova volume-list'
  'nova volume-attach <instance> <volume> auto' *you can use auto or a specific interface (/dev/sdc)
  'nova volume-list' - to see that the volume is attached















