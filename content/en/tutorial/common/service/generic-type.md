---
title: "Generic Type"
date: 2017-11-21T12:27:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Sometimes, we have an interface with generic types, i.e. Validator<T> which Type is an model
that will be validated. There might be a lot of different models that need to be validated in
runtime. And we need to find the right instance of the validator that match to the exact model
type.

Let's say we have this interface.

```
public interface Validator<T> {
    boolean validate(T object);
}

```

Here is an abstract class that implements the interface.

```
public abstract class ValidatorBase<T> implements Validator<T> {
}

```

And here are several implementations.

```
public class LicenseValidator extends ValidatorBase<License> {
    @Override
    public boolean validate(License license) {
        return "license".equals(license.getName());
    }
}

```


```
public class ContactValidator extends ValidatorBase<Contact> {
    @Override
    public boolean validate(Contact contact) {
        return "contact".equals(contact.getName());
    }
}

```


```
public class InfoValidator extends ValidatorBase<Info> {
    Validator<Contact> contactValidator = SingletonServiceFactory.getBean(Validator.class, Contact.class);
    Validator<License> licenseValidator = SingletonServiceFactory.getBean(Validator.class, License.class);

    @Override
    public boolean validate(Info info) {
        return contactValidator.validate(info.getContact()) && licenseValidator.validate(info.getLicense());
    }
}

```

As you can see that InfoValidator is using ContactValidator and LicenseValidator. 

Here is the service.yml configuration that binds them all together. 

```
- com.networknt.service.Validator<com.networknt.service.Contact>:
  - com.networknt.service.ContactValidator
- com.networknt.service.Validator<com.networknt.service.License>:
  - com.networknt.service.LicenseValidator
- com.networknt.service.Validator<com.networknt.service.Info>:
  - com.networknt.service.InfoValidator

```

To test it, you can write a test case to do so.

```
    @Test
    public void testInfoValidator() {
        Validator<Info> infoValidator = SingletonServiceFactory.getBean(Validator.class, Info.class);
        Info info = SingletonServiceFactory.getBean(Info.class);
        Assert.assertTrue(infoValidator.validate(info));
    }

```


