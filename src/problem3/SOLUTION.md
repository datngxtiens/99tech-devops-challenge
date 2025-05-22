Provide your solution here:

## <b>Objective</b>
Step-by-step troubleshooting the issue of VM running 99% of storage and provide possible recovery for each possible root causes.

## Step-by-step Troubleshooting

### Check NGINX status
We first need to check whether our NGINX service is actually running or not:
<pre>sudo systemctl status nginx</pre>

### Check Storage Usage

#### Checking for possible large storage consumption
As the issue we are facing is that the VM is consistently running at 99% storage usage, we will first need to check the overall disk usage to see where the storage is consumed most.  
<pre>df -h</pre>
Above command will show us an overview of the amount of disk space used and available on mounted filesystems.  
To get more details on how our storage is consumed for each directory and its nested content. The command below would give us quick overview of which top-level directories (/var, /home, /usr, etc.) are consuming the most space.
<pre>sudo du -sh /* | sort -rh | head -10</pre>

#### Finding possible causes
By checking the disk consumption, we can get an insight into what is consuming the large portion of our storage.  
One possible cause that we may expect to encounter is that NGINX is throwing out too much log. The indication of this behavior may be large volume size of ```/var``` directory. To better check on this, we check volume size of ```/var/log```, which is the common directory for system and application logs
<pre>du -sh /var/log/*  | sort -hr</pre>
In case of high NGINX log volume, we can also check which file in the directory is responsible for the excessive volume size. We will list all fine/ directories in ```/var/log/nginx/``` folder
<pre>ls -lah /var/log/nginx/</pre>
From the findings above, we can furthur our investigation to determine the root cause of the issue:
- As mentioned above, log file explosion may be one of the cause behind this abnormal behavior. We can furthur investigate on this by running ```tail``` command for the log file in ```/var/log/nginx``` directory and see the frequency new logs are generated. By this, we could also find out any unusual traffic patterns for our NGINX service. There may be a spike of connnection, especially there could be because of some DDOS flood to our system that may cause the problem.
<pre>
tail -n 50 /var/log/nginx/error.log
tail -n 50 /var/log/nginx/access.log
</pre>
- Beside the abnormal traffic flow to NGINX that may case excessive log in our system, one other issue we may face is that the rotation and cleanup of NGINX log is not correctly configure. We may check that by checking the configuration file of NGINX log rotation at ```/etc/logrotate.d/``` directory. Logrotate is a system service that automatically rotates, compresses, and removes old log files; prevents logs from growing infinitely. We could also check the log rotate status by running:
<pre>sudo systemctl status logrotate</pre>

### Check For Any Other Suspicious Actor
We know this instance is running solely NGINX load balancer as a traffic router for upstream services. But we cannot omit the case that we may accidentally deploy other kind of malicious application or other kind of service that may ramp up and consume too much storage of our instance. To confirm there are no suspicious applications/ services that may cause the problem:
<pre>systemctl list-units --type=service --state=running</pre>
The command above woulod look for unusual or unexpected services (e.g., custom, long names, unknown vendors). Beside we could check for any other process that's running on high memory/ CPU, which may then create excessive data volume on our disk.
<pre>
ps aux --sort=-%mem | head -n 20
ps aux --sort=-%cpu | head -n 20
</pre>
Other than that, we could also look at network side to see any process that's listening on unusual port (e.g. NGINX would listen on port 80 and 443 only)
<pre>
sudo lsof -i -P -n | grep LISTEN
ss -tulnp | grep LISTEN
</pre>

### Recovery plan

#### Case: Excessive log
In case of excessive log file of NGINX, we could first delete the current log file, for example:
<pre>
sudo truncate -s 0 /var/log/nginx/access.log
sudo truncate -s 0 /var/log/nginx/error.log
</pre>
Command above will set file size of log file to 0, but not delete it and open it for use for current nginx process. Or we can manually run the log rotation:
<pre>sudo logrotate -f /etc/logrotate.d/nginx</pre>

#### Case: Abnormal traffic/ DDOS attack
We could mitigate the effect of DDOS attack or control the traffic flow into our system by using adopting the usage of Cloudflare or AWS WAF as a protected layer for our NGINX service and also other upstream applications.

#### Case: NGINX and Logrotate not working/ need restart
After making any modification to our configuration for NGINX/ logrotate, we need to restart it for new configuration to take effect
<pre>
sudo systemctl restart nginx
sudo systemctl restart logrotate
</pre>

#### Case: Abnormal process
- We could firstly isolate our instance from the Internet/ from other instances to avoid the malicious program could affect our network and/or other instances.
- Stop the unexpected service:
<pre>
sudo systemctl stop [unexpected_service_name]
sudo kill -9 [suspicious_PID]
</pre>
