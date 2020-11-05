# Blobfuse
Blobfuse Configuration 
A quick guide on how to install and configure Blobfuse on a Linux RHEL 7+ Server.
Installation Steps
Go to https://packages.microsoft.com/config/rhel/6/packages-microsoft-prod.rpm
Download packages-microsoft-prod.rpm and copy it on to the server
Run the command below to install: 

# rpm –Uvh packages-microsoft-prod.rpm 

Run the command below to install blobfuse
# yum install blobfuse

Also install fuse
# yum install fuse
