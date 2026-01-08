## Deploy Application VM via Prism UI

1. SSH to Frontend VM using the **VSCode > Terminal** :octicons-terminal-24:
  
    === "Command"

        ```bash
        ssh -l ubuntu <ip-address-of-fe-user01-01>
        ``` 
    === "Command Sample"

        ```{ .text .no-copy }
        ssh -l ubuntu 10.x.x.130
        ```  

1.  Run the following commands to setup the Front-end application

2.  Install Apache & PHP

     ```bash
     sudo apt install -y apache2
     ```

3.  Install PHP and MySQL extension:

    ```bash
    sudo apt install -y php php-mysql
    ``` 

4.  Install Extra PHP Modules (Optional)

     ```bash
     sudo apt install -y php-gd php-xml php-mbstring
     ```

5.  Disable UFW Firewall for Testing

     ```bash
     sudo ufw disable
     ```

6.  Download & Configure the Web Application (Example: WordPress)

     ```bash
     cd /tmp
     wget https://wordpress.org/latest.tar.gz
     tar -xzvf latest.tar.gz
     sudo cp -r wordpress/* /var/www/html/
     ```
     

17. Set permissions for web root:

     ```bash
     sudo chown -R www-data:www-data /var/www/html
     sudo chmod -R 755 /var/www/html
     ```

18. Set Up WordPress Database Settings. Make a copy of wp-config:

     ```bash
     cd /var/www/html
     sudo cp wp-config-sample.php wp-config.php
     ```


19. Edit wp-config.php:

     ```bash
     sudo nano wp-config.php
     ```


20. Update the following to match your database settings:

     ```bash
     define('DB_NAME', 'appdb');
     define('DB_USER', 'appuser');
     define('DB_PASSWORD', 'StrongPassword123');
     define('DB_HOST', '10.x.x.x'); # (1)
     ```

     1. Remember to update the IP of your database server for DB_HOST.

21. Restart Apache

     ```bash
     sudo systemctl restart apache2
     ```

23. Repeat all steps ``1-22`` to install Wordpress on the second frontend VM ``fe-user01-02``

24. SSH to MySQL VM using the **VSCode > Terminal** :octicons-terminal-24:
  
    === "Command"

        ```bash
        ssh -l ubuntu <ip-address-of-fe-user01-02>
        ``` 
    === "Command Sample"

        ```{ .text .no-copy }
        ssh -l ubuntu 10.x.x.131
        ```  