Using **tfenv** inside a Docker image is a great way to manage multiple Terraform versions in your containerized environment. Here’s a **step-by-step guide** to integrate it properly.

---

## **1️⃣ Start with a base image**

Use a Linux image that has `bash`, `git`, `curl`, and `unzip` installed. For your Go + Terraform container, something like this works:

```dockerfile
FROM ubuntu:22.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    bash \
    wget \
    jq \
    && rm -rf /var/lib/apt/lists/*
```

---

## **2️⃣ Install tfenv**

```dockerfile
# Clone tfenv
RUN git clone https://github.com/tfutils/tfenv.git /opt/tfenv

# Add tfenv to PATH
ENV PATH="/opt/tfenv/bin:${PATH}"
```

---

## **3️⃣ Install Terraform versions using tfenv**

You can either install a **specific version** or **read from an environment variable**:

```dockerfile
# Set the Terraform version (example: 1.7.0)
ENV TERRAFORM_VERSION=1.7.0

# Install Terraform via tfenv
RUN tfenv install $TERRAFORM_VERSION && tfenv use $TERRAFORM_VERSION

# Verify installation
RUN terraform -version
```

> ✅ Tip: You can add multiple `tfenv install <version>` lines if you want multiple versions.

---

## **4️⃣ Full Dockerfile Example (Go + Terraform + tfenv)**

```dockerfile
FROM ubuntu:22.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    bash \
    wget \
    jq \
    golang-go \
    && rm -rf /var/lib/apt/lists/*

# Install tfenv
RUN git clone https://github.com/tfutils/tfenv.git /opt/tfenv
ENV PATH="/opt/tfenv/bin:${PATH}"

# Install Terraform
ENV TERRAFORM_VERSION=1.7.0
RUN tfenv install $TERRAFORM_VERSION && tfenv use $TERRAFORM_VERSION

# Verify Terraform installation
RUN terraform -version

# Set workdir
WORKDIR /workspace

# Optional: copy Go + Terraform project
# COPY . /workspace
```

---

## **5️⃣ Usage Notes**

* Now inside the container you can run:

```bash
terraform version
tfenv list
tfenv use <version>
```

* If you want **automatic switching**, you can include a `.terraform-version` file in your project root:

```text
1.7.0
```

tfenv will automatically use that version if you run `tfenv use` in the container.

* You can extend this container to include **AWS CLI, Go, and your Lambda build tools** for a fully reproducible environment.

---

If you want, I can **update your previous Docker image for the Lambda + Terraform SNS module** to include **tfenv, multiple Terraform versions, and automatic switching**, ready for ECS or local testing.

Do you want me to do that?
