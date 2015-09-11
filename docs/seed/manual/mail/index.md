---
title: "Overview"
type: "manual"
zones:
    - "Seed"
sections:
    - "SeedMail"
tags:
    - "mail"
    - "maven"
menu:
    SeedMail:
        weight: 10
---

Mail support provides integrates JAVA mail in Seed. To add the mail support in your project, use
the following dependency snippet:

    <dependency>
        <groupId>org.seedstack.seed</groupId>
        <artifactId>seed-mail-support-core</artifactId>
    </dependency>
    
You may also need to add the JAVA Mail API (depending on your runtime environment):
    
    <dependency>
        <groupId>javax.mail</groupId>
        <artifactId>mail</artifactId>
    </dependency>

You can use any configured JAVA mail session in your application code by injecting it:
 
    @Inject
    @Named("myProvider1")
    Session myProvider1Session;
    
