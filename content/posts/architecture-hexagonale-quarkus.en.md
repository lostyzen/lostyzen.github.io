---
title: "Hexagonal Architecture in Java with Quarkus"
date: 2025-10-12T10:00:00+02:00
lastmod: 2025-10-12T10:00:00+02:00
draft: false
weight: 3
tags: ["Java", "Quarkus", "Architecture", "Clean Architecture", "DDD", "API", "Ports and Adapters", "Software Design", "Best Practices"]
categories: ["Architecture", "Java"]
author: "lostyzen"
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "Complete guide to hexagonal architecture in Java with Quarkus: structure, concrete examples, testing, and best practices for maintainable and scalable APIs."
canonicalURL: "https://lostyzen.github.io/en/posts/architecture-hexagonale-quarkus/"
disableHLJS: false
disableShare: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/architecture-hexagonale-og.png" # Specific image for this article
    alt: "Hexagonal Architecture Java Quarkus"
    caption: "Structuring Java APIs with hexagonal architecture"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggest changes"
    appendFilePath: true
keywords: ["hexagonal architecture", "java", "quarkus", "ports and adapters", "clean architecture", "ddd", "domain driven design", "api design", "software architecture", "java best practices", "microservices", "spring boot alternative"]
schema:
  type: "BlogPosting"
  datePublished: "2025-10-12T10:00:00+02:00"
  dateModified: "2025-10-12T10:00:00+02:00"
  author: "lostyzen"
  publisher: "DevOps Blog"
# Open Graph / Facebook
images: ["/images/architecture-hexagonale-og.png"]
# Twitter Card (temporarily disabled - reactivate when accounts created)
# twitter:
#   card: "summary_large_image"
#   site: "@votre_twitter"
#   creator: "@votre_twitter"
---

🏗️ **Hexagonal Architecture in Java: Structuring APIs for Maintainability**

After exploring [Quarkus performance](/en/posts/quarkus-api-haute-performance/), let's talk about an equally crucial aspect: how to properly structure your API code. Hexagonal architecture (or "Ports & Adapters") isn't just a theoretical concept - it's a pragmatic approach that transforms the maintainability of your Java projects.

## 🎯 **The Problem with Traditional Architectures**

How many times have you seen Spring Boot projects that end up like this:
- **Fat Controllers**: Business logic mixed with HTTP handling
- **Anemic Services**: Just CRUD operations that map to the database
- **Tight Coupling**: Impossible to test without starting the entire application
- **Technical Debt**: Each new feature becomes more complicated to implement
- **Fragile Tests**: Changing one dependency breaks all tests

The problem? Business logic is scattered everywhere, coupled to technical details (database, HTTP, etc.). Result: code that's difficult to maintain and evolve.

## 🔄 **Hexagonal Architecture: Clear Separation**

### **The Fundamental Principle**
The idea is simple: **isolate business logic** at the center, and **decouple** everything else (database, REST API, messaging, etc.) via interfaces.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Controllers   │───▶│   Use Cases     │───▶│   Repositories  │
│   (Adapters)    │    │   (Domain)      │    │   (Adapters)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
   HTTP/REST              Business Logic            Database
```

### **Concrete Benefits**
✅ **Isolated Tests**: Test your business logic without starting the app  
✅ **Flexibility**: Change database without impacting business logic  
✅ **Scalability**: Easily add new entry channels (GraphQL, gRPC, etc.)  
✅ **Maintainability**: Organized code, clear responsibilities  
✅ **Reusability**: Business logic can be reused in different contexts  

## 🏛️ **Concrete Structure of a Hexagonal Project**

### **Package Organization**
```
org.acme.demo/
├── domain/
│   ├── model/          # Business entities
│   ├── port/           # Interfaces (ports)
│   └── service/        # Use cases / Business services
├── infrastructure/
│   ├── adapter/
│   │   ├── in/         # Incoming adapters (REST, GraphQL)
│   │   └── out/        # Outgoing adapters (DB, External APIs)
│   └── config/         # Spring/Quarkus configuration
└── application/        # Application entry point
```

### **Layers Explained**

**Domain** (the core):
- **Entities**: Business objects with their rules
- **Ports**: Interfaces defining contracts
- **Use Cases**: Pure business logic, no technical dependencies

**Infrastructure** (the periphery):
- **Adapters In**: REST Controllers, GraphQL endpoints, message consumers
- **Adapters Out**: JPA Repositories, HTTP clients, message publishers
- **Config**: Dependency injection, technical configuration

## 🛠️ **Concrete Example with Quarkus**

### **1. Business Entity (Domain)**
```java
// domain/model/User.java
public class User {
    private final UserId id;
    private final Email email;
    private final UserStatus status;
    
    // Business logic in the entity
    public void activate() {
        if (this.status == UserStatus.SUSPENDED) {
            throw new BusinessException("Cannot activate suspended user");
        }
        this.status = UserStatus.ACTIVE;
    }
}
```

### **2. Port (Domain Interface)**
```java
// domain/port/UserRepository.java
public interface UserRepository {
    Optional<User> findById(UserId id);
    void save(User user);
    List<User> findByStatus(UserStatus status);
}
```

### **3. Use Case (Business Logic)**
```java
// domain/service/ActivateUserUseCase.java
@ApplicationScoped
public class ActivateUserUseCase {
    private final UserRepository userRepository;
    
    public void execute(UserId userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        
        user.activate(); // Business logic in the entity
        userRepository.save(user);
    }
}
```

### **4. REST Adapter (Infrastructure)**
```java
// infrastructure/adapter/in/UserController.java
@RestController
@Path("/users")
public class UserController {
    private final ActivateUserUseCase activateUserUseCase;
    
    @POST
    @Path("/{id}/activate")
    public Response activateUser(@PathParam("id") String id) {
        activateUserUseCase.execute(new UserId(id));
        return Response.ok().build();
    }
}
```

### **5. Repository Adapter (Infrastructure)**
```java
// infrastructure/adapter/out/JpaUserRepository.java
@ApplicationScoped
public class JpaUserRepository implements UserRepository {
    @Inject
    EntityManager em;
    
    @Override
    public Optional<User> findById(UserId id) {
        UserEntity entity = em.find(UserEntity.class, id.getValue());
        return entity != null ? Optional.of(entity.toDomain()) : Optional.empty();
    }
}
```

## 🧪 **Testing: The Real Advantage**

### **Use Case Unit Test**
```java
@Test
void should_activate_user_when_user_exists() {
    // Given
    User user = new User(userId, email, UserStatus.INACTIVE);
    when(userRepository.findById(userId)).thenReturn(Optional.of(user));
    
    // When
    activateUserUseCase.execute(userId);
    
    // Then
    verify(userRepository).save(argThat(u -> u.getStatus() == UserStatus.ACTIVE));
}
```

**Benefits**:
- No application startup
- Fast tests (< 100ms)
- Isolated business logic
- Simple dependency mocking

### **Integration Test**
```java
@QuarkusTest
class UserControllerIT {
    @Test
    void should_activate_user_via_rest_api() {
        given()
            .when().post("/users/123/activate")
            .then().statusCode(200);
        
        // Verify in database that user is activated
    }
}
```

## 💡 **Quarkus + Hexagonal Architecture: The Winning Combo**

### **Why They Work Well Together?**

**Native Dependency Injection**:
- `@ApplicationScoped` for use cases
- `@Inject` for automatic port injection
- Annotation-based configuration, no XML

**Performance**:
- Native compilation preserves architecture
- Fast startup even with complex structure
- Reflection eliminated, optimized interfaces

**Integrated Testing**:
- `@QuarkusTest` for integration tests
- `@TestProfile` for different test environments
- Native Mockito mocking

## 📊 **Ideal Use Cases**

### **Complex APIs with Rich Business Logic**
- E-commerce applications (order management, pricing, inventory)
- Management systems (CRM, ERP, etc.)
- APIs with evolving business rules

### **Multi-Team Projects**
- Clear separation of responsibilities
- Domain team vs infrastructure team
- Facilitated integration

### **Applications with Multiple Channels**
- REST + GraphQL + gRPC
- Batch + API + Events
- Progressive migration from legacy systems

## ⚠️ **When NOT to Use It?**

### **Simple CRUD**
For a basic API without complex business logic, it might be overkill. A simple Controller → Service → Repository pattern might suffice.

### **Rapid Prototypes**
For a POC or demo, the complete structure might slow down initial development.

### **Junior Teams**
There's a learning curve. The team needs to be trained in DDD and hexagonal architecture concepts.

## 🚀 **Getting Started**

### **Steps to Begin**
1. **Identify your business domain**: What are your main entities?
2. **Define your use cases**: What does your application actually do?
3. **Create the ports**: What interfaces does your domain need?
4. **Implement the adapters**: REST, database, etc.
5. **Test by layer**: Domain → Use Cases → Adapters

### **Progressive Migration**
No need to refactor everything at once:
- Start with new features
- Gradually isolate existing business logic
- Refactor the most critical parts

### **💡 Concrete Example**
To see all these concepts in action, check out the demonstration repository that implements a complete Quarkus API with hexagonal architecture:

🔗 **[GitHub Repository - Quarkus Demo API](https://github.com/lostyzen/quarkus-demo-basic-api/tree/2.0.0)**

## 🔮 **What's Next?**

### **Domain Driven Design (DDD)**
Hexagonal architecture pairs perfectly with DDD:
- Aggregates, Value Objects, Domain Events
- Bounded contexts for large projects
- Event Sourcing for historization

### **CQRS (Command Query Responsibility Segregation)**
Separate commands (write) from queries (read):
- Command use cases vs read use cases
- Separate performance optimization
- Improved scalability

## 🎉 **In Summary**

Hexagonal architecture + Quarkus means:
- **Maintainable** and **scalable** code
- **Fast** and **reliable** tests  
- **Technological** flexibility
- **Preserved** performance

Yes, it requires a bit more initial effort. But in the long run, you gain enormously in productivity and peace of mind.

Hexagonal architecture isn't just an academic pattern - it's a pragmatic tool for robust and durable APIs.

**Are you already testing it?** Share your feedback!