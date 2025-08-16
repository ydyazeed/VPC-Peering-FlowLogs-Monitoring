# My Journey with AWS VPC Peering and Flow Logs

In this project, I explored setting up VPC peering between two VPCs and monitoring network traffic using VPC Flow Logs in AWS. Here's my experience and the challenges I faced along the way.

## Architecture Overview

Here's the architecture I built:

![Network Architecture](https://github.com/ydyazeed/VPC-Peering-FlowLogs-Monitoring/raw/main/images/network_diagram.png)

The diagram shows:
- Two VPCs (NextWork VPC 1 and 2) with their own public subnets
- Each VPC has its own Internet Gateway, Route Table, and Network ACL
- VPC Peering Connection enabling direct communication
- CloudWatch integration for Flow Logs monitoring
- EC2 instances in each VPC's public subnet

## What I Built

I created a setup where:
- Two VPCs can communicate directly using private IP addresses
- Network traffic is monitored using VPC Flow Logs
- EC2 instances in each VPC can ping each other (but only using private IPs!)

## The Journey

### Step 1: Creating My VPCs

First, I needed to create two VPCs. Here's how I did it:

#### Setting up VPC 1
1. Logged into my AWS Account and headed to the VPC console
2. Created a VPC with these settings:
   - Named it NextWork-1
   - Set CIDR block to 10.1.0.0/16 (this is important - using the default 10.0.0.0/16 caused conflicts later!)
   - Used 1 Availability Zone and 1 public subnet
   - Skipped private subnets to keep things simple
   - Avoided NAT gateways to save costs 💰

#### Setting up VPC 2
For my second VPC, I used the same settings but with two key differences:
- Named it NextWork-2
- Used CIDR 10.2.0.0/16

**Learning Point:** Initially, I tried using the same CIDR block for both VPCs. AWS quickly let me know that was a no-go! VPCs that will be peered must have non-overlapping CIDR blocks.

### Step 2: Launching EC2 Instances

I needed an instance in each VPC to test the peering connection.

#### Instance in VPC 1
1. Launched an Amazon Linux 2023 instance (t2.micro - staying in free tier!)
2. Named it "Instance - NextWork VPC 1"
3. Created a security group "NextWork-1-SG"
   - Added ICMP rule to allow pings (this was crucial for testing!)

#### Instance in VPC 2
Set up similarly to Instance 1, but in VPC 2.

**Challenge Faced:** Initially, I couldn't ping between instances. After some head-scratching, I realized I needed to allow ICMP traffic in the security groups of both instances!

### Step 3: Setting Up Flow Logs

This is where things got interesting (and a bit tricky).

1. Created a CloudWatch log group named "NextWorkVPCFlowLogsGroup"
2. Tried to set up Flow Logs but hit my first major roadblock - IAM permissions!

#### The IAM Permission Dance 🕺

Here's what I learned the hard way - Flow Logs need specific permissions. I had to:

1. Create an IAM Policy:
\`\`\`json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}
\`\`\`

2. Create an IAM Role with a trust relationship:
\`\`\`json
{
  "Principal": {
    "Service": "vpc-flow-logs.amazonaws.com"
  }
}
\`\`\`

**Gotcha #1:** Initially, I forgot to set up the trust relationship correctly. The Flow Logs kept failing silently until I figured this out!

### Step 4: The VPC Peering Adventure

This was the most interesting part! Here's what I did:

1. Created a peering connection named "VPC 1 <> VPC 2"
2. Set up the route tables (this is where the magic happens!)

#### Route Table Configuration
In VPC 1's route table:
- Added route to VPC 2's CIDR (10.2.0.0/16)
- Set target to the peering connection

Did the same for VPC 2's route table, but pointing to VPC 1's CIDR.

**The Big "Aha!" Moment:** When I first tried to ping between instances, I used their public IPs and got timeouts. This was actually correct behavior! VPC peering only works with private IPs. When I switched to using private IPs, everything worked perfectly.

### Step 5: Monitoring the Traffic

Now came the fun part - watching the traffic flow! Here's what I saw in CloudWatch:

![Flow Logs Analysis](https://github.com/ydyazeed/VPC-Peering-FlowLogs-Monitoring/raw/main/images/flow_logs.png)

The Flow Logs revealed some interesting insights:
1. Successful communication between VPCs (notice the traffic between 10.1.5.112 and 10.2.15.210)
2. Various external IPs trying to reach our instances
3. Different byte transfer sizes for different types of traffic

Looking at the logs, I could see:
- Large transfers (27,779,598 bytes) from external sources
- Smaller transfers (48,720 bytes) between our VPC instances
- Regular communication patterns showing our ping tests

**Cool Discovery:** I could see exactly how many bytes were transferred in each ping! For example, one log showed 344 bytes sent from 18.237.140.165 to 10.1.5.112 using TCP on port 22.

## Common Issues I Faced (and How I Solved Them)

1. **Request Timeouts with Public IPs**
   - Problem: Couldn't ping between instances using public IPs
   - Solution: This was actually expected! VPC peering only works with private IPs
   - Learning: Always use private IPs for communication between peered VPCs

2. **Flow Logs Not Appearing**
   - Problem: Set up Flow Logs but no data appeared
   - Solution: Fixed the IAM role's trust relationship
   - Tip: Always double-check IAM permissions when logs are missing

3. **Route Table Confusion**
   - Problem: Instances couldn't communicate even with peering set up
   - Solution: Had to add routes in BOTH VPCs' route tables
   - Remember: Peering needs bi-directional route configuration

## What I Learned

1. VPC peering is powerful but has specific rules (like no overlapping CIDRs)
2. Always use private IPs for peered VPC communication
3. IAM roles and permissions are crucial - get them right first
4. Flow Logs are amazing for troubleshooting network issues

## Want to Try This Yourself?

Feel free to follow my steps above. If you run into issues or have questions, open an issue in this repo. I'd love to help others learn from my experience!

## License
This project is licensed under the MIT License - see the LICENSE file for details.