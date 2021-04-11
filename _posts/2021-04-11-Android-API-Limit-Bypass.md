---
layout: post
title:  "Android API Restrictions"
date:   2021-04-11
categories: Java
title: Android API Restrictions
---

In 2018 Android release new version Android Pie / 9 introducing some **Hidden APIs** in the android ecosystem , with limited access for developers are trying to bypass the restriction.

---
[](#header-1)**Policies**
---

To breakin the restrictions we have to understammd the system restrictions imposed, here i have tried to explain the method through reflection in Java Layer. 

Example :

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

