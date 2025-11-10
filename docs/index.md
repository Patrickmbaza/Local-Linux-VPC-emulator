Building a Virtual Private Cloud (VPC) from Scratch on Linux
🚀 Overview

In this project, I built a fully functional Virtual Private Cloud (VPC) system entirely from scratch using Linux networking primitives. This implementation replicates core AWS VPC features including subnets, routing, NAT gateways, VPC isolation, peering, and security groups - all running on a single Linux host.
🎯 What We're Building

    Custom CLI Tool (vpcctl) - Bash-based VPC management

    Network Isolation - Using Linux namespaces

    Routing & NAT - Internal routing and internet access

    VPC Peering - Cross-VPC communication

    Security Groups - JSON-based firewall rules

    Complete Automation - Creation to cleanup

🏗️ Architecture Overview
text

Linux Host
├── VPC A (10.0.0.0/16)
│   ├── Public Subnet (10.0.1.0/24) → Internet via NAT
│   └── Private Subnet (10.0.2.0/24) → Internal only
├── VPC B (172.16.0.0/16)
│   └── Public Subnet (172.16.1.0/24)
└── VPC Peering Connection (Optional)

🛠️ Technical Stack

    Network Namespaces - Subnet isolation

    veth Pairs - Virtual Ethernet connections

    Linux Bridges - Virtual switching

    iptables - Firewall and NAT rules

    Bash Scripting - Automation and CLI

    JSON - Firewall policy configuration

📦 Installation & Setup
Prerequisites
bash

# Ubuntu/Debian
sudo apt update
sudo apt install iproute2 iptables bridge-utils jq python3 curl

# CentOS/RHEL
sudo yum install iproute iptables bridge-utils jq python3 curl

Get the Project
bash

git clone <your-repo-url>
cd Local-linux-VPC
chmod +x vpcctl scripts/*.sh

🎮 CLI Usage Examples
1. Basic VPC Creation
bash

# Create a VPC
sudo ./vpcctl create-vpc my-vpc 10.0.0.0/16

# Create subnets
sudo ./vpcctl create-subnet my-vpc public public 10.0.1.0/24
sudo ./vpcctl create-subnet my-vpc private private 10.0.2.0/24

# List VPCs
sudo ./vpcctl list

2. Deploy Test Workloads
bash

# Deploy web servers in subnets
sudo ./vpcctl deploy-workload my-vpc public web-server 10.0.1.100
sudo ./vpcctl deploy-workload my-vpc private app-server 10.0.2.100

3. Network Testing
bash

# Test connectivity
sudo ./vpcctl test-connectivity subnet-my-vpc-public 10.0.2.100
sudo ./vpcctl test-internet subnet-my-vpc-public

4. Advanced Features
VPC Peering
bash

# Create second VPC
sudo ./vpcctl create-vpc production 172.16.0.0/16
sudo ./vpcctl create-subnet production web public 172.16.1.0/24

# Establish peering
sudo ./vpcctl create-peering my-vpc production

# Verify peering
sudo ./vpcctl list-peerings

Security Groups
bash

# Add firewall rules
sudo ./vpcctl add-firewall-rule 10.0.1.0/24 80 tcp allow
sudo ./vpcctl add-firewall-rule 10.0.1.0/24 22 tcp deny

# List rules
sudo ./vpcctl list-firewall-rules

🔧 How It Works Under the Hood
Network Namespaces (Subnets)

Each subnet is an isolated network namespace:
bash

# Create namespace for subnet
ip netns add subnet-my-vpc-public

# View all namespaces
ip netns list

Virtual Ethernet (veth) Pairs

Connect namespaces to bridges:
bash

# Create veth pair
ip link add veth-host type veth peer name veth-ns

# Move one end to namespace
ip link set veth-ns netns subnet-my-vpc-public

Linux Bridges (VPC Routers)

Each VPC has a bridge acting as router:
bash

# Create bridge
ip link add br-my-vpc type bridge
ip link set br-my-vpc up

NAT Configuration

Internet access for public subnets:
bash

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Configure NAT
iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j MASQUERADE

🧪 Testing & Validation
Test 1: Internal VPC Connectivity
bash

# Deploy workloads
sudo ./vpcctl deploy-workload my-vpc public web1 10.0.1.10
sudo ./vpcctl deploy-workload my-vpc private app1 10.0.2.10

# Test connectivity
sudo ./vpcctl test-connectivity subnet-my-vpc-public 10.0.2.10
# Expected: SUCCESS

Test 2: Internet Access
bash

# Test public subnet internet
sudo ./vpcctl test-internet subnet-my-vpc-public
# Expected: SUCCESS

# Test private subnet internet  
sudo ./vpcctl test-internet subnet-my-vpc-private
# Expected: FAILED (by design)

Test 3: VPC Isolation
bash

# Create second VPC
sudo ./vpcctl create-vpc isolated 192.168.0.0/16
sudo ./vpcctl create-subnet isolated web public 192.168.1.0/24
sudo ./vpcctl deploy-workload isolated web isolated-app 192.168.1.10

# Attempt cross-VPC communication
sudo ./vpcctl test-connectivity subnet-my-vpc-public 192.168.1.10
# Expected: FAILED (isolation working)

Test 4: VPC Peering
bash

# Enable peering
sudo ./vpcctl create-peering my-vpc isolated

# Test after peering
sudo ./vpcctl test-connectivity subnet-my-vpc-public 192.168.1.10
# Expected: SUCCESS

Test 5: Firewall Rules
bash

# Add restrictive rules
sudo ./vpcctl add-firewall-rule 10.0.1.0/24 22 tcp deny

# Test SSH access (should be blocked)
sudo ip netns exec subnet-my-vpc-public nc -zv 10.0.2.10 22
# Expected: FAILED

📊 Demonstration Script

Run the complete test suite:
bash

# Execute full demonstration
sudo ./examples/demo_workloads.sh

# Or run the video demonstration script
sudo ./scripts/demo-video-script.sh

🗑️ Cleanup & Maintenance
Individual Resource Cleanup
bash

# Delete specific VPC
sudo ./vpcctl delete-vpc my-vpc

# Clear firewall rules
sudo ./vpcctl clear-firewall-rules

# Remove peering
sudo ./vpcctl delete-peering my-vpc isolated

Complete Environment Cleanup
bash

# Remove all VPC resources
sudo ./vpcctl cleanup-all

# Or use the cleanup script
sudo ./scripts/cleanup.sh

Manual Verification

After cleanup, verify no resources remain:
bash

# Check for orphaned resources
ip netns list
ip link show type bridge
iptables -L -n | grep -E "vpc|subnet"

🔍 Troubleshooting Common Issues
Permission Issues
bash

# Always run with sudo
sudo ./vpcctl create-vpc demo 10.0.0.0/16

Network Interface Conflicts
bash

# Check existing bridges
ip link show type bridge

# Remove conflicting bridges
ip link delete br-conflicting-name

Firewall Rule Problems
bash

# Reset iptables if needed
iptables -F
iptables -t nat -F

📈 Advanced Usage Examples
Multi-Tier Application Architecture
bash

# Three-tier setup
sudo ./vpcctl create-vpc three-tier 10.10.0.0/16
sudo ./vpcctl create-subnet three-tier web public 10.10.1.0/24
sudo ./vpcctl create-subnet three-tier app private 10.10.2.0/24  
sudo ./vpcctl create-subnet three-tier db private 10.10.3.0/24

# Deploy workloads
sudo ./vpcctl deploy-workload three-tier web frontend 10.10.1.100
sudo ./vpcctl deploy-workload three-tier app backend 10.10.2.100
sudo ./vpcctl deploy-workload three-tier db database 10.10.3.100

Complex Firewall Policies

Create JSON firewall rules:
json

{
  "10.0.1.0": {
    "ingress": [
      {"port": 80, "protocol": "tcp", "action": "allow"},
      {"port": 443, "protocol": "tcp", "action": "allow"},
      {"port": 22, "protocol": "tcp", "action": "deny"}
    ]
  }
}

🎯 Key Learning Outcomes

Through this project, I've demonstrated:

    Deep Linux Networking Understanding - Namespaces, bridges, routing

    Cloud Networking Concepts - VPCs, subnets, NAT, peering

    Security Implementation - Firewalls and isolation

    Automation Skills - Bash scripting and CLI development

    Troubleshooting Ability - Network debugging and validation

📚 Resources & References

    Linux Network Namespaces

    Linux Bridge

    iptables Tutorial

    AWS VPC Concepts

💡 Future Enhancements

Potential improvements for this project:

    Load Balancing - Add HAProxy between tiers

    Monitoring - Integration with Prometheus

    Container Support - Docker network integration

    GUI Interface - Web-based management console

    Kubernetes CNI - Custom Container Network Interface

🏁 Conclusion

This project successfully demonstrates that complex cloud networking concepts can be implemented using basic Linux tools. The vpcctl CLI provides an intuitive interface for managing virtual networks that behave like real cloud VPCs.