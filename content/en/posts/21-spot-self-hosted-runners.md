+++
title = "Cutting CI Costs ~70% With Spot Self-Hosted Runners"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-05-19"
categories = [
    "AWS",
]
menu = "main"
+++

### "Why is our CI bill bigger than our prod bill?"

That was the question that started this. GitHub-hosted runners are wonderful right up until you look at the invoice. They bill per minute, you don't get to pick the hardware, and once you have a few busy repos the numbers stop being cute.

I got tired of overpaying for CI. So I moved the noisy jobs onto my own self-hosted runners, running on AWS EC2 spot instances, all provisioned with Terraform. The result was roughly 70% off that line item. Here's how I did it, and the parts nobody warns you about.

![Self-hosted runner topology: a GitHub Actions job runs on a spot EC2 instance in a Terraform-provisioned VPC, accessed via SSM instead of SSH](/images/21-spot-runners.svg)

### Where the money actually goes

GitHub-hosted Linux runners bill per minute. The standard 2-core runner is cheap per minute, but minutes pile up fast: every push, every PR, every matrix job, every flaky retry. Multiply that across a team and a year and it's a real number.

Two levers fix most of it:

1. Run the work on hardware you rent directly from AWS, not by the minute through a middleman.
2. Rent that hardware on spot, and on ARM.

Spot instances are spare EC2 capacity at a steep discount (often around 70% off on-demand). ARM (Graviton, the `t4g` family) is cheaper than x86 for the same work. Stack the two and the math gets fun.

### A small cost comparison

Numbers are illustrative but reasonable. Say you burn 50,000 CI minutes a month.

```
Option                          | Unit cost        | Monthly (~50k min)
--------------------------------|------------------|-------------------
GitHub-hosted (2-core Linux)    | $0.008 / min     | ~$400
EC2 t4g.small on-demand fleet   | $0.0168 / hr     | ~$120
EC2 t4g.small SPOT fleet        | ~$0.005 / hr     | ~$36
EC2 t4g.nano SPOT (light jobs)  | ~$0.0015 / hr    | ~$11
```

The per-minute model punishes you for being busy. A small always-on spot fleet does not. Even after you add the boring stuff (a NAT or a public subnet, CloudWatch Logs, a bit of data transfer) you land far under the hosted bill. That's where the ~70% comes from.

### The Terraform that does it

The interesting bits are the spot block and the user-data that registers the runner on boot. Everything else (VPC, subnets, security groups, IAM) is standard plumbing, kept least-privilege.

```hcl
resource "aws_launch_template" "runner" {
  image_id      = data.aws_ami.al2023_arm64.id   # ARM64 / Graviton
  instance_type = "t4g.nano"                      # tiny and cheap; size up per workload

  iam_instance_profile { name = aws_iam_instance_profile.runner.name }
  vpc_security_group_ids = [aws_security_group.runner.id]

  # Spot as "persistent" + "stop": an interrupted runner is stopped and brought
  # back, not terminated and lost. Gate it behind a variable so you can flip to
  # on-demand when you actually need guaranteed capacity.
  dynamic "instance_market_options" {
    for_each = var.use_spot ? [1] : []
    content {
      market_type = "spot"
      spot_options {
        spot_instance_type             = "persistent"
        instance_interruption_behavior = "stop"
      }
    }
  }

  user_data = base64encode(templatefile("${path.module}/user-data.sh.tftpl", {
    repo_url  = var.repo_url                              # owner/repo
    token_arn = aws_secretsmanager_secret.runner_pat.arn # the PAT lives here, not in the AMI
  }))
}
```

The user-data pulls the registration token from Secrets Manager (never baked into the AMI) and runs the official runner image as a container that systemd keeps alive:

```bash
#!/bin/bash
set -euo pipefail

# Pull the registration PAT from Secrets Manager at boot, never from the AMI
ACCESS_TOKEN=$(aws secretsmanager get-secret-value \
  --secret-id "${token_arn}" --query SecretString --output text)

# Run the official runner image. A systemd unit (Restart=always) supervises the
# container, so the runner re-registers and comes back on its own after a reboot
# or a spot stop/start.
docker run --name gha-runner \
  -e REPO_URL="https://github.com/${repo_url}" \
  -e ACCESS_TOKEN="$ACCESS_TOKEN" \
  -e RUNNER_SCOPE="repo" \
  -e LABELS="self-hosted,linux,arm64,spot" \
  myoung34/github-runner:latest
```

The runner is scoped to a single repo (`RUNNER_SCOPE=repo`), not the whole org, and it runs inside a container, so a misbehaving job stays boxed in. Want clean state on every job? The image supports an ephemeral mode too.

### Shell access without SSH keys

I do not put SSH keys on these. Ever. The IAM role gives the instance the SSM agent permissions, and I get a shell through Systems Manager Session Manager:

```bash
aws ssm start-session --target i-0abc123def456
```

No port 22 open, no key pairs to rotate, no bastion. Every session is logged in CloudTrail. The security group has zero inbound rules. This alone is worth the migration.

### Surviving spot interruptions

Spot means AWS can reclaim the instance with two minutes' notice. I don't fight that with a fancy poller. Two boring choices do the work:

- The spot request is **persistent** with the interruption behavior set to **stop**, not terminate. When capacity frees up, the same instance starts again instead of being gone for good.
- The runner container is supervised by **systemd with `Restart=always`**, so once the box is back it re-registers and starts pulling jobs on its own.

A build that was mid-flight when the instance got reclaimed gets dropped and re-run. For CI that's almost always fine, and it's a lot less machinery than an Auto Scaling group and a metadata-poller babysitting every node. Runner logs go to CloudWatch, so when something breaks at 2am you have a paper trail instead of a shrug.

### The honest part: what you now own

The savings are real. So are the trade-offs. Self-hosting CI means you signed up for:

- **Patching.** That AMI is yours now. OS updates, agent updates, the runner binary, all on you.
- **Runner security.** Anything your jobs can do, the runner can do. Keep it ephemeral, keep the IAM role tight, keep secrets out of the AMI.
- **Spot interruptions.** Mostly invisible if you design for it, occasionally annoying if you don't.
- **State and ops.** Keep your Terraform state safe (S3 backend with versioning, or at minimum real backups). It's your infrastructure now.

And one hard rule: **never run self-hosted runners on public repos.** A stranger's pull request can run arbitrary code on your runner. That's a footgun pointed straight at your AWS account. Self-hosted is for private and internal org repos only. No exceptions.

### Worth it?

For a busy private monorepo or a team drowning in matrix builds, absolutely. About 70% off CI, hardware you control, and zero SSH keys to babysit. For a tiny side project with three pushes a week, just stay on hosted runners and keep your weekends.

I stopped overpaying for CI. The Terraform took an afternoon. You're welcome.
