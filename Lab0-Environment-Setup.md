# Lab 0: Environment Setup

| Item | Details |
|---|---|
| Course | IKB42603 Cloud Computing Security Essentials |
| Lab | Lab 0 — Environment Setup |
| Reference | `IKB42603_Lab0_Environment_Setup_Cheatsheet.pdf` |
| Platform shown in evidence | Kali Linux |
| Evidence location | Image files supplied in the project root |

> **Confidentiality notice:** Personal usernames are not reproduced in the report text. The AWS identity shown below is LocalStack's documented dummy identity, not a real AWS credential. No password, private key, session token, MFA seed, or real AWS access key is included.

## 1. Objective

The objective was to install and verify the local tools required for later labs:

- Docker for containers and LocalStack
- AWS CLI v2 for commands sent to LocalStack
- kind and kubectl for a local Kubernetes cluster
- OpenSSL and oathtool as security helper tools
- Trivy through Docker when required in Lab 4

The guide states that the environment runs locally. A real AWS account, credit card, and real AWS credentials are not required.

## 2. Install and verify Docker

### Steps

1. Install Docker using the method for the operating system. On Linux, the guide provides:

   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   ```

2. Log out and back in after changing Docker group membership.
3. Verify the Docker version:

   ```bash
   docker --version
   ```

4. Run the test container:

   ```bash
   docker run --rm hello-world
   ```

### Evidence and result

The evidence reports Docker `28.5.2+dfsg4`. The `hello-world` container completed and displayed Docker's successful-installation message.

![Docker version and hello-world verification](docker.jpeg)

**Result:** Passed.

## 3. Install and verify AWS CLI v2

### Steps

1. Install AWS CLI v2. For Linux, the guide provides:

   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
   unzip awscliv2.zip
   sudo ./aws/install
   ```

2. Verify the installation:

   ```bash
   aws --version
   ```

### Evidence and result

The evidence reports AWS CLI `2.36.10`, satisfying the required 2.x version.

![AWS CLI version verification](AWS.jpg)

**Result:** Passed.

## 4. Install and verify kind

### Steps

1. Download and install kind as shown in the guide:

   ```bash
   curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
   chmod +x ./kind
   sudo mv ./kind /usr/local/bin/kind
   ```

2. Verify the installation:

   ```bash
   kind --version
   ```

### Evidence and result

The evidence reports `kind v0.23.0`.

![kind version verification](KIND.jpg)

**Result:** Passed.

## 5. Install and verify kubectl

### Steps

1. Install kubectl:

   ```bash
   sudo snap install kubectl --classic
   ```

2. Verify the client:

   ```bash
   kubectl version --client
   ```

### Evidence and result

The evidence reports kubectl client `v1.36.3` and Kustomize `v5.8.1`.

![kubectl client verification](KUBECTL.jpg)

**Result:** Passed.

## 6. Verify the helper tools

### 6.1 OpenSSL

1. Run:

   ```bash
   openssl version
   ```

2. Confirm that a version is returned.

The evidence reports OpenSSL `3.6.3` with library version `3.6.2`.

![OpenSSL verification](openSSL.jpg)

**Result:** Passed.

### 6.2 oathtool

1. Install oathtool on Linux if necessary:

   ```bash
   sudo apt install oathtool
   ```

2. Verify it:

   ```bash
   oathtool --version
   ```

The evidence reports OATH Toolkit `2.6.14`.

![oathtool verification](oathtool.jpg)

**Result:** Passed.

### 6.3 Trivy

The guide does not require a separate Trivy installation. Lab 4 runs it through Docker:

```bash
docker run --rm aquasec/trivy image <name>
```

**Result:** No separate installation required.

## 7. Start and check LocalStack

### Steps

1. Start the LocalStack container:

   ```bash
   docker run -d --name localstack -p 4566:4566 localstack/localstack
   ```

2. Check its health:

   ```bash
   curl http://localhost:4566/_localstack/health
   ```

3. Use these commands to stop or start it later:

   ```bash
   docker stop localstack
   docker start localstack
   ```

4. Remove it only when a complete reset is required:

   ```bash
   docker rm -f localstack
   ```

### Evidence and result

Docker downloaded the LocalStack image and returned a container ID. The LocalStack web interface was also reachable on local port `4566`.

![LocalStack image pull and container startup](localstack%20aws.jpg)

![LocalStack interface reachable on localhost](localhostweb.jpeg)

**Result:** LocalStack started and its local interface was reachable. A dedicated screenshot of the `curl .../_localstack/health` response was not supplied.

## 8. Configure AWS CLI for LocalStack

LocalStack accepts dummy credentials. Never enter real AWS credentials for this lab.

### Steps

1. Set the guide's dummy values:

   ```bash
   aws configure set aws_access_key_id test
   aws configure set aws_secret_access_key test
   aws configure set region us-east-1
   ```

2. Set the LocalStack endpoint for the current Bash session:

   ```bash
   EP='--endpoint-url=http://localhost:4566'
   ```

3. Test the connection:

   ```bash
   aws $EP sts get-caller-identity
   ```

### Evidence and result

The command returned LocalStack's dummy identity. Sensitive fields are deliberately represented in the report as follows:

```json
{
  "UserId": "[LOCALSTACK DUMMY USER ID]",
  "Account": "[LOCALSTACK DUMMY ACCOUNT]",
  "Arn": "arn:aws:iam::[LOCALSTACK DUMMY ACCOUNT]:root"
}
```

![AWS CLI connection to LocalStack](otp%20aws%20cli%20configuration.jpeg)

**Result:** Passed. AWS CLI communicated successfully with LocalStack.

## 9. Create and verify the Kubernetes cluster

### Steps

1. Create the cluster named `ccse`:

   ```bash
   kind create cluster --name ccse
   ```

2. Display its information:

   ```bash
   kubectl cluster-info --context kind-ccse
   ```

3. Check the nodes:

   ```bash
   kubectl get nodes
   ```

4. When the cluster is no longer required, remove it:

   ```bash
   kind delete cluster --name ccse
   ```

### Evidence and result

The node output showed:

| Name | Status | Roles | Age | Version |
|---|---|---|---:|---|
| `ccse-control-plane` | Ready | control-plane | 26s | v1.30.0 |

![Kubernetes node verification](kubernetes%20cluster.jpg)

**Result:** Passed. The control-plane node was `Ready`. A separate `kubectl cluster-info` screenshot was not supplied.

## 10. Pre-lab verification checklist

- [x] `docker --version` returns a version.
- [x] `docker run --rm hello-world` succeeds.
- [x] `aws --version` returns AWS CLI 2.x.
- [x] `kind --version` succeeds.
- [x] `kubectl version --client` succeeds.
- [x] LocalStack starts.
- [x] The LocalStack interface is reachable on localhost.
- [x] `aws $EP sts get-caller-identity` returns a local identity.
- [x] `kind create cluster --name ccse` creates a cluster.
- [x] `kubectl get nodes` shows the control-plane node as `Ready`.
- [x] OpenSSL returns a version.
- [x] oathtool returns a version.
- [ ] Capture the exact `curl http://localhost:4566/_localstack/health` output.
- [ ] Capture `kubectl cluster-info --context kind-ccse` for complete evidence.

## 11. Security and privacy review

- The AWS values used here are LocalStack dummy values only.
- Real AWS keys, passwords, MFA/TOTP seeds, private keys, tokens, and cookies must never be placed in screenshots or reports.
- Personal terminal usernames are omitted from the written results.
- Container IDs, image digests, local cluster names, `localhost` URLs, and LocalStack's all-zero dummy account are not authentication secrets.
- If evidence is shared publicly, crop or blur personal display names visible in application chrome even though they are not credentials.

## 12. Conclusion

The Lab 0 environment was successfully prepared. Docker, AWS CLI v2, kind, kubectl, OpenSSL, and oathtool were installed and verified. LocalStack started locally and accepted an AWS STS request through the configured endpoint. The kind cluster was operational, with its control-plane node in the `Ready` state.

The two remaining documentation items are screenshots of the exact LocalStack health command and `kubectl cluster-info`; these do not invalidate the successful setup shown by the supplied evidence.
