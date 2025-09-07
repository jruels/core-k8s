# Lab Setup 
## Lab Environment
- **3 VMs**: 1 Leader node + 2 Follower nodes
- **SSH Key**: lab.pem 
- **SSH Username**: ubuntu
- **OS**: Ubuntu 24.04 LTS



# Clone the lab repo in VS Code

### Step 1: Clone the repo

1. Open Visual Studio Code
2. In Visual Studio Code, click **Clone Repository** and paste `https://github.com/jruels/core-k8s`
3. Hit **Enter**, and in the pop-up window, browse to `C:\Users\tekstudent\Downloads\repos`
4. Click **Select as repository destination**
5. When prompted to open the cloned repo, choose **Open**.
6. After opening the folder, click the third icon in the left toolbar for source control. Next to **changes**, click the ellipses (three dots) and choose **pull**.

## Set up a remote SSH session in Visual Studio Code.   

### Create the SSH configuration file.

On the left sidebar, click the icon that looks like a computer with a connection icon.

In the Remote Explorer, hover your mouse cursor over **SSH**, click on the gear icon (⚙️) in the top right corner, and select the top option: `C:\Users\tekstudent\.ssh\config` This will open the SSH configuration file in a new editor tab.


### Add the SSH configuration for the lab servers.

Add the following lines to the SSH configuration file, replacing `<IP of leader>, <IP of node1>, <IP of node2>` with the actual IP addresses.

```plaintext
Host leader
  HostName <IP of leader>
  IdentityFile "C:\Users\tekstudent\Downloads\repos\core-k8s\keys\lab.pem"
  User ubuntu
  
Host node1
  HostName <IP of node1>
  IdentityFile "C:\Users\tekstudent\Downloads\repos\core-k8s\keys\lab.pem"
  User ubuntu
  
Host node2
  HostName <IP of node2>
  IdentityFile "C:\Users\tekstudent\Downloads\repos\core-k8s\keys\lab.pem"
  User ubuntu
```

### Save the SSH configuration file.

Save the changes to the SSH configuration file and close it.


### Connect to the lab servers.

1. In the Remote Explorer, you should now see the entry for each server under "SSH Targets."
2. Click on the entry to connect to each server.
3. Visual Studio Code will open a new window connected to each server.
4. When prompted for the Operating System, choose 'Linux'.  
5. Accept the SSH fingerprint
6. You can now open a terminal in this new window and run commands on the remote servers.

In the VS Code window connected to the leader, open a terminal, and run: 

```bash
git clone https://github.com/jruels/core-k8s
```



### Create a working directory.

In Visual Studio Code, you can create a new folder or file as if it were on your local machine.
Click **Open Folder** and select `/home/ubuntu`.
In future labs, you will create a directory for each lab.
