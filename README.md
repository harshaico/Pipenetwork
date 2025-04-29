# Pipenetwork Node
Pipenetwork
Alright — you want a script to run a Pipenetwork testnet node inside WSL2 (Windows Subsystem for Linux 2).

Let’s go! 🚀

First: Understanding Pipenetwork
Most blockchain testnets (like Pipenetwork) use a Rust-based chain client (like Substrate or a custom node), and typically:

Build from source (Rust toolchain needed)
OR Run from Docker if provided
Need ports like p2p (e.g., 30333), RPC (e.g., 9933), WebSocket (e.g., 9944).
Here's a full bash script to:
Install dependencies (Rust, Docker, etc.)
Build or pull Pipenetwork node
Run it nicely inside WSL2
📜 Full Script: setup-pipenetwork-node.sh
#!/bin/bash

# Script: setup-pipenetwork-node.sh
# Purpose: Install dependencies, set up Rust/Docker, clone Pipenetwork repo, build and run in WSL2

set -e

echo "Updating system..."
sudo apt update && sudo apt upgrade -y

echo "Installing basic packages..."
sudo apt install -y curl git build-essential pkg-config libssl-dev

# --- Install Rust ---
if ! command -v rustup &> /dev/null
then
    echo "Installing Rust toolchain..."
    curl https://sh.rustup.rs -sSf | sh -s -- -y
    source $HOME/.cargo/env
else
    echo "Rust already installed."
fi

# --- Install Docker (optional if using dockerized version) ---
if ! command -v docker &> /dev/null
then
    echo "Installing Docker..."
    sudo apt install -y ca-certificates gnupg lsb-release
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
        sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    echo "Adding user to Docker group..."
    sudo usermod -aG docker $USER
fi

echo "Docker installation complete (if needed)."

# --- Clone Pipenetwork source code ---
echo "Cloning Pipenetwork node repository..."
git clone https://github.com/pipe-network/node.git pipenetwork-node
cd pipenetwork-node

# --- Build the node ---
echo "Building Pipenetwork node..."
cargo build --release

# --- Run the node ---
echo "Starting Pipenetwork testnet node..."
./target/release/pipe-network-node \
    --chain testnet \
    --name "MyPipeNode" \
    --rpc-cors all \
    --rpc-methods Unsafe \
    --ws-external \
    --rpc-external \
    --port 30333 \
    --rpc-port 9933 \
    --ws-port 9944
✅ What this script does:
Step	Purpose
Update apt	System prep
Install Rust	Needed for building node
Install Docker (optional)	If running dockerized node
Clone Pipenetwork source	From GitHub
Build node	cargo build --release
Run node	Start testnet node with proper ports
🛠 Requirements:
Ubuntu distro on WSL2
Enough RAM (Rust build needs 2-4 GB)
Disk space (building can take ~1GB)
Git access
🔥 Bonus: Quick Tips
Build faster:
If your machine is powerful, you can compile with multiple cores:

cargo build --release --jobs $(nproc)
Auto-run on WSL boot:
Add this to .bashrc:

cd ~/pipenetwork-node && ./target/release/pipe-network-node --chain testnet --name "MyPipeNode"
📦 Alternate: Docker version?
If Pipenetwork provides a Docker image, you could skip Rust and just run:

docker run -d \
  --name pipe-node \
  -p 30333:30333 \
  -p 9933:9933 \
  -p 9944:9944 \
  pipenetwork/node:latest
(If you want, I can script this too.)

Would you like me to also prepare a Docker Compose version if you prefer a more production-like setup? 🚀
Just say "yes docker-compose" if so!
