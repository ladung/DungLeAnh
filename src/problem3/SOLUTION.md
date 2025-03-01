### Handling High Memory Usage Issue on a Single NGINX Load Balancer in Production

# Since this is a production issue, I need to resolve it as quickly as possible.
**Step 1: Check Monitoring Dashboard**

- Before directly accessing the server, I need to check the monitoring dashboard to identify if there is a traffic spike.

    - If there is a traffic spike, the best long-term solution is to add more load balancers to ensure high availability and prevent a single point of failure.
    
    - If the traffic is normal, then the issue is likely related to memory leaks, misconfigurations, or possible malicious activity.

**Step 2: Access the Server and Investigate**

- Identify Processes Consuming the Most Memory

Run the following command to check which processes are using the most memory:
```bash
ps aux --sort=-%mem | head -n 10
```

This will list the top 10 processes sorted by memory usage.

*Scenario 1: NGINX is Using Too Much Memory*

- If NGINX is consuming excessive memory, run the following command to see how much memory each NGINX worker process is using:
```
$ ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | grep nginx

#Check log nginx
$ tail -f /var/log/nginx.log

```

Possible Causes & Recovery:

1. Misconfigured NGINX Caching or Large Buffers

- Check the buffer settings in the configuration:
```
grep buffer /etc/nginx/nginx.conf
```
- Check if NGINX cache files are too large:
```
ls -lh /var/cache/nginx
```
**Solution:**

- Reduce buffer sizes in nginx.conf:
```
proxy_buffer_size 16k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;

```

*Scenario 2: NGINX is Not Using the Most Memory*

- If NGINX is not the process consuming the most memory, but the server is only supposed to run NGINX, then it could be a memory leak or malware.

- Investigate Other Processes: Check for unknown processes consuming memory:
```bash
ps aux --sort=-%mem | head -n 10
```

List all open files associated with a suspicious process using lsof. This will help identify if a rogue process is opening unexpected files.

```bash
lsof -p <PID>
```


Possible Causes:
- Memory leak in a background service
- A hidden malware process consuming RAM
- Logs or cache accumulating over time
  
Recovery: If an unknown process is running, kill it and investigate further

```
sudo kill -9 <PID>
journalctl -xe
```