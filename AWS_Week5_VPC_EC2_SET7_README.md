# AWS – VPC CREATION AND EC2 ACCESS (WEEK 5 + SET-7)

**Practical order:** Create VPC → Create subnets → Create Internet Gateway → Configure route tables → Enable public IP → Configure security group → Launch EC2 instances → Test → SET-7 AWS scenario questions → Git push commands.

**Sources used:**
- Week 5: `W5- VPC Creation and Ec2 Accessing.docx` – VPC, subnet, gateway, route table, security group, EC2 and 16 practice scenarios.
- `SET-7 (1).docx` – AWS EC2 task and its five SET-7 scenario questions.

**Important:** The Week 5 document describes the AWS practical using the AWS Management Console. It does not provide AWS CLI commands. Follow the console clicks below. The Git commands at the end are for pushing this README to your GitHub repository.

---

# PART 1 – CREATE A VPC

## Step 1: Open AWS Management Console

**WHERE: Browser**

1. Open your browser.
2. Go to `https://console.aws.amazon.com/`.
3. Sign in to your AWS account.
4. In the AWS search bar, type `VPC`.
5. Open **VPC**.

## Step 2: Create the VPC

**WHERE: AWS Console → VPC → Your VPCs**

1. Click **Create VPC**.
2. Under Resources to create, choose **VPC only**.
3. Enter:

| Field | Value |
|---|---|
| Name tag | `Custom-VPC` |
| IPv4 CIDR block | `10.0.0.0/16` |

4. Leave other settings at their defaults unless your lab instructions say otherwise.
5. Click **Create VPC**.
6. Wait for creation to finish.
7. Confirm `Custom-VPC` appears in the VPC list.

**What is a VPC?** A Virtual Private Cloud is an isolated virtual network in AWS where you launch resources such as EC2 instances.

---

# PART 2 – CREATE PUBLIC AND PRIVATE SUBNETS

Use the example CIDR blocks from the Week 5 notes:

| Subnet | CIDR block |
|---|---|
| Public Subnet | `10.0.1.0/24` |
| Private Subnet | `10.0.2.0/24` |

Choose an Availability Zone available in your AWS region. The notes' example places both subnets in the same AZ; for a high-availability production design, use multiple AZs.

## Step 1: Open Subnets

**WHERE: AWS Console → VPC → Subnets**

1. Click **Subnets** in the VPC navigation panel.
2. Click **Create subnet**.

## Step 2: Create the Public Subnet

**WHERE: Create subnet page**

1. Select VPC ID: `Custom-VPC`.
2. Choose an Availability Zone.
3. Subnet name:

```text
Public-Subnet
```

4. IPv4 subnet CIDR block:

```text
10.0.1.0/24
```

5. Click **Create subnet**.

## Step 3: Create the Private Subnet

**WHERE: VPC → Subnets → Create subnet**

1. Click **Create subnet** again.
2. Select VPC ID: `Custom-VPC`.
3. Choose the Availability Zone.
4. Subnet name:

```text
Private-Subnet
```

5. IPv4 subnet CIDR block:

```text
10.0.2.0/24
```

6. Click **Create subnet**.

## Step 4: Verify the Subnets

**WHERE: VPC → Subnets**

Confirm that these two entries appear:

- `Public-Subnet` – `10.0.1.0/24`
- `Private-Subnet` – `10.0.2.0/24`

---

# PART 3 – CREATE AND ATTACH AN INTERNET GATEWAY

## Step 1: Create the Internet Gateway

**WHERE: AWS Console → VPC → Internet Gateways**

1. Click **Internet Gateways**.
2. Click **Create internet gateway**.
3. Name:

```text
Custom-IGW
```

4. Click **Create internet gateway**.

## Step 2: Attach It to the VPC

**WHERE: Internet Gateways → select Custom-IGW → Actions**

1. Select `Custom-IGW`.
2. Click **Actions**.
3. Choose **Attach to a VPC**.
4. Select:

```text
Custom-VPC
```

5. Click **Attach internet gateway**.
6. Confirm that the gateway is attached.

**Purpose:** An Internet Gateway provides a path between the VPC and the internet when the subnet route table and instance public addressing are configured correctly.

---

# PART 4 – CREATE AND CONFIGURE ROUTE TABLES

The Week 5 notes require separate route tables for public and private subnets.

## A. PUBLIC ROUTE TABLE

### Step 1: Create the Public Route Table

**WHERE: AWS Console → VPC → Route Tables**

1. Click **Route Tables**.
2. Click **Create route table**.
3. Name:

```text
Public-Route-Table
```

4. Select VPC:

```text
Custom-VPC
```

5. Click **Create route table**.

### Step 2: Add the Internet Gateway Route

**WHERE: Public-Route-Table → Routes**

1. Select `Public-Route-Table`.
2. Open the **Routes** tab.
3. Click **Edit routes**.
4. Click **Add route**.
5. Destination:

```text
0.0.0.0/0
```

6. Target: choose **Internet Gateway**.
7. Select:

```text
Custom-IGW
```

8. Click **Save changes**.

### Step 3: Associate the Public Subnet

**WHERE: Public-Route-Table → Subnet associations**

1. Open **Subnet associations**.
2. Click **Edit subnet associations**.
3. Select `Public-Subnet`.
4. Click **Save associations**.

Public route table configuration is complete.

## B. PRIVATE ROUTE TABLE

### Step 1: Create the Private Route Table

**WHERE: AWS Console → VPC → Route Tables**

1. Click **Create route table**.
2. Name:

```text
Private-Route-Table
```

3. Select VPC:

```text
Custom-VPC
```

4. Click **Create route table**.

### Step 2: Keep the Local Route

**WHERE: Private-Route-Table → Routes**

1. Select `Private-Route-Table`.
2. Open the **Routes** tab.
3. Keep the default local route.
4. Do not add a route to `Custom-IGW`.

### Step 3: Associate the Private Subnet

**WHERE: Private-Route-Table → Subnet associations**

1. Open **Subnet associations**.
2. Click **Edit subnet associations**.
3. Select `Private-Subnet`.
4. Click **Save associations**.

**Important NAT clarification:** The Week 5 notes say that a private EC2 instance accesses the internet through NAT, but they do not provide NAT Gateway creation steps. A private route table with only the local route does not provide internet access. To enable outbound internet access, a NAT Gateway must be created in a public subnet and the private route table must route `0.0.0.0/0` to that NAT Gateway. Do not add an Internet Gateway route directly to the private subnet.

---

# PART 5 – ENABLE AUTO-ASSIGN PUBLIC IPv4

## Step 1: Open the Public Subnet

**WHERE: AWS Console → VPC → Subnets**

1. Select `Public-Subnet`.
2. Click **Actions**.
3. Choose **Edit subnet settings**.

## Step 2: Enable Public IPv4 Assignment

1. Find **Auto-assign public IPv4 address**.
2. Enable it.
3. Save the changes.

This allows instances launched in the public subnet to receive a public IPv4 address automatically, unless the launch configuration overrides it.

---

# PART 6 – CREATE A SECURITY GROUP

## Step 1: Open Security Groups

**WHERE: AWS Console → VPC → Security Groups**

1. Click **Security Groups**.
2. Click **Create security group**.

## Step 2: Enter Security Group Details

1. Security group name:

```text
Custom-SG
```

2. Description:

```text
Security group for EC2 instances
```

3. Select VPC:

```text
Custom-VPC
```

## Step 3: Add Inbound Rules

Under **Inbound rules**, add:

| Type | Port | Source |
|---|---:|---|
| HTTP | 80 | As required for the lab |
| HTTPS | 443 | As required for the lab |
| SSH | 22 | Trusted IP only |

For SSH, select **My IP** if that is the trusted lab computer's public IP, or enter the company's trusted IP CIDR. Do not expose SSH to everyone using `0.0.0.0/0`.

4. Review the rules.
5. Click **Create security group**.

## Step 4: Attach the Security Group to EC2

When launching each EC2 instance, select `Custom-SG` under the network/security settings, or select the appropriate existing security group for that instance.

**Port note:** SET-7 asks about HTTP and application-server ports. The Week 5 notes list HTTP 80, HTTPS 443 and SSH 22. If Tomcat is configured to listen on another port, allow that port only from the required trusted source. Do not assume port 80 is the Tomcat port.

---

# PART 7 – LAUNCH THE PUBLIC EC2 INSTANCE

**WHERE: AWS Console → EC2 → Instances**

## Step 1: Start Launch Instance

1. Open the **EC2** service.
2. Click **Instances**.
3. Click **Launch instances**.

## Step 2: Configure the Instance

1. Name:

```text
Public-EC2
```

2. Choose the AMI required by your lab.
3. Choose the instance type required by your lab.
4. Select an existing key pair or create one and save the private key safely.
5. Under **Network settings**, click **Edit**.
6. VPC:

```text
Custom-VPC
```

7. Subnet:

```text
Public-Subnet
```

8. Enable public IP assignment.
9. Select the security group `Custom-SG` (or the appropriate security group).
10. Review the configuration.
11. Click **Launch instance**.

## Step 3: Verify the Public EC2 Instance

**WHERE: EC2 → Instances**

1. Select `Public-EC2`.
2. Wait for Instance state to show **Running**.
3. Wait for status checks to pass.
4. Check that it is in `Public-Subnet`.
5. Check that it has a **Public IPv4 address**.

---

# PART 8 – LAUNCH THE PRIVATE EC2 INSTANCE

**WHERE: AWS Console → EC2 → Instances**

## Step 1: Start Launch Instance

1. Click **Launch instances**.
2. Name:

```text
Private-EC2
```

3. Choose the AMI required by your lab.
4. Choose the instance type required by your lab.
5. Select or create a key pair as required.

## Step 2: Configure Network Settings

1. Under **Network settings**, click **Edit**.
2. VPC:

```text
Custom-VPC
```

3. Subnet:

```text
Private-Subnet
```

4. Do not assign a public IPv4 address.
5. Select the appropriate security group.
6. Review the settings.
7. Click **Launch instance**.

## Step 3: Verify the Private EC2 Instance

**WHERE: EC2 → Instances**

1. Select `Private-EC2`.
2. Wait for Instance state to show **Running**.
3. Wait for status checks to pass.
4. Confirm it is in `Private-Subnet`.
5. Confirm it has no public IPv4 address.

---

# PART 9 – TEST THE VPC AND EC2 SETUP

## Test 1: Public EC2 Internet Access

**WHERE: AWS Console → EC2 → Instances**

Check each item:

1. `Public-EC2` is in `Public-Subnet`.
2. It has a public IPv4 address.
3. `Custom-IGW` is attached to `Custom-VPC`.
4. `Public-Route-Table` is associated with `Public-Subnet`.
5. The public route table contains:

```text
Destination: 0.0.0.0/0
Target: Custom-IGW
```

6. The security group allows the required traffic.

Expected result: the public EC2 instance has a valid internet path, provided its operating system and other network settings are configured correctly.

## Test 2: Private EC2 Has No Direct Inbound Access

Check:

1. `Private-EC2` is in `Private-Subnet`.
2. It has no public IPv4 address.
3. Its route table has no direct Internet Gateway route.
4. Its security group allows only the required private traffic.

Expected result: the private EC2 instance is not directly reachable from the public internet.

## Test 3: Private EC2 Outbound Internet Through NAT

The Week 5 notes list this expected result:

```text
Private EC2 → Internet via NAT
```

However, the supplied notes do not include NAT Gateway setup. To demonstrate this result, configure a NAT Gateway in the public subnet, allocate/associate an Elastic IP as required, and add this route to the private route table:

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

The private subnet still must not have a direct Internet Gateway route.

---

# PART 10 – SET-7 AWS DEPLOYMENT USING EC2

<badge>SET-7 – AWS Deployment using EC2 (20 Marks)</badge>

**Scenario from SET-7:** The Metro Railway authority wants passengers to access the Metro Reservation application from different locations. The authority decides to deploy the Java web application on an AWS EC2 instance.

**Task from SET-7:** Demonstrate deployment of the Metro Reservation web application in AWS using an EC2 instance.

**Expected output in SET-7:**
- Launch and configure an AWS EC2 instance.
- Access the Metro Reservation application through the EC2 public IP address.

## Step 1: Launch an EC2 Instance

**WHERE: AWS Console → EC2 → Instances → Launch instances**

1. Click **Launch instances**.
2. Enter a name, for example:

```text
MetroReservation-EC2
```

3. Choose the AMI specified by your lab.
4. Choose the required instance type.
5. Select/create a key pair and save it safely.
6. Under **Network settings**, select the VPC and public subnet you created, if using the custom VPC.
7. Enable public IPv4 assignment.
8. Select a security group that allows:
   - SSH (22) from your trusted IP.
   - HTTP (80) if the web application is served on port 80.
   - The actual Tomcat/application-server port if the application is accessed directly through that port.
9. Click **Launch instance**.

## Step 2: Wait for the Instance

**WHERE: EC2 → Instances**

1. Select `MetroReservation-EC2`.
2. Wait until Instance state is **Running**.
3. Wait until status checks pass.
4. Copy its **Public IPv4 address**.

## Step 3: Connect to the EC2 Instance

**WHERE: EC2 → Instances → select instance → Connect**

1. Select `MetroReservation-EC2`.
2. Click **Connect**.
3. Choose a connection method supported by the AMI and your account, such as EC2 Instance Connect or SSH.
4. Follow the displayed connection instructions.

The SET-7 document does not specify an AMI, operating system, username, key-pair name, Tomcat installation command or application deployment command. Use the commands appropriate to the AMI and the lab instructions; do not assume Linux commands if the selected AMI is Windows.

## Step 4: Deploy the Metro Reservation Application

1. Make sure the application WAR file has been built.
2. Make sure Tomcat or the required application server is installed and running on the EC2 instance.
3. Copy/deploy the WAR file to the application server using the deployment method for that server.
4. Confirm the application server has deployed the application.
5. Confirm the EC2 security group and operating-system firewall allow the application port.

**Important:** The SET-7 question paper does not provide the Tomcat installation or WAR deployment commands. Those commands depend on the selected AMI and Tomcat version, so they are not invented in this procedure.

## Step 5: Access the Application Using the Public IP

**WHERE: Browser**

1. Copy the EC2 instance's Public IPv4 address.
2. Open a browser.
3. Enter the public IP with the actual application-server port and application context path.

General format:

```text
http://<EC2-PUBLIC-IP>:<APPLICATION-PORT>/<APP-CONTEXT>
```

If the application is configured on HTTP port 80, use the appropriate HTTP URL. If Tomcat is listening on another port, use that configured port.

4. Confirm that the Metro Reservation application page opens.

**Expected SET-7 output:** The application is accessible through the EC2 public IP address.

---

# PART 11 – SET-7 AWS SCENARIO-BASED QUESTIONS

These are the five AWS questions from the final page of `SET-7 (1).docx`.

## Q1. What could be the reasons if passengers cannot access the Metro Reservation application using the EC2 public IP address?

**Answer:**

Check the following:

1. EC2 instance is in the **Running** state.
2. Instance has a public IPv4 address.
3. Subnet is public and its route table has `0.0.0.0/0` pointing to the Internet Gateway.
4. Internet Gateway is attached to the correct VPC.
5. Security group allows the application's port.
6. Network ACL and operating-system firewall do not block the traffic.
7. Tomcat/application server is running and the application is deployed.
8. Use the correct public IP, port and application context path in the browser.

## Q2. Why should HTTP and application server ports be configured in the EC2 security group?

**Answer:**

Security groups act as virtual firewalls for EC2 instances. HTTP and the required application-server port must be allowed so users can access the web application. Only necessary ports and trusted sources should be allowed to reduce unauthorized access.

## Q3. What happens if Tomcat stops while the EC2 instance is still running?

**Answer:**

The EC2 virtual machine remains running, but the Java web application hosted by Tomcat becomes unavailable. Users may receive connection errors or service-unavailable errors. Tomcat must be restarted and the application checked.

## Q4. How can you make the Metro Reservation application available if the EC2 instance fails?

**Answer:**

- Run multiple application instances across different Availability Zones.
- Place them behind an Elastic Load Balancer.
- Use health checks to detect unhealthy instances.
- Replace failed instances using Auto Scaling.
- Keep application data in a suitable managed or highly available database.

## Q5. What are the advantages of deploying the Metro Reservation application on AWS instead of a local server?

**Answer:**

1. **Scalability:** Increase or decrease resources when demand changes.
2. **Availability:** Deploy across multiple Availability Zones.
3. **Accessibility:** Users can access the application over the internet.
4. **Flexibility:** Choose different EC2 instance types and services.
5. **Managed services:** Use services such as RDS, load balancers and monitoring tools.
6. **Pay-as-you-go:** Pay for the resources used.

---

# PART 12 – WEEK 5 SCENARIO QUESTIONS

These are the 16 scenario questions from page 3 of the Week 5 VPC document. They are separate from the five SET-7 AWS questions above.

## Q1. You have a web application where users access a website, but the database should not be exposed to the internet. How would you design the VPC? Which resources go into public and private subnets?

**Answer:**
- Place the web server or load balancer in a public subnet.
- Place the database in a private subnet.
- Allow database connections only from the application server's security group.
- Do not give the database direct public internet access.

## Q2. An EC2 instance in a public subnet cannot access the internet. What VPC components would you check and why?

**Answer:**
1. Check that the Internet Gateway is attached to the VPC.
2. Check that the public route table has `0.0.0.0/0` pointing to the Internet Gateway.
3. Check that the public subnet is associated with that route table.
4. Check that the EC2 instance has a public IPv4 address.
5. Check security groups and Network ACLs.

## Q3. Your application server needs to connect to an RDS database securely. How would you configure security groups and subnets?

**Answer:**
- Place RDS in private subnets.
- Use a separate security group for the database.
- Allow the database port only from the application server's security group.
- Do not expose the database directly to the internet.

## Q4. You are asked to design a VPC for 500 servers today, but it should scale to 2,000 servers in the future. How would you choose the CIDR block?

**Answer:**
Choose a sufficiently large VPC CIDR block with enough IP addresses for current and future instances. Plan subnet sizes across Availability Zones, account for AWS-reserved IP addresses and avoid overlapping CIDR ranges.

## Q5. A company wants secure connectivity between its on-premises data center and AWS VPC. Which AWS services would you choose and why (VPN vs Direct Connect)?

**Answer:**
- **Site-to-Site VPN:** Encrypted connection over the internet; usually quicker to set up.
- **Direct Connect:** Dedicated network connection to AWS; useful for more consistent network performance.
Choose based on security, performance, cost and connectivity requirements.

## Q6. You need to connect two VPCs, but both use the same CIDR range. How would you solve this problem?

**Answer:**
Overlapping CIDR ranges cause routing conflicts. Renumber one VPC to use a non-overlapping CIDR range, then configure VPC connectivity.

## Q7. Multiple microservices running in different subnets need to communicate securely. How would you design routing and security groups?

**Answer:**
Use private subnets, configure routes for required communication, and create separate security groups for the microservices. Allow only required ports and sources.

## Q8. Design a VPC for a 3-tier application (Web, App, DB) with high security and scalability. Explain subnets, route tables, gateways, and security groups.

**Answer:**
- Public subnets: web/load-balancer tier.
- Private subnets: application tier.
- Private database subnets: DB tier.
- Internet Gateway: internet route for public subnets.
- NAT Gateway: outbound internet for private application instances if required.
- Separate route tables and security groups.
- Allow only Web → App and App → DB communication as required.

## Q9. You suspect unusual traffic inside your VPC. How would you monitor and analyze network traffic?

**Answer:**
Enable VPC Flow Logs and analyze traffic records using CloudWatch Logs or S3. Use CloudTrail to review AWS API activity and security monitoring tools to investigate suspicious activity.

## Q10. Why can’t a private subnet have an Internet Gateway directly attached?

**Answer:**
An Internet Gateway attaches to a VPC, not directly to a subnet. A subnet is public when its route table has a route to the Internet Gateway and instances have suitable public addressing. A private subnet has no direct Internet Gateway route.

## Q11. What happens if route tables are misconfigured in a VPC?

**Answer:**
Traffic may not reach its destination. Instances may lose internet access or fail to communicate with other subnets. Correct the route targets and subnet associations.

## Q12. What happens if a route table has no local route?

**Answer:**
The local route is normally created automatically for the VPC CIDR. If it is missing or unavailable due to incorrect configuration, resources in different subnets may fail to communicate using the expected local route.

## Q13. An EC2 instance allows traffic on port 80 in the security group, but traffic is still blocked. What could be the reason?

**Answer:**
Check the route table, Internet Gateway, public IP, Network ACL, operating-system firewall and whether the web server is actually running on port 80.

## Q14. You need to create a VPC that will host 1,000+ EC2 instances across multiple AZs. How do you decide the CIDR block?

**Answer:**
Choose a sufficiently large CIDR block for all instances and future growth. Divide it into subnets across multiple Availability Zones and avoid overlapping with connected networks.

## Q15. Only a company’s corporate IP should be able to SSH into EC2 instances. How would you implement this securely in AWS VPC?

**Answer:**
In the EC2 security group's inbound rules, allow SSH TCP port `22` only from the company's trusted public IP/CIDR. Do not allow `0.0.0.0/0`.

## Q16. An EC2 instance can send traffic out but cannot receive responses. Which VPC component might be misconfigured and why?

**Answer:**
A Network ACL may be blocking inbound response traffic. Network ACLs are stateless, so both outbound and inbound response rules, including required ephemeral ports, must be allowed. Also verify the route table.

---

# PART 13 – COMMANDS TO PUSH THIS README TO GITHUB

The AWS practical itself is performed in the AWS Management Console. The following commands are for uploading this README to your existing GitHub repository.

**Repository previously used for your lab files:**

```text
https://github.com/sanjana112233/SE_lab_internal_2.git
```

## Step 1: Open the Local Repository Folder

**WHERE: Windows File Explorer**

1. Open the folder where you cloned `SE_lab_internal_2`.
2. Confirm that it contains a `.git` folder.
3. Copy this README file into that folder:

```text
AWS_Week5_VPC_EC2_SET7_README.md
```

If you have not cloned the repository on this computer, open CMD in the parent folder where you want it and run:

```bat
git clone https://github.com/sanjana112233/SE_lab_internal_2.git
```

Then open the cloned `SE_lab_internal_2` folder and copy the README into it.

## Step 2: Open CMD in the Repository Folder

**WHERE: Windows File Explorer**

1. Open the `SE_lab_internal_2` folder.
2. Click the File Explorer address bar.
3. Type:

```text
cmd
```

4. Press Enter.

CMD opens in that repository folder.

## Step 3: Check Git Status

**WHERE: CMD inside SE_lab_internal_2**

Run:

```bat
git status
```

Check that `AWS_Week5_VPC_EC2_SET7_README.md` appears as an untracked file or modified file.

## Step 4: Add the README

Run:

```bat
git add AWS_Week5_VPC_EC2_SET7_README.md
```

## Step 5: Commit the README

Run:

```bat
git commit -m "Add AWS VPC and EC2 practical README"
```

## Step 6: Push to GitHub

Run:

```bat
git push origin main
```

If your repository's branch is `master` instead of `main`, use:

```bat
git push origin master
```

## Step 7: Verify on GitHub

**WHERE: Browser → GitHub**

1. Open:

```text
https://github.com/sanjana112233/SE_lab_internal_2
```

2. Refresh the repository page.
3. Confirm that `AWS_Week5_VPC_EC2_SET7_README.md` appears.

**If Git says there is no upstream branch**, run the command it suggests or use:

```bat
git push -u origin main
```

If GitHub shows a permission/403 error, check which GitHub account is authenticated and whether that account has write access to `sanjana112233/SE_lab_internal_2`.

---

# FINAL PRACTICAL CHECKLIST

- [ ] Create `Custom-VPC` with CIDR `10.0.0.0/16`.
- [ ] Create `Public-Subnet` with `10.0.1.0/24`.
- [ ] Create `Private-Subnet` with `10.0.2.0/24`.
- [ ] Create and attach `Custom-IGW`.
- [ ] Create `Public-Route-Table`.
- [ ] Add `0.0.0.0/0` → `Custom-IGW`.
- [ ] Associate the public subnet.
- [ ] Create `Private-Route-Table`.
- [ ] Keep the local route and associate the private subnet.
- [ ] Enable Auto-assign Public IPv4 for the public subnet.
- [ ] Create `Custom-SG` with the required inbound rules.
- [ ] Launch `Public-EC2` in the public subnet.
- [ ] Launch `Private-EC2` in the private subnet.
- [ ] Verify the public and private network configurations.
- [ ] For private outbound internet, configure NAT Gateway separately.
- [ ] Complete the SET-7 EC2 deployment task using the public IP.
- [ ] Copy this README into the Git repository.
- [ ] Run `git add`, `git commit` and `git push`.
