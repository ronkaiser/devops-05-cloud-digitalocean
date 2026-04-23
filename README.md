# Module 5 - Cloud & Infrastructure as Service Basics

**Demo Project:** Create server and deploy application on DigitalOcean

**Technologies used:** DigitalOcean, Linux, Java, Gradle

**Project Description:**

- Setup and configure a server on DigitalOcean
- Create and configure a new Linux user on the Droplet (Security best practice)
- Deploy and run a Java Gradle application on Droplet

*Module 5: Cloud & Infrastructure as Service Basics*

---

## Description

Throughout the DevOps Bootcamp, tools such as Nexus (artifact repository) and Jenkins (build automation) are installed on **remote dedicated servers in the cloud**, not only on your laptop. That pattern matches real environments: you deploy and operate applications on servers you control.

**Cloud computing** is the delivery of computing services (servers, storage, databases, networking, software, and more) over the internet.

**Infrastructure as a Service (IaaS)** gives you compute, storage, and networking on demand. Instead of buying physical hardware, you rent virtual machines and manage the OS and your workloads. IaaS is one model alongside others you will meet later, such as **SaaS**, **PaaS**, and **serverless**.

**IaaS providers:** AWS is widely used and powerful but more complex (covered in a later module). **DigitalOcean** is a practical place to start: fewer moving parts while you learn VMs, SSH, and deployment on a real cloud.

On DigitalOcean, Linux virtual machines are called **Droplets**.

## Prerequisites

- A [DigitalOcean](https://www.digitalocean.com/) account (the course uses signup credits where available).
- An **SSH client** on your computer (OpenSSH is enough: `ssh`, `ssh-keygen`, `scp`).
- **Java 17** and **Gradle** on your machine if you build the course example locally. The sample app uses a Java 17 toolchain (see `java` block in `build.gradle`).
- Optional: read the module handout alongside this README: [05 - Cloud Basics & IaaS Handout.pdf](../02%20-%20Entering%20Learning%20Phase%202%20-%20DevOps%20Fundamentals/05%20-%20Cloud%20%26%20Infrastructure%20as%20Service%20Basics%20with%20DigitalOcean/05%20-%20Cloud%20Basics%20%26%20IaaS%20Handout.pdf).

**Example application (Gradle / Spring Boot):** [java-react-example](../02%20-%20Entering%20Learning%20Phase%202%20-%20DevOps%20Fundamentals/05%20-%20Cloud%20%26%20Infrastructure%20as%20Service%20Basics%20with%20DigitalOcean/java-react-example/)

---

## Setup Server On DigitalOcean

Goal: prepare an Ubuntu Droplet you can reach over SSH and on which you can run a packaged Java application (JAR).

### 1. Create a DigitalOcean account and sign in

Complete account creation in the DigitalOcean control panel. This matches the handout prerequisite for using DO with new-account credits where applicable.

### 2. Configure SSH keys

SSH keys let you log in to any Droplet from your machine **without a password**, using public-key authentication.

**On your computer (examples):**

1. Check for an existing key (often `~/.ssh/id_ed25519.pub` or `~/.ssh/id_rsa.pub`).
2. If you need a new key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519
```

3. Copy the **public** key (`.pub` file). In the DigitalOcean web UI, add it under **Settings → Security → SSH keys** (or add it when you create the Droplet).

Keep your **private** key secret; never upload it to DigitalOcean or commit it to Git.

### 3. Create a Droplet (Linux Ubuntu)

1. In DigitalOcean, choose **Create → Droplets**.
2. Select an **Ubuntu** image (LTS is a good default).
3. Choose a size and datacenter region appropriate for learning.
4. Under **Authentication**, choose **SSH key** and select the key you added.
5. Create the Droplet.

**Terminology:** A Droplet is a **Linux-based virtual machine** running on DigitalOcean’s infrastructure.

### 4. Open SSH (port 22) with a firewall

You must allow **inbound** TCP traffic on **port 22** so your computer can open an SSH session.

On DigitalOcean, create or attach a **Cloud Firewall** (or equivalent networking rules) so that:

- **Inbound rules** describe traffic **into** the Droplet (here: SSH from your IP or a controlled range—tightening the source improves security).
- **Outbound rules** describe traffic **from** the Droplet out to the internet (defaults are often permissive for learning).

Exact clicks vary slightly over time; use DigitalOcean’s docs for **Cloud Firewalls** and attach the firewall to your Droplet.

### 5. SSH into the server using its public IP

1. In the Droplet’s page, copy its **public IPv4** address.
2. From your machine (first login is often as `root` when the provider configures it that way):

```bash
ssh root@YOUR_DROPLET_PUBLIC_IP
```

If you use a non-root user later, replace `root` with that username.

### 6. Install Java on the Droplet

The example application targets **Java 17**.

**Examples on Ubuntu** (run on the Droplet after SSH):

```bash
java -version
```

If Java 17 is not installed, install an OpenJDK 17 runtime (package names can vary by Ubuntu release):

```bash
sudo apt update
sudo apt install -y openjdk-17-jre-headless
java -version
```

Commands may differ on other distributions; adjust to match your Droplet’s OS.

---

## Deploy and run application artifact on Droplet

Goal: **build** a runnable JAR on your machine, **copy** it to the Droplet, and **run** it there—mirroring the handout flow (“in real world, applications will run on a remote server”).

The course example lives in [java-react-example](../02%20-%20Entering%20Learning%20Phase%202%20-%20DevOps%20Fundamentals/05%20-%20Cloud%20%26%20Infrastructure%20as%20Service%20Basics%20with%20DigitalOcean/java-react-example/). After a successful build, the **executable** Spring Boot JAR is typically `build/libs/java-react-example.jar` (alongside a `*-plain.jar` that is not the fat executable—use the one **without** `-plain`).

*You may use `root` or another account for this first pass; the next section documents creating dedicated Linux users as a security best practice for ongoing work.*

### 1. Build the JAR file

On your **local** machine, from the `java-react-example` directory:

```bash
cd "/path/to/java-react-example"
gradle bootJar
```

If your project includes the [Gradle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html), use:

```bash
./gradlew bootJar
```

Confirm the artifact:

```bash
ls -1 build/libs/
```

Use `java-react-example.jar` for deployment.

### 2. Copy the JAR to the remote server

Replace the user, host, and local path as needed.

```bash
scp build/libs/java-react-example.jar root@YOUR_DROPLET_PUBLIC_IP:~/
```

If you deploy under a different Linux user, replace `root` with that username and ensure that user’s home (or target directory) is writable.

### 3. Run the application

SSH to the Droplet, then:

```bash
java -jar ~/java-react-example.jar
```

Spring Boot listens on **8080** by default unless you changed `server.port` in configuration. For a quick test from your laptop (example):

```bash
curl http://YOUR_DROPLET_PUBLIC_IP:8080/
```

**Firewall note:** HTTP **8080** must be allowed **inbound** on the Droplet (or via load balancer) if you reach the app from outside. SSH (22) alone is not enough for browser or `curl` access on 8080.

This runs the process in the **foreground**; stopping the SSH session may stop the app unless you use `tmux`, `screen`, or a process manager—out of scope for the minimal handout steps, but important in production.

---

## Create and configure a Linux user on cloud server

Goal: follow **security best practices** from the handout—do not do everyday work as `root`, and separate responsibilities with Linux accounts.

**Context:** Every cloud platform configures access slightly differently. On a new Droplet you often start as **`root`**. The handout recommends creating a dedicated **admin** user, then (for real systems) creating **per-application** users (for example Nexus, Jenkins, or your app) with only the permissions each workload needs.

### 1. Why avoid routine use of `root`

`root` can change anything on the system. Mistakes are catastrophic, and attackers who compromise `root` control the entire VM. Prefer a normal user with **`sudo`** for administration, and dedicated users for services.

### 2. Create an administrative user (example: `admin`)

SSH in as `root`, then (Ubuntu/Debian style):

```bash
adduser admin
usermod -aG sudo admin
```

Set a strong password or rely on SSH keys for `admin` (keys are preferable). To allow SSH key login for `admin`, copy the authorized key pattern from `/root/.ssh/authorized_keys` to `/home/admin/.ssh/authorized_keys` with correct ownership:

```bash
mkdir -p /home/admin/.ssh
cp /root/.ssh/authorized_keys /home/admin/.ssh/authorized_keys
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
```

From your laptop, test:

```bash
ssh admin@YOUR_DROPLET_PUBLIC_IP
```

### 3. Optional: dedicated user for the application

For a single demo JAR, you might run as `admin`. In broader practice, create a user that **only** runs the app and owns its files:

```bash
sudo adduser --disabled-password --gecos "" myapp
sudo mkdir -p /opt/myapp
sudo mv /path/to/java-react-example.jar /opt/myapp/
sudo chown -R myapp:myapp /opt/myapp
```

Run the service as `myapp` (for example with `sudo -u myapp java -jar /opt/myapp/java-react-example.jar` for a quick test). Production systems usually add **systemd** units or containers so the app restarts automatically and logs consistently—follow your Linux and platform modules for that depth.

### 4. Summary

- **Do not** rely on `root` for daily administration.
- Use an **`admin`** (or similarly named) **sudo** user for maintenance.
- Use **separate users** for major applications, with **least privilege**.
- Align file permissions and SSH keys so each user can only do what it should.

---

## References

- Course handout (full slide narrative): [05 - Cloud Basics & IaaS Handout.pdf](../02%20-%20Entering%20Learning%20Phase%202%20-%20DevOps%20Fundamentals/05%20-%20Cloud%20%26%20Infrastructure%20as%20Service%20Basics%20with%20DigitalOcean/05%20-%20Cloud%20Basics%20%26%20IaaS%20Handout.pdf)
- README structure guidance: [Make a README](https://www.makeareadme.com/)
- DigitalOcean documentation: [Droplets](https://docs.digitalocean.com/products/droplets/), [How to Connect to Droplets with SSH](https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/), [Cloud Firewalls](https://docs.digitalocean.com/networking/firewalls/)
