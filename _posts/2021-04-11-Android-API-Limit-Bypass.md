---
layout: post
title:  "Android API Limit Bypass"
date:   2021-04-11
categories: Java
title: Android API Limit Bypass
---

In 2018 Android release new version Android Pie / 9 introducing some **Hidden APIs** in the android ecosystem , with limited access for developers are trying to bypass the restriction.

---
[](#header-1)**Policies**
---

To breakin the restrictions we have to understammd the system restrictions imposed, here i have tried to explain the method through reflection in Java Layer. 

Example :

       case Domain::kPlatform: {
    DCHECK(callee_context.GetDomain() == Domain::kCorePlatform);

     // If it is the Core Platform API that needs to be exposed, through
  if ((runtime_flags & kAccCorePlatformApi) ! = 0) {
    Return false;
        }

    // Close the access restriction completely, through
    // The default in Android Q is off, and it is unknown in R.
   EnforcementPolicy policy = Runtime::Current()->GetCorePlatformApiEnforcementPolicy();
    if (policy == EnforcementPolicy::kDisabled) {
    Return false;
         }

      return details::HandleCorePlatformApiViolation(member, caller_context, access_method, policy);
        }
