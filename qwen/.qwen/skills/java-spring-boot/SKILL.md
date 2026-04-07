---
name: java-spring-boot
description: "Java Spring Boot development. Use for Spring applications, REST APIs, microservices, dependency injection, JPA/Hibernate, Spring Security, and Java best practices. Trigger words: spring, spring boot, java, jpa, hibernate, spring security,@RestController, @Service, @Repository, @Entity, microservice."
---

# Java Spring Boot Development

## When to Use

- Building REST APIs with Spring Boot
- Working with JPA/Hibernate entities
- Configuring Spring Security
- Setting up Spring Data repositories
- Dependency injection patterns
- Application configuration (application.yml/properties)

## Spring Boot Conventions

### Controller Layer

```java
@RestController
@RequestMapping("/api/v1/resources")
public class ResourceController {

    private final ResourceService resourceService;

    public ResourceController(ResourceService resourceService) {
        this.resourceService = resourceService;
    }

    @GetMapping
    public List<ResourceDto> findAll() {
        return resourceService.findAll();
    }

    @GetMapping("/{id}")
    public ResourceDto findById(@PathVariable Long id) {
        return resourceService.findById(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResourceDto create(@Valid @RequestBody ResourceDto dto) {
        return resourceService.create(dto);
    }
}
```

### Service Layer

```java
@Service
@Transactional(readOnly = true)
public class ResourceService {

    private final ResourceRepository resourceRepository;

    public ResourceService(ResourceRepository resourceRepository) {
        this.resourceRepository = resourceRepository;
    }

    public List<ResourceDto> findAll() {
        return resourceRepository.findAll().stream()
            .map(this::toDto)
            .toList();
    }

    @Transactional
    public ResourceDto create(ResourceDto dto) {
        Resource entity = toEntity(dto);
        Resource saved = resourceRepository.save(entity);
        return toDto(saved);
    }

    private ResourceDto toDto(Resource entity) {
        return new ResourceDto(entity.getId(), entity.getName());
    }

    private Resource toEntity(ResourceDto dto) {
        Resource entity = new Resource();
        entity.setName(dto.name());
        return entity;
    }
}
```

### Repository Layer

```java
@Repository
public interface ResourceRepository extends JpaRepository<Resource, Long> {

    Optional<Resource> findByName(String name);

    List<Resource> findByActiveTrue();

    @Query("SELECT r FROM Resource r WHERE r.name LIKE %:name%")
    List<Resource> searchByName(@Param("name") String name);
}
```

### Entity

```java
@Entity
@Table(name = "resources")
public class Resource {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column
    private Boolean active = true;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    // Getters and setters
}
```

## Best Practices

1. **Constructor injection** — Use constructor injection for required dependencies
2. **DTOs** — Use record classes for DTOs in Java 17+
3. **Transactional** — Use `@Transactional` on service methods, not on controllers
4. **Validation** — Use `@Valid` and Bean Validation annotations
5. **Exception handling** — Use `@ControllerAdvice` for global exception handling
6. **DTO mapping** — Use MapStruct or manual mapping, avoid mapping in controllers
7. **Pagination** — Use `Page<>` and `Pageable` for large datasets

## Testing

```java
@WebMvcTest(ResourceController.class)
class ResourceControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ResourceService resourceService;

    @Test
    void findAll_returnsResources() throws Exception {
        when(resourceService.findAll()).thenReturn(List.of(
            new ResourceDto(1L, "Test")
        ));

        mockMvc.perform(get("/api/v1/resources"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].name").value("Test"));
    }
}
```