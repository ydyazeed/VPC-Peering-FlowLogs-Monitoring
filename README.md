# My Journey with AWS VPC Peering and Flow Logs

In this project, I explored setting up VPC peering between two VPCs and monitoring network traffic using VPC Flow Logs in AWS. Here's my experience and the challenges I faced along the way.

## Architecture Overview

Here's the architecture I built:

![Network Architecture](https://raw.githubusercontent.com/ydyazeed/VPC-Peering-FlowLogs-Monitoring/main/images/network_diagram.png)

The diagram shows:
- Two VPCs (NextWork VPC 1 and 2) with their own public subnets
- Each VPC has its own Internet Gateway, Route Table, and Network ACL
- VPC Peering Connection enabling direct communication
- CloudWatch integration for Flow Logs monitoring
- EC2 instances in each VPC's public subnet

[Rest of the content remains the same until the Flow Logs image]

### Step 5: Monitoring the Traffic

Now came the fun part - watching the traffic flow! Here's what I saw in CloudWatch:

![Flow Logs Analysis](https://raw.githubusercontent.com/ydyazeed/VPC-Peering-FlowLogs-Monitoring/main/images/flow_logs.png)

[Rest of the content remains exactly the same]