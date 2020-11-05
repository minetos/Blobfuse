# Blobfuse
Blobfuse Configuration 
A quick guide on how to install and configure Blobfuse on a Linux RHEL 7+ Server.
Installation Steps
Go to https://packages.microsoft.com/config/rhel/6/packages-microsoft-prod.rpm
Download packages-microsoft-prod.rpm and copy it on to the server
Run the command below to install: 

  #rpm –Uvh packages-microsoft-prod.rpm 

Run the command below to install blobfuse
  #yum install blobfuse

Also install fuse
  #yum install fuse

Prerequisites - Azure Storage Accounts
Read all about the different storage acount types in Azure: https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview
Note 1: Testing has shown that Premium Tier of a General-purpose V2 storage account does not work with Blobfuse. It requires further testing (on my to-do list....)
Note 2: A Blobcontainer is required

BlobFuse Configuration

In Azure, you may use the ephemeral disks (SSD) available on your VMs to provide a low-latency buffer for blobfuse. In Red Hat, the disk is mounted on '/mnt/resource/*’. Execute as root user.

	# mkdir –p /mnt/resource/blobfusetmp  (see note below)*
	# sudo chown root /mnt/resource/blobfusetmp

Note 1: Please check the ephemeral disk as it varies in size. For example, on E32 Type VM’s the ephemeral disk is 512GB. This is enough for a standard database backup (~200GB). On smaller VM’s the ephemeral disk may not enough to store a database backup. In this case we have created a separate logical volume (e.g. /blob/cache) which is used temporarily to store the backup (caching). 

Create a directory /etc/blobfuse where the blobfuse configuration file will be stored. 
	#mkdir –p /etc/blobfuse
	 # chmod 600 /etc/blobfuse

Create the blobfuse configuration file: 
	# vi fuse_connection.cfg
	# chmod 600 fuse_connection.cfg

Populate the configuration file (fuse_connection.cfg) with the following information:
	
# vi /etc/blobfuse/ fuse_connection.cfg
accountName myaccount
accountKey storageaccesskey
containerName mycontainer

Note 1: Change the permissions of the file to 600

Create an empty directory for mounting
# mkdir –p /mycontainer

Mount & Automount Process

Create a script inside /mycontainer with the name blobmount.sh. The contents of the script must contain the blobfuse command along with the parameters required: 
blobfuse /mycontainer --tmp-path=/mnt/resource/blobfusetmp  --config-file=/etc/blobfuse/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120 -o allow_other -o nonempty

Note 1: Change the permissions of the file to 700
 
Edit the /etc/fstab file and add the following entry so the filesystem can be automounted when the server is rebooted:
/mycontainer/blobmount.sh /mnt/resource/blobfusetmp  fuse _netdev,nofail

List the filesystems and test by creating files or folders – see example below:
blobfuse         8124792 2134020   5555012  28% /mycontainer     (not visible when the server is rebooted but still mounted)

In /mycontainer create sub-directories using the mkdir –p command.


Unmount Process

To unmount the blobfuse filesystem: 

	# fusermount –u /mycontainer

Note 1: Do not use the standard mount command to unmount the blobfuse filesystem
Note 2: Do not change the permissions of the blobfuse filesystem using the standard commands. Permissions change by using the “allow_other” flag in the command path


Usefull Links

https://public-wiki.iucc.ac.il/index.php/How_to_mount_Azure_Blob_Storage_inside_a_Linux_machine

https://docs.microsoft.com/en-us/archive/blogs/uk_faculty_connection/blobfuse-is-an-open-source-project-developed-to-provide-a-virtual-filesystem-backed-by-the-azure-blob-storage

https://github.com/Azure/azure-storage-fuse

https://semanticinsight.wordpress.com/2018/04/04/azure-big-data-attach-azure-blob-storage-to-centos-7-vm/
