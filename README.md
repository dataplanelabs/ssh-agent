Here’s a rewritten and streamlined version of your README:

---

# `ssh-agent` GitHub Action

This action simplifies SSH key management in GitHub workflows by:  
- Starting the `ssh-agent`.  
- Exporting the `SSH_AUTH_SOCK` environment variable.  
- Loading one or more private SSH keys into the agent.  

It works across all GitHub Actions virtual environments, including container-based workflows. However, support for Windows and Docker is newer and may have edge cases. If these environments work well for you, consider leaving feedback [here](https://github.com/webfactory/ssh-agent/pull/17).  

This action also supports multiple GitHub deployment keys, mapping them to repositories via SSH key comments.

## Why Use This Action?

GitHub Actions have access only to the repository they run for. If your workflow requires access to private repositories, you can:  
1. Create an SSH key with the necessary permissions.  
2. Use this action to load the key into the `ssh-agent`.  

With this setup, `git clone` commands using SSH URLs will work seamlessly, and other `ssh` operations will also utilize the key.

---

## How to Use

### Step 1: Set Up Your SSH Key  
1. Generate a new SSH key with the required access. Avoid using personal keys—create one dedicated to GitHub Actions.  
2. Ensure the private key has no passphrase.  
3. Add the public key to the target private repository as a "Deploy Key".  
4. Save the private key as a GitHub secret:  
   - Go to your repository's **Settings > Secrets**.  
   - Add a new secret, e.g., `SSH_PRIVATE_KEY`, and paste the private key contents.  

### Step 2: Add the Action to Your Workflow  
Add the following to your workflow file:  

```yaml
jobs:
  my_job:
    steps:
      - uses: actions/checkout@v4
      - uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

To customize the SSH agent socket path, use the `ssh-auth-sock` input.

---

## Using Multiple Keys  

You can load multiple SSH keys by passing them as secrets:  

```yaml
- uses: webfactory/ssh-agent@v0.9.0
  with:
    ssh-private-key: |
      ${{ secrets.FIRST_KEY }}
      ${{ secrets.SECOND_KEY }}
```

> **Note:** SSH servers may abort after trying a certain number of invalid keys. Use GitHub deployment keys with comments to ensure the right key is used.

---

## Deployment Key Support  

To use GitHub deployment keys effectively:  
1. Add the repository URL to the key comment when creating it, e.g., `ssh-keygen ... -C "git@github.com:owner/repo.git"`.  
2. The action scans key comments and sets up custom Git and SSH configurations for seamless repository access.

---

## Inputs  

- `ssh-private-key` (**required**): Private SSH keys as secrets.  
- `ssh-auth-sock`: Custom path for the SSH agent socket.  
- `log-public-key`: Defaults to `true`. Set to `false` to suppress public key logging.  
- `ssh-agent-cmd`, `ssh-add-cmd`, `git-cmd`: Optional paths for custom binaries.

---

## Exported Variables  

- `SSH_AUTH_SOCK`: Path to the agent socket.  
- `SSH_AGENT_PID`: Process ID of the agent.  

---

## Known Limitations  

- **Job-specific:** Keys are only available in the job where this action is used.  
- **Key format:** Keys must be in PEM format. Convert using:  
  ```bash
  ssh-keygen -p -f path/to/key -m pem
  ```

---

## Special Use Cases  

- **Container Workflows:** Ensure SSH tools are installed in your container.  
- **Docker Builds:** Pass the agent socket with:  
  ```yaml
  ssh: default=${{ env.SSH_AUTH_SOCK }}
  ```
- **Windows/Rust/Cargo:** Enable `git-fetch-with-cli` for private dependencies.  

---

## Licensing  

Developed by webfactory GmbH, Bonn, Germany. Released under the [MIT license](LICENSE).  

For more details, visit [webfactory](https://www.webfactory.de) or follow us on [Twitter](https://twitter.com/webfactory).
