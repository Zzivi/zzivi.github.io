---
layout:     post
title:      "Deleting old AMIs and snapshots"
subtitle:   "About deleting old AMIs by date and orphaned snapshots"
date:       2015-05-20 00:00:00
author:     "Daniel Viorreta"
header-img: "img/post-bg-06.jpg"
---

<h2>Deleting AMIs by date</h2>
<p>
When you are a heavy use of Amazon you can have a lot of Amazon Machine Images (AMIs). I have created a boto function to delete AMIs by date:

{% highlight python %}
def delete_old_amis(awsAccount, vRetentionTime=68, vdryRun=False):

    ec2conn = boto.ec2.connect_to_region("eu-west-1", profile_name=awsAccount)
 
    images = ec2conn.get_all_images(owners=['self'])

    count = 0
    for image in images:
        creation_date = image.creationDate.split('T')[0].split('-')
        split_time = (date.today() - date(int(creation_date[0]), int(creation_date[1]), int(creation_date[2]))).days
        if split_time > vRetentionTime:
            print image.id+ " : is going to be deregistered"
            print "Split_time is : "+str(split_time)
            if vdryRun == True:
               image.deregister(delete_snapshot=True)
            count = count + 1
    print 'AMIs deleted: ' + str(count)
{% endhighlight %}



</p>

<h2>Deleting orphaned snapshots</h2>

When you create a new AMI, AWS creates a S3 snapshot of all of the instance’s EBS volumes. But when you delete an AMI the snapshots are left behind. I have created a boto function to delete snapshot not associated with an existing AMI:

{% highlight python %}
def delete_orphan_snapshots(awsAccount, vdryRun=False):
    ec2conn = boto.ec2.connect_to_region("eu-west-1", profile_name=awsAccount)

    imageIds = []
    for image in ec2conn.get_all_images(owners=['self']):
        #print image.id, image.name + ", " + image.description
        #imageIds, imageNames = zip(*[(image.id, image.id)])
        imageIds.append(image.id)

    print 'Number of AMIs: '+str(len(imageIds))

    # Get list of snapshots and AMIs
    reAmi = re.compile('ami-[^ ]+')
    snapshots = []
    snapshotsToDelete = []
    snapshotsUnknown = []
    imageSnapshots = {}

    for snapshot in ec2conn.get_all_snapshots(owner='self'):
        # Get id and image ID via regex.
        snapshotId = snapshot.id
        snapshotImageId = reAmi.findall(snapshot.description)
        if len(snapshotImageId) != 1:
            snapshotsUnknown.append(snapshotId)
        else:
            snapshotImageId = snapshotImageId[0]
            
        # Update lists
        snapshots.append(snapshotId)
        #print 'snapshotImageId: '+str(snapshotImageId)
        if snapshotImageId not in imageIds:
            snapshotsToDelete.append(snapshotId)
        else:
            if snapshotImageId in imageSnapshots:
                imageSnapshots[snapshotImageId].append(snapshotId)
            else:
                imageSnapshots[snapshotImageId] = [snapshotId]
    
    print 'Number of orphaned snapshots to delete: '+str(len(snapshotsToDelete))
    print 'Number of mapped snapshots to keep: '+str(len(imageSnapshots))
    
    if vdryRun == True:
        for snapshot in snapshotsToDelete:
            print 'Removing ' + snapshot + '...'
            ec2conn.delete_snapshot(snapshot)
{% endhighlight %}



