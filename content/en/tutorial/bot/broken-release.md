---
title: "Broken Release"
date: 2020-10-01T13:48:26-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When using release-maven to release the light-platform components together, it is sometimes broken in the middle due to 502 error or build error. 

When this happens, you don't need to upgrade the version again to start the release process from the beginning. 

Follow the steps below to continue the build. 

### Release

To continue the release from the failed component, you need to comment out the released components in the release section. 

Update the failed component and build it in another terminal to confirm it is ready. 

After that, you need to update the flags to set skip_checkout and skip_release to false and all others to true. 

Call the run.sh again to continue the build and release.

### Resume
Once the release is done, you need to update the config file to remove the release section's comment. 

Set skip_release, skip_change_log and skip_checkin to true and others to false. 

Rerun the run.sh again to checkout again, publish release notes, deploy and upload artifacts. 

Note publishing needs the checkout tag to get the branch to publish to GitHub. 

The above steps will ensure all the activities of the release are done. 

### Recover

Recover all the changes in the config file to prepare for the next release. 

