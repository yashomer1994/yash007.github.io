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
    
    2. Store caller domain, validate it and pass it .

    3. If conditions doesn't met as above,  The domain used in the code is **kApplications**, which will be looked first.

**GetCaller** ::

     bool VisitFrame() override REQUIRES_SHARED(Locks::mutator_lock_) {
       ArtMethod *m = GetMethod();
        if (m == nullptr) {
        // Attached native thread. Assume this is *not* boot class path.
    caller = nullptr;
        Return false;
    } else if (m->IsRuntimeMethod()) {
        // Skip the internal method of ART Runtime
        Return true;
    }

    ObjPtr<mirror::Class> declaring_class = m->GetDeclaringClass();
    if (declaring_class->IsBootStrapClassLoaded()) {
        // Skip the internal method in java.lang.Class
        if (declaring_class->IsClassClass()) {
            Return true;
    }

        // Skip java.lang.invoke. *
    ObjPtr<mirror::Class> lookup_class = GetClassRoot<mirror::MethodHandlesLookup>();
        if ((declaring_class == lookup_class || declaring_class->IsInSamePackage(lookup_class))
    &&! m->IsClassInitializer()) {
            Return true;
    }

        // Skip from java.lang.reflect if PREVENT_META_REFLECTION_BLACKLIST_ACCESS is Enabled. * Visit
        // The key to the system's restriction on "set baby reflection" is here.
    ObjPtr<mirror::Class> proxy_class = GetClassRoot<mirror::Proxy>();
        if (declaring_class->IsInSamePackage(proxy_class) && declaring_class ! = proxy_class) {
            if (Runtime::Current()->isChangeEnabled(kPreventMetaReflectionBlacklistAccess)) {
                Return true;
    }
    }
    }

    Caller = m;
    Return false;
    }

    --- 

**Domain According to Caller** 

    static Domain ComputeDomain(ObjPtr<mirror::ClassLoader> class_loader, const DexFile* dex_file) {
    if (dex_file == nullptr) {
        // dex_file == nullptr && class_loader == nullptr (i.e., the class loaded by BootClassLoader), then trusted
        return ComputeDomain(/* is_trusted= */ class_loader.IsNull());
    }
       // Get the Domain of dex_file
    return dex_file->GetHiddenapiDomain();
    }

    static Domain ComputeDomain(ObjPtr<mirror::Class> klass, const DexFile* dex_file)
    REQUIRES_SHARED(Locks::mutator_lock_) {

    Domain Domain = ComputeDomain(klass->GetClassLoader(), dex_file);

    if (domain == Domain::kApplication && klass->ShouldSkipHiddenApiChecks() && Runtime::Current()->IsJa vaDebuggable()) {
        // Under debug mode, developers can actively specify certain types of trust for debugging.
    domain = ComputeDomain(/* is_trusted= */ true);
    }

    Return domain;
    }

---
---
When the class is loaded first time, it initalises **dex_files** :

    static Domain DetermineDomainFromLocation(const std::string& dex_location,
    ObjPtr<mirror::ClassLoader> class_loader) {

    if (ArtModuleRootDistinctFromAndroidRoot()) {
        // Within several core apex modules
        if (LocationIsOnArtModule(dex_location.c_str())
    || LocationIsOnConscryptModule(dex_location.c_str())
    || LocationIsOnI18nModule(dex_location.c_str())) {
            return Domain::kCorePlatform;
    }

        // Under apex but not the core module
        if (LocationIsOnApex(dex_location.c_str())) {
            return Domain::kPlatform;
    }
    }

    // Under /system/framework/
    if (LocationIsOnSystemFramework(dex_location.c_str())) {
        return Domain::kPlatform;
    }

    // Under /system/ext/framework/
    if (LocationIsOnSystemExtFramework(dex_location.c_str())) {
        return Domain::kPlatform;
    }

    // ClassLoader is null (i.e. BootClassLoader)
    if (class_loader.IsNull()) {
    LOG(WARNING) << "DexFile" << dex_location <<" is in boot class path, but not in a known Location";
        return Domain::kPlatform;
    }

      return Domain::kApplication;
    }