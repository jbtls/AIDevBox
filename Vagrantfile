# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 2.2.0"

# ============================================================================
# USER CONFIGURATION
# ============================================================================
# Set which tools to install. Options: "opencode", "copilot", or both
INSTALL = ["opencode", "copilot"]

# ============================================================================
# ENVIRONMENT LOADER
# ============================================================================
env_file = File.expand_path(".env", File.dirname(__FILE__))

gh_token = nil
openrouter_key = nil

if File.exist?(env_file)
  File.foreach(env_file) do |line|
    line.strip!
    next if line.empty? || line.start_with?("#")

    # Parse GH_TOKEN
    if line =~ /^(?:export\s+)?GH_TOKEN\s*=\s*(.+)$/
      value = $1.strip.sub(/\s+#.*$/, "")
      if (value.start_with?('"') && value.end_with?('"')) ||
         (value.start_with?("'") && value.end_with?("'"))
        value = value[1..-2]
      end
      gh_token = value
    end

    # Parse OPENROUTER_API_KEY
    if line =~ /^(?:export\s+)?OPENROUTER_API_KEY\s*=\s*(.+)$/
      value = $1.strip.sub(/\s+#.*$/, "")
      if (value.start_with?('"') && value.end_with?('"')) ||
         (value.start_with?("'") && value.end_with?("'"))
        value = value[1..-2]
      end
      openrouter_key = value
    end
  end
end

# Fallback to environment variables
gh_token ||= ENV["GH_TOKEN"]
openrouter_key ||= ENV["OPENROUTER_API_KEY"]

# ============================================================================
# VALIDATION
# ============================================================================
install_copilot = INSTALL.include?("copilot")
install_opencode = INSTALL.include?("opencode")

if install_copilot && (gh_token.nil? || gh_token.empty?)
  abort "GH_TOKEN is required when installing Copilot CLI.\n" \
        "Add GH_TOKEN=<token> to .env or export it before vagrant up"
end

if install_opencode && (openrouter_key.nil? || openrouter_key.empty?)
  puts "WARNING: OPENROUTER_API_KEY not found. OpenCode will run without authentication."
  puts "         You can run '/connect' inside OpenCode to add keys later."
end

# ============================================================================
# VAGRANT CONFIGURATION
# ============================================================================
Vagrant.configure("2") do |config|
  config.vm.box      = "ubuntu/jammy64"
  config.vm.hostname = "dev-env"

  # vagrant-vbguest 0.32.0 crashes on Ruby 3.2+ (File.exists? removed)
  config.vbguest.auto_update = false if Vagrant.has_plugin?("vagrant-vbguest")

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 4096
    vb.cpus   = 4
  end

  config.vm.synced_folder "./project", "/project"

  # --- Base provisioning (root) ---
  config.vm.provision "shell", privileged: true, inline: <<~'SHELL'
    set -euo pipefail
    export DEBIAN_FRONTEND=noninteractive

    apt-get update -qq
    apt-get install -y -qq curl ca-certificates
  SHELL

  # --- Copilot provisioning (root) ---
  if install_copilot
    config.vm.provision "shell", privileged: true, inline: <<~'SHELL'
      set -euo pipefail

      # Install GitHub CLI if missing
      if ! command -v gh >/dev/null 2>&1; then
        curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
          | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg

        chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg

        echo "deb [arch=$(dpkg --print-architecture) \
          signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] \
          https://cli.github.com/packages stable main" \
          > /etc/apt/sources.list.d/github-cli.list

        apt-get update -qq
        apt-get install -y -qq gh
      fi

    SHELL
  end

  # --- User provisioning ---
  config.vm.provision "shell",
    privileged: false,
    env: { 
      "GH_TOKEN" => gh_token || "",
      "OPENROUTER_KEY" => openrouter_key || "",
      "INSTALL_COPILOT" => install_copilot ? "1" : "0",
      "INSTALL_OPENCODE" => install_opencode ? "1" : "0"
    },
    inline: <<~'SHELL'
      set -euo pipefail

      # --- Copilot setup ---
      if [ "$INSTALL_COPILOT" = "1" ]; then
        # Ensure ~/.local/bin is in PATH for this session
        export PATH="$HOME/.local/bin:$PATH"

        # Install Copilot CLI if missing (user-scoped, non-interactive)
        if ! command -v copilot >/dev/null 2>&1; then
          curl -fsSL https://gh.io/copilot-install -o /tmp/copilot-install.sh
          echo "y" | bash /tmp/copilot-install.sh
          rm -f /tmp/copilot-install.sh
        fi

        # Authenticate GitHub CLI — always re-auth so token changes in .env take effect
        if echo "$GH_TOKEN" | gh auth login --with-token; then
          echo "✓ GitHub CLI authenticated"
        else
          echo "ERROR: gh auth login failed — GH_TOKEN may be invalid or missing Copilot scope."
          echo "       Update GH_TOKEN in .env and re-run 'vagrant provision'."
        fi

        # Persist PATH only — do NOT export GH_TOKEN to avoid overriding stored gh credentials
        # shellcheck disable=SC2016
        grep -qxF 'export PATH="$HOME/.local/bin:$PATH"' ~/.bashrc || \
          echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
      fi

      # --- OpenCode setup ---
      if [ "$INSTALL_OPENCODE" = "1" ]; then
        # Install OpenCode if missing (user-scoped)
        if ! command -v opencode >/dev/null 2>&1; then
          curl -fsSL https://opencode.ai/install | bash
        fi

        # Ensure path
        # shellcheck disable=SC2016
        grep -qxF 'export PATH="$HOME/.opencode/bin:$PATH"' ~/.bashrc || \
          echo 'export PATH="$HOME/.opencode/bin:$PATH"' >> ~/.bashrc

        mkdir -p ~/.local/share/opencode

        # Write auth.json if key is present
        if [ -n "$OPENROUTER_KEY" ]; then
          cat > ~/.local/share/opencode/auth.json <<EOF
{
  "openrouter": {
    "type": "api",
    "key": "$OPENROUTER_KEY"
  }
}
EOF
          chmod 600 ~/.local/share/opencode/auth.json
        fi

        # Create minimal project config (only if it doesn't already exist)
        mkdir -p /project
        if [ ! -f /project/opencode.json ]; then
          cat > /project/opencode.json <<'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {}
    }
  }
}
EOF
        fi
      fi

      # --- Shell hints ---
      if ! grep -q "dev-env hint" ~/.bashrc; then
        cat >> ~/.bashrc <<'EOF'

# dev-env hint
if [[ $- == *i* ]]; then
  cd /project
  echo "============================================"
  echo " Development environment ready"
  echo "============================================"
EOF

        if [ "$INSTALL_OPENCODE" = "1" ] && [ "$INSTALL_COPILOT" = "1" ]; then
          cat >> ~/.bashrc <<'EOF'
  echo "Available tools: OpenCode, Copilot CLI"
  echo "Run 'opencode' or 'copilot' to start"
  echo "============================================"
fi
EOF
        elif [ "$INSTALL_OPENCODE" = "1" ]; then
          cat >> ~/.bashrc <<'EOF'
  echo "Available tool: OpenCode"
  echo "Run 'opencode' to start"
  echo "Run '/connect' to add API keys"
  echo "============================================"
fi
EOF
        elif [ "$INSTALL_COPILOT" = "1" ]; then
          cat >> ~/.bashrc <<'EOF'
  echo "Available tool: Copilot CLI"
  echo "Run 'copilot' to start"
  echo "============================================"
fi
EOF
        fi
      fi
    SHELL
end
