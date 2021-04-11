---
layout: post
title:  "Android API Restrictions"
date:   2021-04-11
categories:  Android, Java
title: Android API Restrictions
---

In 2018 Android release new version Android Pie / 9 introducing some **Hidden APIs** in the android ecosystem , with limited access for developers are trying to bypass the restriction.

---
[](#header-1)**Policies**
---

To breakin the restrictions we have to understammd the system restrictions imposed, here i have tried to explain the method through reflection in Java Layer. 

Example : Method through reflection is Java Layer "**Class.getMethod/getDeclaredMethod**".

Method Implementation : 

     static jobject Class_getDeclaredMethodInternal(JNIEnv* env, jobject javaThis, jstring name, jobjectA rray args) {
  // Omit irrelevant code...
    handle<mirror::Method> result = hs.NewHandle(
    mirror::Class::GetDeclaredMethodInternal<kRuntimePointerSize>(
    soa.Self(),
    Klass,
    soa.Decode<mirror::String> (name),
    soa.Decode<mirror::ObjectArray<mirror::Class>>(args),
    GetHiddenapiAccessContextFunction(soa.Self()));
    if (result == nullptr || ShouldDenyAccessToMember(result->GetArtMethod(), soa.Self())) {
     Return nullptr;
    }
      return soa.AddLocalReference<jobject>(result.Get());
    }

---

Going through code , if **ShouldDenyAccessToMember** returns value as true, the value returned is null and the uppeer layer will throw an exception that the method doesn't exist. 

To call **ShouldDenyAccessToMember** we use **hiddenapi::ShouldDenyAccessToMembe**, as shown below :

        template<typename T>
    Inline bool shouldDenyAccessToMember(T* member,
                                     const std::function<AccessContext()>& fn_get_access_context,
    AccessMethod access_method)
    REQUIRES_SHARED(Locks::mutator_lock_) {

    const uint32_t runtime_flags = GetRuntimeFlags(member);

    // 1: If the member is a public API, directly through
    if ((runtime_flags & kAccPublicApi) ! = 0) {
    Return false;
    }

      // 2: Not a public API (i.e., a hidden API), to get the Domain of the caller and the accessed member
      // Because obtaining the caller needs to trace back to the call stack, the performance is very poor, so try to avoid this consumption.
      const AccessContext caller_context = fn_get_access_context();
      const AccessContext callee_context(member->GetDeclaringClass());

      // 3: If the caller is trusted, return directly
      if (caller_context.CanAlwaysAccess(callee_context)) {
        Return false;
    }

      // 4: Untrusted caller tries to access the hidden API and determines behavior according to the caller's Domain
      switch (caller_context.GetDomain()) {
        case Domain::kApplication: {
    DCKCK(! callee_context.IsApplicationDomain());

      // If access checking is completely disabled, then return directly
      EnforcementPolicy policy = Runtime::Current()->GetHiddenApiEnforcementPolicy();
      if (policy == EnforcementPolicy::kDisabled) {
        Return false;
    }

      // 5: Call detail::ShouldDenyAccessToMemberImpl to decide if access is required.
      return detail::ShouldDenyAccessToMemberImpl(member, api_list, access_method);
    }
    // Omit
    }
 
 ---
 
 ---
 **Analysis**
 ---
Here i will try to explain and analyse the above code step by step.

    1. Validate the target memeber is a Public API, if check got passed it  will acquired by "**GetRunTimeFlags**" , this flag is stored in  the member **access_flags**.