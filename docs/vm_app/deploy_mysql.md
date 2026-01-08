
## Deploy MySQL Database 

1. SSH to MySQL VM using the **VSCode > Terminal** :octicons-terminal-24:
  
    === "Command"

        ```bash
        ssh -l ubuntu <ip-address-of-db-user01-01>
        ``` 
    === "Command Sample"

        ```{ .text .no-copy }
        ssh -l ubuntu 10.x.x.124
        ```     

2. Install MySQL
   
    ```bash
    sudo apt install mysql-server
    ```

3. Secure MySQL Installation. Run the built-in script to improve security (set root password, remove test DB, disallow anonymous login, etc.)
   
    Follow prompts to:
    
    - Set root password
    
    - Remove anonymous users
    
    - Disallow remote root login (optional)
    
    - Remove test database
   
    === ":octicons-command-palette-16: Command"
  
        ```
        sudo mysql_secure_installation
        ```
         
    === ":octicons-command-palette-16: Command output"
  
        ```bash
        sudo mysql_secure_installation
      
        Securing the MySQL server deployment.
        
        Connecting to MySQL using a blank password.
        The 'validate_password' component is installed on the server.
        The subsequent steps will run with the existing configuration
        of the component.
        
        Skipping password set for root as authentication with auth_socket is used by default.
        If you would like to use password authentication instead, this can be done with the "ALTER_USER" command.
        See https://dev.mysql.com/doc/refman/8.0/en/alter-user.html#alter-user-password-management for more information.
        
        By default, a MySQL installation has an anonymous user,
        allowing anyone to log into MySQL without having to have
        a user account created for them. This is intended only for
        testing, and to make the installation go a bit smoother.
        You should remove them before moving into a production
        environment.
        
        Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
        Success.
        
        
        Normally, root should only be allowed to connect from
        'localhost'. This ensures that someone cannot guess at
        the root password from the network.
        
        Disallow root login remotely? (Press y|Y for Yes, any other key for No) : N
        
          ... skipping.
        By default, MySQL comes with a database named 'test' that
        anyone can access. This is also intended only for testing,
        and should be removed before moving into a production
        environment.
        
        
        Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
          - Dropping test database...
        Success.
        
          - Removing privileges on test database...
        Success.
        
        Reloading the privilege tables will ensure that all changes
        made so far will take effect immediately.
        
        Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
        Success.
        
        All done! 
        ```

4. Create Database & User, login to MySQL:
   
    ```bash
    sudo mysql -u root -p
    # Press enter to login. There is no password set yet
    ```


5. Inside MySQL prompt, run the following database setup commands:

    ```sql
    CREATE DATABASE appdb;
    CREATE USER 'appuser'@'%' IDENTIFIED BY 'StrongPassword123';
    GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
    FLUSH PRIVILEGES;
    EXIT;
    ```

6.  Allow Remote Connections to MySQL database
    
     ```bash
     sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
     ```
    
    Find:
    
     ```text
     bind-address = 127.0.0.1
     ```
    
    Change to your DB server IP (e.g., 192.168.1.100) or 0.0.0.0 for all interfaces:
    
     ```text
     bind-address = 0.0.0.0
     ```

7. Restart MySQL:

    ```bash
    sudo systemctl restart mysql
    ```

We are all set with MySQL database to receive incoming connections. Let's move on to setting up Wordpress as a front-end application. 