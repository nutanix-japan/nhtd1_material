## Configure HAProxy VM

1. SSH to HAProxy VM using the **VSCode > Terminal** :octicons-terminal-24:
  
    === "Command"

        ```bash
        ssh -l ubuntu <ip-address-of-haproxy-user01-01>
        ``` 

    === "Command Sample"

        ```{ .text .no-copy }
        ssh -l ubuntu 10.x.x.123
        ```  

4. Install HAProxy

    ```bash
    sudo apt update
    sudo apt install -y haproxy
    ```

5. Configure HAProxy

    Edit configurations:
    
    ```bash
    sudo nano /etc/haproxy/haproxy.cfg
    ```

    Replace or append:
    
    ```bash hl_lines="7"
    frontend http_front
        bind *:80
        default_backend web_servers
    
    backend web_servers
        balance roundrobin
        server web1 _your_fe1_server_ip:80 check
        server web2 _your_fe2_server_ip:80 check
    ```

6. Enable and Restart HAProxy
   
    ```bash
    sudo systemctl enable haproxy
    sudo systemctl restart haproxy
    ```

7. Verify status:

    ```bash
    sudo systemctl status haproxy
    ```
8. Firewall Rules (If Enabled)

    ```bash
    sudo ufw allow 80
    sudo ufw enable
    ```

## Validation

1. Finish the Wordpress install in Browser

    On any browser, point to your web server’s IP, e.g.:
    
    ```url
    http://_your_webserver_ip/wp-admin/install.php
    ```

2. Test Load Balancing

    Open a browser to point to your HAProxy VM IP.
    
    ```url
    http://<HAPROXY_IP>:8404/stats
    ```
