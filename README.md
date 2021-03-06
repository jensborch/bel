# BEL - Minimalistic Java EE executor library with back-off

This library provides a CDI injectable resilient executor with a configurable back-off. The executor simplifies calling potentially fragile services without overloading them, as it will	 retry code execution based on a back-off strategy.

This library implements an [Executor](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Executor.html) as a simple wrapper around the Java EE [ManagedScheduledExecutorService](http://docs.oracle.com/javaee/7/api/javax/enterprise/concurrent/ManagedScheduledExecutorService.html). The Executor is a dependent CDI bean and will thus be instantiated in the same scope as the bean it is injected into.

The resilient executor will automatically start a new transaction when executing code using ManagedScheduledExecutorService, further simplifying creating resilient asynchronous code.

# Status

Module is considered beta quality.

[![Build Status](https://travis-ci.org/Nykredit/bel.svg?branch=master)](https://travis-ci.org/Nykredit/bel) [![Coverage Status](https://coveralls.io/repos/github/Nykredit/bel/badge.svg?branch=master)](https://coveralls.io/github/Nykredit/bel?branch=master)

# Usage

Add the following Maven dependency to your project:

```xml
<dependency>
    <groupId>dk.nykredit.resilience</groupId>
    <artifactId>bel</artifactId>
    <version>0.9.2</version>
</dependency>
```

The resilient executor implements the Executor interface and using the executor is thus similar to using Java EE ManagedScheduledExecutorService:

```java
@Inject
@Resilient(strategy = PolynomialBackoffStrategy.class)
@PolynomialBackoffStrategyConfig(delay = 1, maxDelay = 1800, retries = 100, timeUnit = TimeUnit.SECONDS)
private ResilientExecutor executor;

executor.execute(() -> {
        Customer customer = repository.get(id);
        StatusRepresentation status = statusConnector.get(customer.getId());
        customer.setStatus(status.getStatus());
    }, e -> {
        LOGGER.error("Error calling service...", e);
        Customer customer = repository.get(id);
        customer.setStatus(Status.UNKNOWN);
    });
}
```
