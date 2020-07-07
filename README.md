# Server Administration Quick Reference

## Table of Contents
1. [Filesystem Commands](#filesystem-commands)
2. [Kubernetes Commands](#kubernetes-commands)
3. [Vim](#vim)


## Introduction

This is an assorted list of potentially useful Unix commands. These are commands I've found useful in the past that I don't want to forget.


## Filesystem Commands

#### Zeroing out a file

    cat/dev/null > <filename>



#### Grep multiple files across multiple directories

    find . -type f -print | xargs fgrep -i -l "text_to_grep_for"


#### Find files larger than 1000000 bytes

    ll -R <directoryname> | awk '{if ($5 > 1000000) print $5 "\t" $9}'

    find . -xdev -size +1000000c

On Linux:

    find . -xdev -size +1000000c -exec ls -al {} \; | sort -k 5n

On HP-UX:

    find . -xdev -size +1000000c -exec ls -al {} \; | sort -r +4n


#### Find files modified 3 or fewer days ago

On Linux:

    find . -xdev -type f -mtime 3 -exec ls -al {} \; | sort -rk +5n

On HP-UX:

    find . -xdev -type f -mtime 3 -exec ls -al {} \; | sort -r +4n


#### Delete object files and print what's deleted

    find . -name "*.o" -exec echo 'rm -f {}' \; -exec rm -f {} \;


#### Show list of directories and their sizes in kilobytes

On Linux:

    du -xhk | sort -k 1n

On HP-UX:

    du -xk | sort +0n


#### Show how much space a directory is taking up in kilobytes

    du -xks


Alternate version (only shows one level):

    du -shx



#### Determine what process has a file open

    fuser -u /path/to/file

    psg <pid>



#### Monitor a directory for open filehandles. Repeat command every second forever.

    while true; do lsof +d /tmp; sleep 1; done



#### Show space on drives

    df -k


#### Querying file locks max value on HP-UX

    kcusage | grep nflocks


#### Create a file of a given size

    dd if=/dev/zero of=don.out bs=1024 count=10240



#### Delete file by inode

    ls -li

    find . inem <inode number> -exec rm -l {} \;



#### Search/Replace in a file

    sed -ri 's/(test1|test2)/value3/g' app.cfg



#### Delete logfiles without an open file handle

    find /tmp -type f -exec bash -c "fuser {} || rm {}" \;



#### Determine if it is an HDD or SSD (0 means SSD, 1 means HDD)

    cat /sys/block/sdc/queue/rotational  



#### Manually formatting and mounting a block device

    lsblk  
    file -s /dev/xvdb  
    mkfs -t ext4 /dev/xvdb  
    mkdir /mnt/jenkins  
    mount /dev/xvdb /mnt/jenkins  
    ll /mnt/jenkins  
    df -h  


## Kubernetes Commands

### General Commands

#### Login on Azure

    az aks get-credentials --name <resource name> -g <resource group>

    az aks get-credentials --name <resource name> -g <resource group> --admin



#### Get Context

    kubectl config -get-contexts



#### Pods

    kubectl get pods --all-namespaces

    kubectl get pods -l app=nginx-ingress --all-namespaces

    kubectl get pods -l app=nginx-ingress -o wide --namespace=kube-system

    kubectl describe pods <pod name> --namespace=kube-system



#### Services

    kubectl get services --all-namespaces

    kubectl get svc <service name> --namespace=kube-system

    kubectl describe services --all-namespaces

    kubectl describe svc <service name> --namespace=kube-system



#### Deployments

    kubectl describe deployment <deployment name>



#### Configmaps

    kubectl get configmaps

    kubectl get configmaps --namespace=kube-system

    kubectl get configmaps --namespace=kube-system -o yaml



#### SSH

    kubectl exec -it <pod name> -- /bin/bash

    kubectl exec -it <pod name>-n <namespace> -- /bin/bash



#### Endpoints

    kubectl get ep



#### Secrets

    kubectl get secrets --all-namespaces



#### Ingress

    kubectl get ingress



#### Events

    kubectl get events --all-namespaces



#### Logs

    kubectl logs <pod name> --namespace <namespace name>

    kubectl logs -f <pod name>



#### Horizontal Pod Autoscaler

    kubectl autoscale deployment <deployment name> --cpu-percent=50 --min=1 --max=10

    kubectl get hpa

    kubectl describe hpa



#### Scale Deployment

    kubectl scale --replicas =3 <deployment name> -n <namespace name>



### Setting up Ingress

#### Create Cert

    openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out don_test.crt -keyout don_test.key


#### Add Cert

    kubectl create secret tls <secret name> --cert=don_test.crt --key=don_test.key



#### Create Public IP on Azure

    az network public-ip create -g <resource group name> -n <namespace name> --alocation-method static --reverse-fqdn example.westus.cloudapp.azure.com --dns-name example



#### Create Ingress Controller

    helm install <chart name> --namespace <namspace name> --set controller.service.loadBalancerIP="<insert ip here>" --set controller.replicaCount=2



#### Deploy App

    kubectl apply -f <yaml filename>





### Log Analytics on Azure

#### CPU Graph

    Perf |where CounterName == "cpuUsageNaneCores and ObjectName == "K8SContainer"

    | where TimeGenerated > ago(1d)

    | summarize avg(CounterValue), percentiles(CounterValue, 50, 95) by bin(TimeGenerated, 1h)



#### Events

    KubeEvents | where TimeGenerated > ago(1d)

    KubeEvents | where SourceComponent == "cluster-autoscaler"



## Vim

#### Removing ^M characters at end of lines in vi

    :%s/^V^M//g

The ^V is a CONTROL-V character and ^M is a CONTROL-M. When you type this, it will look like this:

    :%s/^M//g

alternate command:

    dos2unix <filename>


#### Comment a block of text in vim

- control-V
- \<highlight text with cursor\> (use arrow keys. Only one column will be highlighted)
- shift-I
- \#
- escape


#### Copy column in vim

 - control-V
 - \<highlight text with cursor\>
 - p



#### Opening a new file in vim

    :n <filename>

    :e <filename>



#### List buffers

    :ls



#### Switch to a different buffer

    :b<buffer number>

    :bnext

    :bprev



#### Open file in new tab

    :tabe <filename>



#### Multiple Windows in Vim

| Syntax | Description |
| ----------- | ----------- |
| :split \<filename\> | split window and load another file |
| vplit \<filename\> | vertical split |
| ctrl-w up arrow | move cursor up a window |
| ctrl-w ctrl-w | move cursor to another window (cycle) |
| ctrl-w_ | maxmize current window |
| ctrl-w= | make all equal size |
| 10 ctrl-w+ | increase window size by 10 lines |
| :hide | close current window |
| :only | keep only this window open |


#### Enable mouse in vim (lets you resize split windows)
    :set mouse=a


#### Set tab to 4 spaces

    set smartindent
    set tabstob=4
    set shiftwidth=4
    set expandtab


#### Enable tab character
    set noexpandtab
