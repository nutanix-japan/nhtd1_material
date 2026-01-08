## Test Three-Tier Stack

In this section of the lab, we will test the following:

 * Test the complete application flow from HAProxy through ``Node.js`` app to Postgres database 
 * Verify connectivity, data persistence, and load balancing functionality.​

### Verify VM Connectivity

Confirm all three VMs are running and network accessible from your workstation:

1. Get IP addresses from Prism UI (Compute > VMs) or SSH into each

    ```bash
    ping fe-user01-01    # Application VM
    ping db-user01-01     # Database VM  
    ping haproxy-user01-01 # Load balancer
    ```

2. SSH test from HAProxy VM to app and DB:
   
    ```bash
    # On haproxy-user01-01
    ssh ubuntu@fe-user01-01 "curl -I http://localhost:3000"
    ssh ubuntu@db-user01-01 "sudo netstat -tlnp | grep 5432"
    ```
    
    All VMs must resolve via DHCP hostnames or update /etc/hosts on each VM.​

## Load Balancing Test

1. Install wrk tool on your jumphost VM
   
    ```bash
    sudo apt install wrk -y
    ```

2. Test multiple requests:

    ```bash
    wrk -t4 -c100 -d30s http://_HAPROXY_IP/
    ```

   Check HAProxy stats for even distribution of traffic.

   ```url
   http://_HAPROXY_IP/:8404/stats
   ```

   Let it run for a few minutes or so and observe.


## Performance Validation

Monitor resource usage in Prism UI (**Compute** > **VMs** > **Analyze**):

- App VM: CPU < 50%, Memory < 3GB during load tests

- DB VM: Disk IOPS low, connections = active users

- HAProxy: Minimal CPU/RAM usage​

Success criteria: 100% API uptime, data persistence, HAProxy health checks green.

## Troubleshooting

Open ``VSCode`` > ``Terminal`` on your jumphost

Check logs and ensure functionality:

```bash
ssh ubuntu@haproxy-user01-01-user01 'sudo journalctl -u haproxy -f' # HAProxy logs
ssh ubuntu@fe-user01-01 'pm2 logs' # App logs
ssh ubuntu@db-user01-01-user01 'sudo tail -f /var/log/postgresql/*.log' # Postgres logs
```

**Common issues:**

| Issue                | Symptoms                      | Fix                                                                  |
| -------------------- | ----------------------------- | -------------------------------------------------------------------- |
| 502 Bad Gateway      | HAProxy can't reach app       | Verify app:3000, firewall, hostname resolution confluence.atlassian​ |
| DB Connection Failed | App logs show Postgres errors | Check postgres service, password, todos DB exists github​            |
| No HAProxy response  | Port 80 blocked               | sudo ufw allow 80 or disable firewall         |

