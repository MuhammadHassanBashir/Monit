# Using Monit to Monitor Docker Containers

      links: **https://github.com/naskio/monit-docker**
      links: **https://www.tecmint.com/monit-linux-services-monitoring/**

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
   docker top "uptime-kuma"              --> uptime-kuma is a container name
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
   sudo nano /etc/monit/conf.d/check_container_my-container.conf
   ```

2. Add the following configuration:
   ```plaintext
   CHECK PROGRAM uptime-kuma WITH PATH /etc/monit/scripts/check_container_my-container.sh
     START PROGRAM = "/usr/bin/docker start uptime-kuma"
     STOP PROGRAM = "/usr/bin/docker uptime-kuma"
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
docker stop uptime-kuma
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


remember: hr container creation k against process create hota ha monit process ko monitor kerta ha linux ma jis sa humy idea hojata ha k container kitni cpu/memory consume ker rha ha





### Steps to Check the Uptime of a Docker Container with Monit on OS(AlmaLinux 9.4 (Seafoam Ocelot) based on RHEL)

This document outlines the steps to monitor the uptime of a Docker container (e.g., `uptime-kuma`) using Monit on a Linux server.

---

### **1. Create a Custom Directory for Monit Scripts**
To organize your Monit scripts, create a directory under `/etc/monit.d` or `/etc/monit`.

#### **Step 1.1: Create the Directory**
Run the following command:
```bash
sudo mkdir -p /etc/monit.d/scripts
```

---

### **2. Create the Monitoring Script**
Create a script to check the status of the Docker container.

#### **Step 2.1: Create the Script File**
Run the following command:
```bash
sudo nano /etc/monit.d/scripts/check_container_uptime-kuma.sh
```

#### **Step 2.2: Add the Script Content**
Paste the following content into the file:
```bash
#!/bin/bash
docker top "uptime-kuma"
exit $?
```

#### **Step 2.3: Make the Script Executable**
Run the following command to make the script executable:
```bash
sudo chmod +x /etc/monit.d/scripts/check_container_uptime-kuma.sh
```

---

### **3. Update the Monit Configuration**
Edit the Monit configuration file to include the new script.

#### **Step 3.1: Open the Monit Configuration File**
Locate and open the `monitrc` file. On most systems, it is located at `/etc/monitrc`. Use the following command to edit it:
```bash
sudo nano /etc/monitrc
```

#### **Step 3.2: Add the Monitoring Configuration**
At the end of the file, add the following content:
```plaintext
CHECK PROGRAM uptime-kuma WITH PATH /etc/monit.d/scripts/check_container_uptime-kuma.sh
  START PROGRAM = "/usr/bin/docker start uptime-kuma"
  STOP PROGRAM = "/usr/bin/docker stop uptime-kuma"
  IF status != 0 FOR 3 CYCLES THEN RESTART
  IF 2 RESTARTS WITHIN 5 CYCLES THEN UNMONITOR
```

---

### **4. Reload and Restart Monit**
After updating the configuration, reload and restart Monit.

#### **Step 4.1: Test the Monit Configuration**
Run the following command to validate the configuration syntax:
```bash
sudo monit -t
```
Expected Output:
```plaintext
Control file syntax OK
```

#### **Step 4.2: Reload Monit**
Run the following command to reload Monit:
```bash
sudo monit reload
```

#### **Step 4.3: Start Monit (if not running)**
Ensure that Monit is running:
```bash
sudo systemctl enable monit
sudo systemctl start monit
```

---

### **5. Verify Monit is Monitoring the Container**
Check if Monit is monitoring the `uptime-kuma` container.

#### **Step 5.1: Check Monit Status**
Run the following command:
```bash
sudo monit status
```
You should see an entry for `uptime-kuma` with its current state.

#### **Step 5.2: Simulate a Failure**
To test Monit’s restart functionality, manually stop the container:
```bash
docker stop uptime-kuma
```
Wait for 3 cycles (default cycle is 2 minutes) and verify that Monit restarts the container.

---

### **6. Optional: Adjust Monit Cycle Time**
If you want Monit to check the container’s status more frequently, update the global Monit configuration.

#### **Step 6.1: Edit the Monit Daemon Configuration**
Open the `monitrc` file:
```bash
sudo nano /etc/monitrc
```
Find the following line:
```plaintext
set daemon 120
```
Change `120` (2 minutes) to your desired interval (e.g., `30` for 30 seconds):
```plaintext
set daemon 30
```

#### **Step 6.2: Reload Monit**
Run the following command to apply the changes:
```bash
sudo monit reload
```

---

### **7. Check Logs for Troubleshooting**
If Monit does not behave as expected, check the logs for errors:
```bash
sudo tail -f /var/log/monit.log
```

---

### Summary
1. Created a custom script at `/etc/monit.d/scripts/check_container_uptime-kuma.sh` to monitor the container.
2. Updated the `monitrc` file to include the monitoring configuration.
3. Reloaded and verified Monit’s status to ensure the container is being monitored.

Monit will now:
- Restart the container if it fails for 3 cycles.
- Unmonitor the container if it fails twice within 5 cycles.




