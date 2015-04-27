---
layout:     post
title:      "Route53 and ELBs"
subtitle:   "About private zones and private ELBs"
date:       2015-04-27 00:00:00
author:     "viorreta"
header-img: "img/route-53-logo2.png"
---

<p>
Route53 is the DNS service offered by Amazon Web Services. Route53 is composed by hosted zones. Each hosted zone is a collection of resource records sets. 

</p>

<h2>Alias Resource Record Sets</h2>
<p>

Alias resource record sets is a specific extension to standard DNS functionality provided by Route53. Instead of using standard CNAME entries, using Alias Resources is advantageous because:
<ul>
<li>Save time because Route53 automatically recognizes changes in the resource record sets that the alias resource record set refers to. If the IP address of the load balancer changes, Amazon Route 53 will automatically reflect those changes in DNS answers.</li>
<li>They are not counted as chargeable Route 53 requests so they are free.</li>
</ul>
</p>

<p>

Use Alias resources (instead of the usual CNAME entry) to point to resources as:
<ul>
<li>An Elastic Load Balancer</li>
<li>An Amazon S3 bucket that is configured as a static website</li>
<li>Another record in the same DNS zone with Amazon Route 53</li>
<li>An alternate domain name for a CloudFront distribution</li>
</ul>
</p>
<p>
In this example we set up an alias record set responding to an ELB:
{% highlight javascript %}
  "myELB" : {
    "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
    "Properties" : {
      "AvailabilityZones" : [ "us-east-1a" ],
      "Listeners" : [ {
        "LoadBalancerPort" : "80",
        "InstancePort" : "80",
        "Protocol" : "HTTP"
      } ]
    }
  },
  "myDNS" : {
    "Type" : "AWS::Route53::RecordSetGroup",
      "Properties" : {
        "HostedZoneName" : "example.com.",
        "Comment" : "Zone apex alias targeted to myELB LoadBalancer.",
        "RecordSets" : [ {
          "Name" : "example.com.",
          "Type" : "A",
            "AliasTarget" : {
              "HostedZoneId" : { "Fn::GetAtt" : ["myELB", "CanonicalHostedZoneNameID"] },
              "DNSName" : { "Fn::GetAtt" : ["myELB","CanonicalHostedZoneName"] }
            }
          }
        ]
    }
  }
{% endhighlight %}

</p>


<h2>Private zones</h2>
<p>
A private hosted zone is a container that holds information about how you want to route traffic for a domain and its subdomains within one or more Amazon Virtual Private Clouds (Amazon VPCs). Typically we have to use internal ELBs together with private hosted zones to route traffic inside your VPC. To route traffic from internet then we have to use public hosted zones with public ELBs.
</p>	
<p>
For example, imagine that we have one ELB responding to example1.com and another one responding to example2.com. There are requests from internet to both sites but because of dependencies between them we have to route internal traffic between them too. In that case we have to create two private zones for the VPC with two internal ELBs and two public zones with two public ELBs.
</p>
<p>	
In this example we create an internal ELB, a private hosted zone and an alias record set for "example.com": 

{% highlight javascript %}
  "myELBPriv" : {
    "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
    "Properties" : {
      "AvailabilityZones" : [ "us-east-1a" ],
      "Scheme" : "internal",
      "Listeners" : [ {
        "LoadBalancerPort" : "80",
        "InstancePort" : "80",
        "Protocol" : "HTTP"
      } ]
    }
  },
  "DNS": {
    "Type": "AWS::Route53::HostedZone",
    "Properties": {
      "HostedZoneConfig": {
        "Comment": "My hosted zone for example.com"
      },
      "Name": "example.com",
      "VPCs": [{
        "VPCId": "vpc-abcd1234",
        "VPCRegion": "ap-northeast-1"
      }],
      "HostedZoneTags" : [{
        "Key": "SampleKey1",
        "Value": "SampleValue1"
      },
      {
        "Key": "SampleKey2",
        "Value": "SampleValue2"
      }]
    }
  },
  "myDNS" : {
    "Type" : "AWS::Route53::RecordSetGroup",
      "Properties" : {
        "HostedZoneId" : { "Fn::Join" : ["",[ "/hostedzone/", { "Ref" : "DNS"  } ]]},
        "Comment" : "Zone apex alias targeted to myELBPriv LoadBalancer.",
        "RecordSets" : [ {
          "Name" : "example.com.",
          "Type" : "A",
            "AliasTarget" : {
              "HostedZoneId" : { "Fn::GetAtt" : ["myELBPriv", "CanonicalHostedZoneNameID"] },
              "DNSName" : { "Fn::GetAtt" : ["myELBPriv","CanonicalHostedZoneName"] }
            }
          }
        ]
    }
  }
{% endhighlight %}
</p>


