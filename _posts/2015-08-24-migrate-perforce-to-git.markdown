---
layout:     post
title:      "Migration from Perforce to Git"
subtitle:   "Migrating Perforce to Git (GitHub) using GitFusion"
date:       2015-08-24 00:00:00
author:     "Daniel Viorreta"
header-img: "img/gitfusion.jpg"
---

<h2>Motivation</h2>
<p>
After almost ten years using Perforce in our company I have decided to move to Git. You can find a lot of posts about the advantages of using Git over other version control system (VCS). In my opinion, the most important advantage is that Git is becoming the default VCS.
</p>


<h2>Git Fusion</h2>
<p>
Perforce launched a very nice tool called Git Fusion. With Git Fusion you can work with your Git client without knowing that there is a Perforce server behind it. When a Git user pushes a change in the repo, GitFusion translates those Git changes into Perforce changes and submits them to the Perforce depot. And when a Perforce user submits a change to the Perforce depot Git Fusion translates those changes to Git depot. 
</p>

<p>
Git Fusion is a powerfull tool that allows you keep using Perforce through it. But, as I only used it to move to Git, I just set the configuration that I needed.
</p>

<p>
My Perforce server was an old machine in our datacenter with RedHat 4 and Git Fusion was only available from RedHat 5. So, I installed a new machine in Amazon Web Services with the rpms of Git Fusion.
</p>

<p>
Once GitFusion is configured, you will find a new Perforce user called <i> git-fusion-user </i>. A new Perforce depot in <i> //.git-fusion </i> will be automatically created too.
</p>

<p> I set my user adding a new file in <i> //.git-fusion/users/daniel.viorreta/keys/keymac </i> with my ssh public key. Then, I created a file like this <i> //.git-fusion/repos/myrepo/p4gf_config </i> for each repo that I wanted to migrate. As I used peforce streams my file looks like this:

{% highlight ini %}
[@repo]
description = Repo for MyRepo

[master]
git-branch-name = master
stream = //myrepo/mainline
original-view = //myrepo/mainline/... ...

[myfeature]
git-branch-name = feature/myfeature
stream = //myrepo/myfeature
original-view = //myrepo/myfeature/... ...
{% endhighlight %}


To clone the Git repo you can do it like a normal Git repo with git clone:

{% highlight bash %}
git clone git@git.gitfusion.zzivi.com:myrepo
{% endhighlight %}

Notice that the first time that you clone the repo Git Fusion will translate all the changes to Git repo, so it can take several minutes or hours depending on the size of your repo.

</p>


<h2>GitHub</h2>
<p>
Once that I had Git Fusion running I wanted to move my Git repo to GitHub. So I cloned my Git Fusion repo with the bare option. After creating a new empty repo in GitHub, I pushed my changes to the new repo adding a new remote to my Git configuration: 

{% highlight bash %}
git clone git@git.gitfusion.zzivi.com:myrepo --bare  #clone repo from Git Fusion
git remote add github git@github.com:zzivi/myrepo.git # add github remote
git push github --mirror # push changes to github
{% endhighlight %}
</p>
<p>
Finally I removed the write permissions in the Perforce depot and started the new challenge of using Git!!
</p>