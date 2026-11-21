# Disaster Recovery Test: Automated Failover from `us-west-2a` to `us-west-2b`

An active-passive high-availability and disaster recovery architecture on AWS using **Amazon Route 53**, **Amazon EC2**, **Amazon CloudWatch**, and **Amazon SNS**. This project simulates and verifies automated DNS failover across Availability Zones (`us-west-2a` to `us-west-2b`) during a primary server outage with zero manual intervention.

---

## Architecture Overview

### How It Works
1. **Active-Passive Deployment:** Identical web applications are hosted on two Amazon EC2 instances across separate Availability Zones in **US West (Oregon) (`us-west-2`)**:
   - **Primary Instance:** `CafeInstance1` (`us-west-2a`) — handles all live user traffic under normal conditions.
   - **Secondary Instance:** `CafeInstance2` (`us-west-2b`) — remains on standby ready to absorb traffic immediately if the primary becomes unhealthy.
2. **Automated Health Monitoring:** An **Amazon Route 53** health check monitors the primary endpoint every **10 seconds** (fast interval) and pairs with **Amazon CloudWatch** alarms and **Amazon SNS** to dispatch real-time outage notifications.
3. **Seamless Traffic Redirection:** When `CafeInstance1` is stopped to simulate a production server failure, Route 53 detects consecutive failed health checks, triggers the SNS alarm, and automatically reroutes DNS queries to `CafeInstance2` in `us-west-2b`.

---

## Repository Structure

```text
Automatic-failover/
├── images/
│   ├── Screenshot 2026-10-16 122816.png
│   ├── Screenshot 2026-10-16 122947.png
│   ├── Screenshot 2026-10-16 123127.png
│   ├── Screenshot 2026-10-16 123558.png
│   └── Screenshot 2026-10-16 123924.png
└── README.md
```

---

## AWS Services Used

| Service | Resource / Configuration | Purpose |
| :--- | :--- | :--- |
| **Amazon EC2** | `CafeInstance1` (`us-west-2a`), `CafeInstance2` (`us-west-2b`) | Hosts the primary and secondary Cafe web application servers across two Availability Zones. |
| **Amazon Route 53** | Failover Routing Policy + 10s Health Check | Continuously checks primary endpoint health and automatically switches DNS resolution to the secondary instance on failure. |
| **Amazon CloudWatch** | Health Check Status Alarm | Evaluates Route 53 health check metrics and triggers an alarm state when the primary endpoint fails. |
| **Amazon SNS** | Real-Time Alert Topic & Email Subscription | Delivers immediate email notifications when the primary instance transitions to an unhealthy state. |

---

## Setup & Deployment Walkthrough

### 1. Deploy Multi-AZ EC2 Web Servers (`CafeInstance1` & `CafeInstance2`)
1. Launch **`CafeInstance1`** in **`us-west-2a`** and **`CafeInstance2`** in **`us-west-2b`** with identical Cafe web application configurations.
2. Ensure the Security Group allows inbound **HTTP (Port 80)** traffic so both users and Route 53 health checkers can reach the endpoints.

- **Screenshot:** [View Screenshot 2026-10-16 122816.png](https://github.com/deeh-tech/Automatic-failover/blob/main/images/Screenshot%202026-10-16%20122816.png)

### 2. Configure Route 53 10-Second Health Check & SNS Alarm
1. Open the **Amazon Route 53 Console** > **Health checks** and click **Create health check**.
2. Point the health check to **`CafeInstance1`**'s public IP address on **HTTP Port 80**.
3. Under **Advanced configuration**, set the **Request interval** to **Fast (10 seconds)** for rapid failure detection.
4. Enable **Create alarm** and link it to an **Amazon SNS** topic with a confirmed email subscription so alerts fire immediately on failure.

- **Screenshot:** [View Screenshot 2026-10-16 122947.png](https://github.com/deeh-tech/Automatic-failover/blob/main/images/Screenshot%202026-10-16%20122947.png)

### 3. Configure DNS Failover Routing Records
1. Open your **Hosted Zone** in **Amazon Route 53** and create two **A records** sharing the same domain name:
   - **Primary Record:** Points to `CafeInstance1` (`us-west-2a`) with Routing policy set to **Failover (`Primary`)** and associated with the health check.
   - **Secondary Record:** Points to `CafeInstance2` (`us-west-2b`) with Routing policy set to **Failover (`Secondary`)**.

- **Screenshot:** [View Screenshot 2026-10-16 123127.png](https://github.com/deeh-tech/Automatic-failover/blob/main/images/Screenshot%202026-10-16%20123127.png)

---

## Disaster Recovery Test & Verification

### 4. Simulate Primary Outage & Trigger Unhealthy State
1. Verify baseline traffic resolves to **`CafeInstance1`** (`us-west-2a`) while the health check status is **Healthy**.
2. Open the **Amazon EC2 Console**, select **`CafeInstance1`**, and click **Instance state** > **Stop instance**.
3. Within seconds, Route 53 detects consecutive failed health checks, marks the endpoint **Unhealthy**, and fires an **Amazon SNS** alarm email notification.

- **Screenshot:** [View Screenshot 2026-10-16 123558.png](https://github.com/deeh-tech/Automatic-failover/blob/main/images/Screenshot%202026-10-16%20123558.png)

### 5. Verify Seamless Traffic Redirection to `us-west-2b`
1. Refresh the web application in the browser without making any manual DNS updates.
2. Confirm that Route 53 transparently redirects all incoming requests to **`CafeInstance2`** in **`us-west-2b`**.

- **Screenshot:** [View Screenshot 2026-10-16 123924.png](https://github.com/deeh-tech/Automatic-failover/blob/main/images/Screenshot%202026-10-16%20123924.png)

---

## Key Findings & Takeaways

- **Multi-AZ Isolation:** Hosting identical workloads across `us-west-2a` and `us-west-2b` prevents a single Availability Zone or primary host failure from taking the application offline.
- **Faster Outage Detection:** A 10-second Route 53 health check interval detects endpoint failures faster than the standard 30-second interval, reducing time to failover.
- **Zero-Touch Failover & Alerting:** Stopping `CafeInstance1` automatically triggered health check failure evaluation, SNS alert delivery, and DNS switchover to `CafeInstance2` with no manual record edits.
- **DNS Caching Behavior:** Actual end-user switchover time in production also depends on client and resolver DNS TTL settings.
