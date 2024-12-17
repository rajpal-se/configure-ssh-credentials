# Configure SSH Credentials GitHub Action

## Overview

The **Configure SSH Credentials** GitHub Action enables users to set up SSH credentials dynamically within their workflows. This action simplifies SSH configuration management by generating, resetting, and validating SSH keys while providing an easy way to run commands over SSH. The configuration is stored securely under a temporary `.tmp/.ssh` directory and supports customizable options for hostname, user, and key management.

## Features

- **Dynamic SSH Key Configuration**: Generate or reset SSH keys dynamically within a workflow.
- **Customizable Options**: Flexible inputs for hostname, port, user, and private key filename.
- **Command Execution**: Supports running commands over SSH after configuration.
- **Secure File Handling**: Ensures that the SSH credentials are securely stored and ignored via `.dockerignore`.
- **Environment Variable Outputs**: Exports file paths and SSH commands for further usage in workflows.

---

## Inputs

| Name        | Description                                | Required | Default          |
| ----------- | ------------------------------------------ | -------- | ---------------- |
| `host`      | Alias or nickname for the SSH connection   | No       | `${{ user }}`    |
| `user`      | The SSH user                               | No       | `github-actions` |
| `hostname`  | The SSH host IP                            | No       |                  |
| `key`       | The private SSH key                        | No       |                  |
| `filename`  | Filename for the SSH key                   | No       | `${{ user }}`    |
| `port`      | SSH server port                            | No       | `22`             |
| `log-level` | Log level setting                          | No       | `QUIET`          |
| `run`       | Command to run                             | No       |                  |
| `reset`     | Remove and regenerate the private key file | No       | `false`          |

---

## Outputs

| Name               | Description                            |
| ------------------ | -------------------------------------- |
| `ssh-config-file`  | Path to the generated SSH config file  |
| `private-key-file` | Path to the generated private key file |
| `ssh`              | Pre-configured SSH command alias       |

---

## Example Usage

### Basic Setup

The following example demonstrates how to configure SSH credentials with a specified hostname and private key:

```yaml
jobs:
  setup-ssh:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Configure SSH
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.SSH_PRIVATE_KEY }}"
          host: "ec2-dev"

      - name: Test SSH Connection
        run: |
          ssh -F .tmp/.ssh/config ec2-dev "echo 'Connected successfully!'"
```

### Running Commands

To execute commands on the remote server after setting up the SSH configuration:

```yaml
jobs:
  execute-remote-command:
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH and Run Command
        id: configure_ssh
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.SSH_PRIVATE_KEY }}"
          host: "ec2-dev"
          run: "ls -la /home/user"

      - name: Run Commands
        uses: ./
        with:
          run: |
            ${{ steps.configure_ssh.outputs.ssh }} << 'EOF'
              // cmds ...
            EOF
```

### Resetting the Key File

To regenerate the SSH key file, set `reset` to `true`:

```yaml
jobs:
  reset-ssh-key:
    runs-on: ubuntu-latest
    steps:
      - name: Reset SSH Configuration
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.SSH_PRIVATE_KEY }}"
          host: "ec2-dev"
          reset: "true"

      - name: Verify SSH Connection
        run: |
          ssh -F .tmp/.ssh/config ec2-dev "echo 'Regenerated key and connected successfully!'"
```

---

## Use Cases

### Continuous Deployment

Utilize this action for secure deployment pipelines where SSH is required for accessing remote servers:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure SSH for Deployment
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.DEPLOY_KEY }}"
          host: "ec2-deploy"

      - name: Deploy Application
        run: |
          ssh -F .tmp/.ssh/config ec2-deploy "bash /home/user/deploy.sh"
```

### Automated Backups

Set up secure backups from a remote server:

```yaml
jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH for Backup
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.BACKUP_KEY }}"
          host: "backup-server"

      - name: Perform Backup
        run: |
          rsync -e "ssh -F .tmp/.ssh/config" -avz /local/path backup-server:/remote/path
```

---

## Debugging and Logs

Set `log-level` to `DEBUG` for verbose output:

```yaml
jobs:
  debug-ssh:
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH with Debug Logging
        uses: ./
        with:
          hostname: "192.168.**.***"
          key: "${{ secrets.DEBUG_KEY }}"
          host: "debug-host"
          log-level: "DEBUG"

      - name: Debug SSH Connection
        run: |
          ssh -F .tmp/.ssh/config debug-host "echo 'Debugging connection'"
```

---

## Security Considerations

- Use GitHub Secrets to store private SSH keys and sensitive information securely.
- Always ensure `.tmp/.ssh` is ignored in `.dockerignore` or `.gitignore` to avoid accidental exposure of sensitive files.
- Validate your workflow steps to prevent exposing sensitive data.
- Regularly rotate your SSH keys to maintain security.

---

## Contribution

Contributions to this action are welcome! Please fork the repository, make your changes, and submit a pull request.

---

## License

This project is licensed under the MIT License.

<!-- See the [LICENSE](LICENSE) file for details. -->
