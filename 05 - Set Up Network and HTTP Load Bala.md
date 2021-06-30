# 05 - Set Up Network and HTTP Load Balancers
## Task 1: Set the default region and zone for all resources
```shell
gcloud config set compute/zone us-central1-a
--------------------------------------------
Updated property [compute/zone].
```

```shell
gcloud config set compute/region us-central1
--------------------------------------------
Updated property [compute/region].
```

## Task 2: Create multiple web server instances
```shell
gcloud compute instances create www2 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"
-------------------------------------------------------------------------------------------------
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/zones/us-central1-a/instances/www1].
NAME  ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
www1  us-central1-a  n1-standard-1               10.128.0.2   35.202.122.220  RUNNING
```

```shell
gcloud compute instances create www3 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
-------------------------------------------------------------------------------------------------
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/zones/us-central1-a/instances/www3].
NAME  ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
www3  us-central1-a  n1-standard-1               10.128.0.3   35.222.210.7  RUNNING
```

```shell
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
---------------------------------------------------------------
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/global/firewalls/www-firewall-network-lb].
Creating firewall...done.
NAME                     NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
www-firewall-network-lb  default  INGRESS    1000      tcp:80        False
```
其中反斜線 `\` 代表指令尚未結束，換行輸入指令。

```shell
gcloud compute instances list
------------------------------------------------------------------------------------
NAME  ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
www1  us-central1-a  n1-standard-1               10.128.0.2   35.202.122.220  RUNNING
www2  us-central1-a  n1-standard-1               10.128.0.4   34.72.6.54      RUNNING
www3  us-central1-a  n1-standard-1               10.128.0.3   35.222.210.7    RUNNING
```

```shell
curl http://[IP_ADDRESS]
curl http://35.202.122.220
------------------------------------------------------
<!doctype html><html><body><h1>www1</h1></body></html>
```


## Task 3: Configure the load balancing service
```shell
gcloud compute addresses create network-lb-ip-1 \
 --region us-central1
--------------------------------------------------
Created 
[https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/addresses/network-lb-ip-1].
```

```shell
gcloud compute http-health-checks create basic-check
----------------------------------------------------
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/global/httpHealthChecks/basic-check].
NAME         HOST  PORT  REQUEST_PATH
basic-check        80    /
```

```shell
gcloud compute target-pools create www-pool \
  --region us-central1 --http-health-check basic-check
------------------------------------------------------
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/targetPools/www-pool].
NAME      REGION       SESSION_AFFINITY  BACKUP  HEALTH_CHECKS
www-pool  us-central1  NONE                      basic-check
```

```shell
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
----------------------------------------------------
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/targetPools/www-pool].
```

```shell
gcloud compute forwarding-rules create www-rule \
    --region us-central1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
-------------------------------------------------
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/forwardingRules/www-rule].
```

## Task 4: Sending traffic to your instances
```shell
gcloud compute forwarding-rules describe www-rule --region us-central1
----------------------------------------------------------------------
IPAddress: 35.223.191.53
IPProtocol: TCP
creationTimestamp: '2021-06-30T09:15:54.749-07:00'
description: ''
fingerprint: QANXKztnhmw=
id: '3701391351325206101'
kind: compute#forwardingRule
labelFingerprint: 42WmSpB8rSM=
loadBalancingScheme: EXTERNAL
name: www-rule
networkTier: PREMIUM
portRange: 80-80
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/forwardingRules/www-rule
target: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/regions/us-central1/targetPools/www-pool
```

```shell
while true; do curl -m1 IP_ADDRESS; done
while true; do curl -m1 35.223.191.53; done
-----------------------------------------------------
<!doctype html><html><body><h1>www1</h1></body></html>
<!doctype html><html><body><h1>www3</h1></body></html>
<!doctype html><html><body><h1>www2</h1></body></html>
<!doctype html><html><body><h1>www3</h1></body></html>
```

## Task 5: Create an HTTP load balancer
HTTP(S) Load Balancing is implemented on Google Front End (GFE). GFEs are distributed globally and operate together using Google's global network and control plane. You can configure URL rules to route some URLs to one set of instances and route other URLs to other instances. Requests are always routed to the instance group that is closest to the user, if that group has enough capacity and is appropriate for the request. If the closest group does not have enough capacity, the request is sent to the closest group that does have capacity.

To set up a load balancer with a Compute Engine backend, your VMs need to be in an instance group. The managed instance group provides VMs running the backend servers of an external HTTP load balancer. For this lab, backends serve their own hostnames.

1. First, create the load balancer template:
   ```shell
   gcloud compute instance-templates create lb-backend-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --image-family=debian-9 \
   --image-project=debian-cloud \
   --metadata=startup-script='#! /bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
   --------------------------------------------------------------
   Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-ec418ae05ed8/global/instanceTemplates/lb-backend-template].
   NAME                 MACHINE_TYPE   PREEMPTIBLE  CREATION_TIMESTAMP
   lb-backend-template  n1-standard-1               2021-06-30T09:21:44.907-07:00
   ```