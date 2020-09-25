# SSHing between EC2 instances

I'd set up three EC2 instances and realized it would be desirable to be able to
ssh between any two. I moved a .pem file into ~/.aws on each and figured

     ubuntu@ip-10-20-30-400:~$ ssh -i ~/.aws/my.pem ubuntu@10.20.30.500

would work. It turns out it didn't -- it would hang -- and I learned the issue
was in the security group used when I set up the instances.

The security group I originally configured had:

*   Inbound rules: SSH traffic from a specific IP. This allowed me to ssh in
    from that remote host.
*   Outbound rules: all traffic.

This resulted in security group sg-0b30...

The inbound rules were the issue. It turns out I needed to add one more:
type=SSH with source sg-0b30... That is, the rule references the security group
itself. After modifying the security group in the AWS console, everything worked
like I wanted it to.

The modification was performed without shutting down or rebooting the instances.
