# SSH Agent Setup Workflow

This workflow demonstrates how to set up SSH agent with an ed25519 SSH key in GitHub Actions.

## Overview

The workflow performs the following steps:
1. Starts the SSH agent using `eval "$(ssh-agent -s)"`
2. Creates the `.ssh` directory with proper permissions
3. Adds the SSH private key from GitHub Secrets to `~/.ssh/id_ed25519`
4. Loads the key into the SSH agent using `ssh-add`
5. Configures SSH client settings

## Setup Instructions

### 1. Generate an ed25519 SSH Key (if you don't have one)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519
```

### 2. Add the SSH Private Key to GitHub Secrets

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `SSH_PRIVATE_KEY`
5. Value: Copy the entire contents of your private key file (`~/.ssh/id_ed25519`)
   ```bash
   cat ~/.ssh/id_ed25519
   ```
6. Click **Add secret**

### 3. Add the SSH Public Key to Your Target Server

Add the public key to the target server's `~/.ssh/authorized_keys`:

```bash
cat ~/.ssh/id_ed25519.pub
# Copy this and add it to the target server
```

## Usage

The workflow can be triggered in two ways:

1. **Manual trigger**: Go to Actions tab → SSH Agent Setup Example → Run workflow
2. **Automatic trigger**: Push to `main` or `master` branch

## Security Notes

⚠️ **Important Security Considerations:**

- Never commit private keys to the repository
- Always store private keys in GitHub Secrets
- The workflow disables strict host key checking for demonstration purposes. In production:
  - Enable `StrictHostKeyChecking yes`
  - Add known host keys explicitly
  - Use the `ssh-keyscan` command to add host keys

### Production-Ready Known Hosts Setup

For production use, replace the SSH config step with:

```yaml
- name: Setup SSH with Known Hosts
  env:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  run: |
    eval "$(ssh-agent -s)"
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
    chmod 600 ~/.ssh/id_ed25519
    ssh-add ~/.ssh/id_ed25519
    
    # Add known hosts
    ssh-keyscan -H github.com >> ~/.ssh/known_hosts
    ssh-keyscan -H your-server.com >> ~/.ssh/known_hosts
```

## Common Use Cases

This SSH agent setup is useful for:

- Cloning private repositories
- Deploying to remote servers via SSH
- Pushing commits using SSH authentication
- Running remote commands on servers
- Syncing files with `rsync` over SSH

## Troubleshooting

### "Permission denied (publickey)"

- Ensure the private key is correctly added to GitHub Secrets
- Verify the public key is added to the target server's authorized_keys
- Check that file permissions are correct (600 for private key)

### "Could not open a connection to your authentication agent"

- The workflow starts ssh-agent with `eval "$(ssh-agent -s)"`
- Ensure this command runs before any ssh-add commands

### "No identities loaded in ssh-agent"

- Verify the SSH_PRIVATE_KEY secret is set and not empty
- Check the private key format is correct (PEM format for ed25519)

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [SSH Key Generation Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [OpenSSH Manual](https://www.openssh.com/manual.html)
