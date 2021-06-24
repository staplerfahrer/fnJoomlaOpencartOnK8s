# fnJoomlaOpencartOnK8s
The full configuration of a real-world K8s deployment from private docker registry, with a number of problems solved.

This project was mainly to learn about Kubernetes and Docker. It was executed on a set of VMs: 1 control plane node, 2 worker nodes, 1 NFS server VM, 1 HAProxy VM. I need to implement a few best practices, such as using a secrets file, PersistentVolumes that address the right sub path (removing it from the deployment yaml files), host names instead of IP addresses.

1)	Joomla & OpenCart are built into a single Docker image because they are integrated with many plugins between them.
2)	DB connections are configured with environment variables. For future reference, replace that with a secrets file (environment vars may leak thru logs, phpinfo() and the like).
3)	Mysql image is completely stock.
4)	Dockerfile creates databases, users, and restores the databases if they are found in the fnjoomlaopencart-vol-claim-mysql-config directory.
5)	Dockerfile installs PHP extensions and Apache mods (per the example of Joomla dockerfile on Docker Hub).
6)	DB deployment yaml file sets the MySQL user, group, and fsgroup context to "user". It mounts to an NFS share with subPath db-vol. This can probably be moved to the PersistentVolume declaration, just include the subPath at the end of the nfs share path.
7)	DB PersistentVolumeClaim specifies the volumeName, to specifically request that particular volume.
8)	Joomla Opencart deployment yaml file modifies the pod host, to pull Docker image from private registry. It also mounts specific subPaths on its volumes.
9)	Joomla Opencart service is configured for LoadBalancer with all pod IP addresses. This can be used for HAProxy load balancing and high availability.
10)	Joomla Opencart volume claims include volumeNames.

HAProxy proved to be easy to configure in a simple test, to forward requests to each one of the nodes and the service's high port number.
