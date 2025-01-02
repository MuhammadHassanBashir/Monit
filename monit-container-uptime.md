# Using Monit to Monitor Docker Containers

Monit is an open-source tool used to monitor services and processes, ensuring they run as expected and, if possible, remediating issues automatically. This guide explains how to set up Monit to monitor Docker containers, including container status and basic restart capabilities.

---

## **1. Create a Scripts Directory**

To store custom scripts for Monit:

1. Create a new directory inside the Monit directory:
   ```bash
   sudo mkdir /etc/monit/scripts
   ```

2. Set proper permissions for this directory:
   ```bash
   sudo chmod 750 /etc/monit/scripts
   ```

---

## **2. Create Your Monitoring Script**

1. Navigate to the `scripts` directory:
   ```bash
   cd /etc/monit/scripts
   ```

2. Create a script for monitoring a Docker container. For example, to monitor a container named `my-container`:
   ```bash
   sudo nano check_container_my-container.sh
   ```

3. Add the following content to the script:
   ```bash
   #!/bin/bash
   docker top "my-container"
   exit $?
   ```

   Replace `my-container` with the name of your Docker container.

4. Make the script executable:
   ```bash
   sudo chmod +x /etc/monit/scripts/check_container_my-container.sh
   ```

---

## **3. Add Configuration to Monitor the Script**

1. In your `conf.d` directory (`/etc/monit/conf.d`), create a configuration file for the container:
   ```bash
   sudo nano /etc/monit/conf.d/check_container_my-container
   ```

2. Add the following configuration:
   ```plaintext
   CHECK PROGRAM my-container WITH PATH /etc/monit/scripts/check_container_my-container.sh
     START PROGRAM = "/usr/bin/docker start my-container"
     STOP PROGRAM = "/usr/bin/docker stop my-container"
     IF status != 0 FOR 3 CYCLES THEN RESTART
     IF 2 RESTARTS WITHIN 5 CYCLES THEN UNMONITOR
   ```

---

## **4. Include `conf.d` in the Main `monitrc`**

1. Open your main Monit configuration file (`monitrc`):
   ```bash
   sudo nano /etc/monit/monitrc
   ```

2. Ensure the following line is included and **not commented out**:
   ```plaintext
   include /etc/monit/conf.d/*
   ```

3. Save and close the file.

---

## **5. Reload Monit**

1. Restart the Monit service to apply changes:
   ```bash
   sudo monit reload
   ```

2. Check the Monit status to ensure your script is being monitored:
   ```bash
   sudo monit status
   ```

---

## **Directory Structure After Configuration**

After completing the setup, your Monit-related directories should look like this:

```plaintext
/etc/monit/
├── conf-available
├── conf.d/
│   ├── check_container_my-container
├── conf-enabled
├── monitrc
├── templates
└── scripts/
    ├── check_container_my-container.sh
```

---

## **Testing**

### **Check Logs**
View the Monit logs to ensure there are no errors related to the script or configuration:
```bash
sudo tail -f /var/log/monit.log
```

### **Trigger an Alert**
Stop the container manually to see if Monit restarts it:
```bash
docker stop my-container
```

### **Verify Status**
Run the following command to confirm the container is being monitored:
```bash
sudo monit status
```

---

## **Summary**

By organizing your custom scripts in `/etc/monit/scripts` and configurations in `/etc/monit/conf.d`, you create a clean, modular, and scalable setup for monitoring Docker containers with Monit. This configuration focuses on ensuring container uptime and automatically restarting failed containers, providing a simple yet effective monitoring solution for small-scale setups.

For further monitoring of container resources (CPU, memory, network, etc.), additional scripts and tools like `docker stats` can be integrated into the setup.

with this monit will get the container uptime. you can start , stop, restart , disable container service from monit as well.. 

the script only use the "docker top <container-name>" command exit successfully and see the status "0". its simply mean container is running. if it will not get 0 status its mean container is not running and monit will send you alert nontification.
