---
title: "Getting a verified user on Github"
description: "This is a step by step example what you have to do, to get a verified user on Github"
categories: [linux, developer, github]
tags: [github, verified user, authentication, github]
date: 2020-03-27
---


  When you create a new file directly on Github and push it to your branch, you will see in the commit, that this was done by an verified user.<br /><img class="alignnone size-full wp-image-484" src="https://joergi77.files.wordpress.com/2020/03/github1-2.png" alt="github1" width="947" height="227" />


  If you push it from your command line, it normally looks like this:


  <img class="alignnone size-full wp-image-474" src="https://joergi77.files.wordpress.com/2020/03/github2.png" alt="github2" width="947" height="60" />  
  P.S. if you followed the tutorial, and something went wrong, it will look like this:  
    <img class="alignnone size-full wp-image-473" src="https://joergi77.files.wordpress.com/2020/03/github3_fail.png" alt="github3_fail" width="947" height="60" />  

  But let's start from the beginning!    
First of all: Github gives you a great Readme for this, that's why I link it here were needed.

<ol><li style="list-style-type:none;"><ol><li>Add a <a href="https://help.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key">new Github GPG</a> key <a href="https://github.com/settings/gpg/new">here.</a><br />I used my 1234567890+your_github_username@users.noreply.github.com as my email is protected on Github.<br />You can always change the email of your <a href="https://help.github.com/en/github/authenticating-to-github/associating-an-email-with-your-gpg-key">key here</a></li><li>You should tell your git to use it, so you have to sign it as <a href="https://help.github.com/en/github/authenticating-to-github/telling-git-about-your-signing-key">described here</a></li><li>Sign every commit as mentioned <a href="https://help.github.com/en/github/authenticating-to-github/signing-commits">here</a></li><li>when you make now a commit, you will see that the commit is signed.</li></ol></li></ol><p>At the end, it <del>should</del> looks like this:</p><p><img class="alignnone size-full wp-image-483" src="https://joergi77.files.wordpress.com/2020/03/github_verified-1.png" alt="github_verified" width="952" height="235" /><br />I tried this tutorial on my other computer, and it all worked, if you have questions, let me know.</p>
