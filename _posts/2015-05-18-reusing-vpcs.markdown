---
layout:     post
title:      "Reusing VPCs between stacks"
subtitle:   "About reusing vpcs, security groups and route53 zones when deploying new stacks"
date:       2015-05-18 00:00:00
author:     "viorreta"
header-img: "img/post-bg-06.jpg"
---

<p>
Amazon VPCs lets create your own isolated section in AWS in your own virtual network. 
</p>

<h2>The problem</h2>
<p>
I have only one VPC and its subnets created in my production environment. Then I have different stacks with my production services that use them. A new deployment of my application consists in deploying new stacks using the VPC deployed previously and delete the old stacks without modifying the VPC. The problem is how to identify the VPC where I want to deploy my new stacks without deploying a new VPC each time that I have a new version of my code. Similar for security groups and route53 private zones.
</p>

<h2>Tagging VPCs and subnets</h2>
<p>
To identify my VPC and my subnet I use a label to tag them. This is a part of a cloudformation where I tag the VPC and one of its subnets. Assume than we pass labelstack as a cloudformation parameter:
{% highlight javascript %}
...
    "MyVPC" : {
      "Type" : "AWS::EC2::VPC",
      "Properties" : {
        "CidrBlock" : { "Ref" : "CIDRVPC" },
        "EnableDnsSupport" : "true",
        "EnableDnsHostnames" : "true",
        "Tags": [ {"Key" : "Name", "Value" : { "Ref" : "labelstack" }} ]
      }
    },
    "MySubnet1" : {
      "Type" : "AWS::EC2::Subnet",
      "Properties" : {
        "AvailabilityZone" : { "Ref" : "Subnet1AZ" },
        "CidrBlock" : { "Ref" : "CIDRSubnet1" },
        "VpcId" : { "Ref" : "MyVPC" },
        "Tags": [ {"Key" : "Name", "Value" : { "Fn::Join" : ["-", [ { "Ref" : "labelstack" }, "MySubnet1" ]]}} ]
      }
    },
...
{% endhighlight %}

Note that we can do something similar to security groups.

</p>

<h2>Getting VPCs, subnets and route53 zones by tag name</h2>
Then, via boto I can get the vpcs and subnet using the labelstack to identify them. Similar for security groups and route53 private zones.

This is the 'api' that our class will have to implement:
{% highlight python %}
class IManageVpc:
    """ Class to manage vpcs and security groups reuse
    """
    def get_vpc(self, vpc_name):
        """ Get the vpc by name
        """
        raise NotImplementedError( "Should have implemented this" )

    def get_subnets_by_vpc_and_name(self, vpc, subnet_name):
        """ Get the subnet by vpc name and by subnet name
        """
        raise NotImplementedError( "Should have implemented this" ) 

    def get_security_groups_by_vpc_and_name(self, vpc, sg_name):
        """ Get the security group by vpc name and by security group name
        """
        raise NotImplementedError( "Should have implemented this" )

    def get_zone_by_vpc_and_name(self, vpc_id, zone_name):
        """ Get the zone by vpc and zone name
        """
        raise NotImplementedError( "Should have implemented this" )
{% endhighlight %}

And this is an implementation of it:


{% highlight python %}
import boto.vpc
import boto.ec2
import boto.route53
import imanagevpc
import time


REGION='eu-west-1'

class ManageVpc(imanagevpc.IManageVpc):

    def __init__(self, ec2conn, profile_name):
        self.ec2conn = ec2conn
        self.profile_name = profile_name
        self.vpcconn = self.__connect_vpc()
        self.route53conn = self.connect_route53()

    def connect_vpc(self):
        region = boto.ec2.get_region(REGION)
        conn = boto.vpc.VPCConnection(region=region, profile_name=self.profile_name)
        return conn

    def connect_route53(self):
        conn = boto.route53.connect_to_region(REGION, profile_name=self.profile_name)
        return conn

    def filter_by_name(self, function, name):
        filters = {'tag:Name': name}
        return function(filters=filters)

    def filter_by_name_and_vpc(self, function, vpc, name):
        filters = {'vpc_id': vpc.id, 'tag:Name': name}
        return function(filters=filters)

    def get_vpc(self, vpc_name):
        for vpc in self.filter_by_name(self.vpcconn.get_all_vpcs, vpc_name):
            return vpc
        raise ValueError("A VPC for the given vpc name doesn't exist")

    def get_subnets_by_vpc_and_name(self, vpc, subnet_name):
        for subnet in self.filter_by_name_and_vpc(self.vpcconn.get_all_subnets, vpc, subnet_name):
            return subnet
        raise ValueError("A Subnet in the VPC doesn't exist")    

    def get_security_groups_by_vpc_and_name(self, vpc, sg_name):
        for security_group in self.filter_by_name_and_vpc(self.vpcconn.get_all_security_groups, vpc, sg_name):
            return security_group
        raise ValueError("A Segurity Group for the VPC doesn't exist")

    def get_zone_by_vpc_and_name(self, vpc_id, zone_name):
        for zone in self.route53conn.get_zones():
            if zone.name == zone_name:
                try:
                    if self.route53conn.get_hosted_zone(zone.id)['GetHostedZoneResponse']['VPCs']['VPC']['VPCId'] == vpc_id:
                        valueToReturn = {}
                        valueToReturn["zoneName"] = zone.name
                        valueToReturn["zoneId"] = zone.id
                        return valueToReturn
                except boto.route53.exception.DNSServerError as e:
                    print "Error getting route53 zones. Retrying it...:"
                    print e
                    time.sleep(20)


{% endhighlight %}

Now, each time that I deploy a new version on my application a pass the parameter whith the label where I want to attach the stacks.

