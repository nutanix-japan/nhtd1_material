## Test Three-Tier Stack

In this section of the lab, we will test the following:

 * Test the complete application flow from HAProxy through ``Node.js`` app to Postgres database 
 * Verify connectivity, data persistence, and load balancing functionality.​

### Verify VM Connectivity

Confirm all three VMs are running and network accessible from your workstation:

1. Get IP addresses from Prism UI (Compute > VMs) or SSH into each

    ```bash
    ping app-vm-01.local    # Application VM
    ping db-vm-01.local     # Database VM  
    ping haproxy-vm-01.local # Load balancer
    ```

2. SSH test from HAProxy VM to app and DB:
   
    ```bash
    # On haproxy-vm-01
    ssh ubuntu@app-vm-01.local "curl -I http://localhost:3000"
    ssh ubuntu@db-vm-01.local "sudo netstat -tlnp | grep 5432"
    ```
    
    All VMs must resolve via DHCP hostnames or update /etc/hosts on each VM.​

### Test HAProxy Load Balancer

1. Access the application through HAProxy frontend:
    
    === ":octicons-command-palette-16: Command"

        ```bash
        # From workstation or haproxy-vm-01
        curl -i http://haproxy-vm-01.local/
        curl http://haproxy-vm-01.local/todos
        curl -X POST http://haproxy-vm-01.local/todos \
          -H "Content-Type: application/json" \
          -d '{"title":"Test Todo","completed":false}'
        ```

    === ":octicons-command-palette-16: Command ouput"

        ```bash
        HTTP/1.1 200 OK
        [{"id":1,"title":"Test Todo","completed":false}]
        ```

Check HAProxy stats: curl http://haproxy-vm-01.local/haproxy?stats (enable in config).​

### Verify App-to-Database Connectivity

1. On app-vm-01, confirm Node.js connects to Postgres:
    
    ```bash
    # Check app logs
    sudo journalctl -u node -f    # or pm2 logs if using process manager
    tail -f /var/log/node-app.log
    
    # Test direct DB connection from app VM
    sudo apt install postgresql-client
    psql postgres://postgres:labpass@db-vm-01.local:5432/todos -c "SELECT 1;"
    ```

App should show successful DB queries in logs during curl tests.​

### Validate Data Persistence

1. Create and verify data flows through all layers:

    ```bash
    # 1. Create todo via HAProxy
    curl -X POST http://haproxy-vm-01.local/todos \
      -H "Content-Type: application/json" \
      -d '{"title":"Lab Demo Todo","completed":false}' \
      -w "\nStatus: %{http_code}\n"
    ```
    
2. List todos (should show new item)

    ```bash
    curl http://haproxy-vm-01.local/todos
    ```

3. Verify in Postgres directly

    ```bash
    ssh ubuntu@db-vm-01.local "sudo -u postgres psql todos -c \"SELECT * FROM todos;\""
    ```

Database query should match API response JSON.​

## Load Balancing Test (Optional Scale)

1. Deploy second app VM (app-vm-02-userXX) via UI or API, add to HAProxy backend:

2. On the HAProxy VM, modify the ``/etc/haproxy/haproxy.cfg`` file to include the second front-end app VM.

    === ":octicons-file-code-16: /etc/haproxy/haproxy.cfg"
    
        ```bash hl_lines="5"
        # Edit /etc/haproxy/haproxy.cfg on haproxy-vm-01
        backend app-servers
            balance roundrobin
            server app1 app-vm-01-user01.local:3000 check
            server app2 app-vm-02-user01.local:3000 check
        ```

3. Reload haproxy with new settings
   
    ```bash
    sudo systemctl reload haproxy
    ```

4. Test multiple requests:

    ```bash
    for i in {1..10}; do curl http://haproxy-vm-01.local/todos; done
    ```

   Check HAProxy stats for even distribution of traffic.

## Monitoring and Troubleshooting

Open ``VSCode`` > ``Terminal`` on your jumphost

Check logs and ensure functionality:

```bash
ssh ubuntu@haproxy-vm-01-user01 'sudo journalctl -u haproxy -f' # HAProxy logs
ssh ubuntu@app-vm-01-user01 'pm2 logs' # App logs
ssh ubuntu@db-vm-01-user01 'sudo tail -f /var/log/postgresql/*.log' # Postgres logs
```

**Common issues:**

| Issue                | Symptoms                      | Fix                                                                  |
| -------------------- | ----------------------------- | -------------------------------------------------------------------- |
| 502 Bad Gateway      | HAProxy can't reach app       | Verify app:3000, firewall, hostname resolution confluence.atlassian​ |
| DB Connection Failed | App logs show Postgres errors | Check postgres service, password, todos DB exists github​            |
| No HAProxy response  | Port 80 blocked               | sudo ufw allow 80 or disable firewall         |

## Performance Validation

Monitor resource usage in Prism UI (**Compute** > **VMs** > **Analyze**):

- App VM: CPU < 50%, Memory < 3GB during load tests

- DB VM: Disk IOPS low, connections = active users

- HAProxy: Minimal CPU/RAM usage​

Success criteria: 100% API uptime, data persistence, HAProxy health checks green.