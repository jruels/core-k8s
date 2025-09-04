# Lab Setup 
## Lab Environment
- **3 VMs**: 1 Leader node + 2 Follower nodes
- **SSH Key**: lab.pem 
- **SSH Username**: ubuntu
- **OS**: Ubuntu 24.04 LTS

## MacOS 
Download `lab.pem` from the `keys` directory

### Set permission on SSH key 
```
chmod 600 /path/to/lab.pem
```

### SSH to lab servers 
The username for SSH is `ubuntu`

**Leader node (install Kubernetes here first):**
```
ssh -i /path/to/lab.pem ubuntu@<LEADER_IP>
```

**Follower nodes (join to cluster after leader setup):**
```
ssh -i /path/to/lab.pem ubuntu@<FOLLOWER_1_IP>
ssh -i /path/to/lab.pem ubuntu@<FOLLOWER_2_IP>
```


## Windows 
Download `lab.ppk` from `keys` directory

Open Putty and configure a new session. 
  
![](index/C4EC1E64-175D-4C84-8C49-D938337FA35A%202.png)

Expand “Connection_SSH_Auth and then specify the PPK file 
![](index/6FFB137C-1AD8-48A1-97E6-F5F6DA4BC55B%202.png)

 Now save your session    

![](index/FD3BA694-FD69-4C86-8EAF-4D5FC813EABA%202.png)
