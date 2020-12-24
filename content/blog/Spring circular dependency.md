---
title: "How to avoid CI in Spring"
date: 2020-09-12T12:14:34+06:00
description: "How To Avoid Circular Dependency In Spring"
author: "Sajad Hayatlou"
type: "post"
---


### 1.Definition of a bean


`Bean` is a key concept of the Spring Framework. As such, understanding this notion is crucial to get the hang of the framework and to use it in an effective way.

Here's a definition of beans in the Spring Framework documentation:

{{< quotate >}}

In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.

{{< /quotate >}}

Inversion of Control, or `IoC` for short, is a process in which an object defines its dependencies without creating them. This object delegates the job of constructing such dependencies to an IoC container.

But what is circular dependency and when it happens?

It happens when a bean say `A depends on another bean B, while the B itself depends on the A as well`:

{{< figure src="https://i.ytimg.com/vi/7tPwr56PXWs/maxresdefault.jpg" 
  width=300 height=300 style="float:left;" >}}

Of course, we could have a more complex scenario:

`Bean A → Bean B → Bean C → Bean D → Bean E → Bean A`

### 2. What Happens in Spring

When Spring context is loading all the beans, it tries to create beans in the order needed for them to work completely. For instance, if we didn’t have a circular dependency, like the following case:

Bean A → Bean B → Bean C

Spring will create bean `C`, then create bean `B` (and inject bean `C` into into the `B`) , then create bean `A` (and inject bean `B` into `A`).

But, when having a circular dependency, Spring cannot decide which of the beans should be created first, since they depend on one another. In these cases, Spring will raise a `BeanCurrentlyInCreationException` while loading the context.

Note that, this problem can happen if you choose the constructor injection; if you use other types of injections you should not find this problem since the dependencies will be injected when they are needed and not on the context loading.


### 3. A Simple Example

Let’s define two beans that depend on each other (via constructor injection of course)


{{< highlight java >}}

/**
* This class needs B
*/
@Component
public class BeanA {
    private BeanB beanB;

    @Autowired
    public BeanA(BeanB beanB) {
        this.beanB = beanB;
    }

    // rest of the code
}

/**
* This class needs A
*/
@Component
public class BeanB {
    private BeanA beanA;

    @Autowired
    public BeanB(BeanA beanA) {
        this.beanA = beanA;
    }

    // rest of the code
}

{{< /highlight >}}

### 4. The Workarounds

Now, there are some of the most popular ways to deal with this problem

### 4.1. Redesign

When you have a circular dependency, it’s likely you have a design problem and the responsibilities are not well separated. You should try to redesign components properly so their hierarchy prevents any circular dependency to appear.

If you cannot redesign the components (there can be a few reasons, e.g., legacy code, code that has already been tested and cannot be modified, not enough time or resources for a comprehensive redesign, ...) there are some workarounds to try.

### 4.2. Use @Lazy Annotation

A simple fix to break the cycle is saying Spring to initialize one of the beans lazily. That is, instead of fully initializing the bean, it will create a proxy to inject it into the other bean. The injected bean will only be fully created when it’s first needed.

To try this with our example, you can change the CircularDependencyA to the following:


{{< highlight java >}}
/**
* Using Lazy annotation to avoid circular dependency (Not recommended!)
/*
@Component
public class BeanA {
    private BeanB beanB;

    /**
    * Lazily inject the required bean 
    /*
    @Autowired
    public BeanA(@Lazy BeanB beanB) {
        this.beanB = beanB;
    }

    // rest of the code
}
{{< /highlight >}}


### 4.3. Use Setter or Field Injection

One of the most popular workarounds, and also what Spring documentation proposes, is using setter injection.

Simply put if you change the way your beans are wired to use `setter or field injection` instead of `constructor injection` – that does address the problem. This way Spring creates beans, but the dependencies are not injected in the phase of object initialization, until they are needed.

### 4.4. Use @PostConstruct

Another way to break the cycle is injecting the dependency using `@Autowired` on one of the beans, and then use a method annotated with `@PostConstruct` to set the other dependency.

{{< highlight java >}}
@Component
public class BeanA {

    @Autowired
    private BeanB beanB;

    @PostConstruct
    public void init() {
        beanB.setBeanA(this);
    }

    public BeanB getBeanB() {
        return beanB;
    }
}

@Component
public class BeanB {
    private BeanA beanA;

    public void setBeanA(BeanA beanA) {
        this.beanA = beanA;
    }
}

{{< /highlight >}}

### In Conclusion

There are many ways to deal with circular dependencies in Spring. The first thing to consider is to redesign your components so there is no need for circular dependencies, they are usually a symptom of a design that can be improved.

But if you absolutely need to have circular dependencies in your project, you can follow some of the workarounds suggested here.

The preferred method is using setter injections, But there are other alternatives, generally based on stopping Spring from managing the initialization and injection of the beans, and doing that yourself using one strategy or another.





