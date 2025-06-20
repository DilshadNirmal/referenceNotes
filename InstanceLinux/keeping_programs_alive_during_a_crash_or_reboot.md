# Using Systemd Service to Restart MongoDB Instances

This approach builds on the replica set creation guide and sets up MongoDB instances inside an AWS EC2 instance running Ubuntu. By creating `systemd` service files, you can ensure your MongoDB instances are managed and restarted automatically when needed.

---

## Step 1: Create Systemd Service Files for Each Instance

For each MongoDB instance, create a separate service file:

1. **Create a Service File for Instance 1**
    
    ```bash
    sudo nano /etc/systemd/system/mongo1.service
    ```
    
    Similarly, create service files for the other instances, e.g., `mongo2.service` and `mongo3.service`.
    

---

## Step 2: Add Configuration to the Service Files

Add the following content to each service file, adjusting the `dbpath` and `port` for each instance:

```ini
[Unit]
Description=MongoDB Instance 1
After=network.target

[Service]
ExecStart=/usr/bin/mongod --bind_ip 0.0.0.0 --dbpath /path/to/27017 --port 27017 --replSet "rs0"
Restart=always
User=mongodb
Group=mongodb

[Install]
WantedBy=multi-user.target
```

Replace `/path/to/27017` with the appropriate database path for the instance. Repeat this for `mongo2.service` and `mongo3.service` with the respective `dbpath` and `port` values.

---

## Step 3: Enable and Start the Services

Once the service files are created, follow these steps:

1. **Reload Systemd Daemon**
    
    ```bash
    sudo systemctl daemon-reload
    ```
    
2. **Enable the Services** This ensures the services start automatically on reboot:
    
    ```bash
    sudo systemctl enable mongo1.service
    sudo systemctl enable mongo2.service
    sudo systemctl enable mongo3.service
    ```
    
3. **Start the Services**
    
    ```bash
    sudo systemctl start mongo1.service
    sudo systemctl start mongo2.service
    sudo systemctl start mongo3.service
    ```
    

---

## Step 4: Verify Service Status

Check the status of each MongoDB service to ensure they are running correctly:

```bash
sudo systemctl status mongo1.service
sudo systemctl status mongo2.service
sudo systemctl status mongo3.service
```

You should see an `active (running)` status if everything is set up correctly.

---

## Benefits of Using Systemd Services

By setting up these `systemd` service files:

- Your MongoDB instances will automatically restart if they crash or when the EC2 instance reboots.
- You can manage each instance independently through `systemctl` commands.

This setup ensures a robust and reliable environment for your MongoDB replica set.