+++
date = "2012-10-23T10:27:00-07:00"
draft = false
tags = ["continuous delivery"]
title = "Notes On Continuous Delivery"

+++

### Only Build Your Binaries Once
- Reduce the risk of introducing difference on each build.
- Takes time to compile code (specially in large systems)

### Deploy the Same Way to Every Environment
- Once you reach Production you are confident that your deployments work.

### Smoke-Test your Deployments
- After deploying, immediately make sure that the most basic functionality works as expected.

### Deploy into a copy of Production
- Make your environments a similar to production as pipeline.  Again, to reduce the risk of ofintroducing bugs.

### Each Change Should Propagate through the Pipeline Instantly
- Each checking should trigger the start of the build process.

### If Any Part of the Pipeline Fails, Stop the Line
- Developers should make sure that their code didn't break anything.  If it does, they should fix it immediately.
