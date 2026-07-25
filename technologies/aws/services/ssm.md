# AWS SSM (Systems Manager)

#technology #aws

**SSM** is an umbrella product, not one feature.

The three that matter for app deployment are unrelated to each other except that they share an IAM policy prefix:

- **Parameter Store** — config/secret storage. Plain API calls (`get-parameter`, `put-parameter`), no agent involved.
- **Session Manager** — an interactive shell into an instance with no SSH, no open port 22, no key pair.
- **Run Command** — executes a script on an instance remotely without an interactive session at all — the mechanism behind agentless CI/CD deploys to EC2.

Session Manager and Run Command both depend on the **SSM Agent** running on the instance and authenticating via its **instance profile**; Parameter Store doesn't need the agent at all. Part of: [Amazon Web Services](../aws.md).

---

## Resources

- [AWS Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
- [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html)
- [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

---

## Core Concepts

- **Parameter**
  A single key-value pair — the equivalent of one env var, not a container/bucket. `/showtimex/prod/APP_JWT_SECRET` is one parameter; a "path" like `/showtimex/prod/` is just a naming convention, not an object you create — it only exists implicitly because parameters share that prefix, and lets `get-parameters-by-path` fetch a whole group at once.

- **Standard vs Advanced tier**
  **Standard** — up to 10,000 parameters per account, free. **Advanced** — higher limits + parameter policies (expiration, notifications), $0.05/parameter/month. Default to Standard; nothing about app config usually needs Advanced.

- **SecureString**
  Encrypts the value with a KMS key (default `alias/aws/ssm`, no extra monthly fee) on write, decrypts on read (`--with-decryption`). Each decrypt is a billed KMS API call, but at low request volume this rounds to effectively $0/month.

- **SSM vs Secrets Manager**
  Secrets Manager costs **$0.40/secret/month** + API calls, and adds automatic rotation + tighter native integration with a few services (e.g. auto-rotating RDS credentials on a schedule).

- **SSM Agent + instance profile**
  Session Manager and Run Command only work if the instance has an **IAM instance profile** with `AmazonSSMManagedInstanceCore` (or equivalent) attached, and the agent can reach SSM's service endpoints — either via a NAT Gateway (private subnet) or a public IP + IGW route (public subnet), or via VPC interface endpoints (`ssm`, `ssmmessages`, `ec2messages`) if you'd rather not route through NAT/IGW at all.

- **Why Run Command fits agentless CI/CD deploys**
  Session Manager and Run Command are both **outbound-only** from the instance's perspective — the agent initiates the connection to AWS, so nothing needs to be listening for inbound traffic. A GitHub Actions job can run a script on a private (or public but locked-down) EC2 instance via `aws ssm send-command` with **zero open inbound ports**, no SSH key to manage as a CI secret, and every invocation audited (command history, IAM-authenticated). This is the alternative to SSH-based deploy actions when the target host isn't meant to be directly reachable.

---
