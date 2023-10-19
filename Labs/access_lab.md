# Lab Setup 
To access the lab machine you need to download the GitHub repository. 
Go to the [repo](https://github.com/jruels/ms-labs) and in the top right corner click the green "Code" button, then click "Download as zip". 
Once the download is done extract the zip file to somewhere you can easily access it. 

## macOS
On macOS, or Linux use the terminal to connect to the lab machine.

### Set permission on SSH key 
```
chmod 600 ms-labs-main/keys/lab.pem
```

### SSH to lab servers 
The username for SSH is `ubuntu`
```
ssh -i ms-labs-main/keys/lab.pem ubuntu@<LAB IP> 
```


## Windows 
On Windows use Putty to access the lab machine.   
Open Putty and configure a new session. 
  
![](assets/C4EC1E64-175D-4C84-8C49-D938337FA35A.png)

Expand “Connection_SSH_Auth and then specify the PPK file from the `keys` directory 
![](assets/6FFB137C-1AD8-48A1-97E6-F5F6DA4BC55B.png)

 Now save your session    

![](assets/FD3BA694-FD69-4C86-8EAF-4D5FC813EABA.png)
