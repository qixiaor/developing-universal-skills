---
name: spring-test-two-class
description: 规范 Spring Boot 测试只保留两个轻量测试类：`ApiTest` 用于带鉴权的接口测试，`ServiceTest` 用于直接调用业务实现层。适用于用户提到“写接口测试”“补 Service 测试”“不要新增 Test 类”“登录拿 token 后再调接口”“只允许两个测试类”“结果用 System.out.println 打印”“JSON 转字符串再打印”等场景，也适用于 Controller、MockMvc、token 登录流和 service 逻辑直测的测试补充或重构。
---

# Spring Test Two Class

# Overview

Keep Spring tests inside exactly two classes:

- `ApiTest`: controller/interface tests that go through HTTP and require authentication.
- `ServiceTest`: service implementation tests that call business methods directly.

Write tests so they are short, direct, and runnable without extra abstraction.

## Mandatory Rules

- Allow only two test classes.
- Do not create extra test classes. Add new API cases to `ApiTest`. Add new service cases to `ServiceTest`.
- Do not introduce `BaseTest`, abstract test classes, utility classes, or multi-layer wrappers.
- Keep setup and assertions close to the test method. Favor inline code over reusable helpers.
- Keep test classes lightweight, readable, and commented only where the intent is not obvious.
- Use `System.out.println` for logs.
- Convert JSON to a string before printing it.
- Do not print raw objects directly.
- Make the final code runnable in the target project. Reuse existing endpoints, DTOs, services, and JSON libraries already present in the repo.

## Choose The Test Type

Use `ApiTest` when the target is:

- `Controller` behavior
- endpoint verification
- token or login protected APIs
- request/response flows that should go through `MockMvc`

Use `ServiceTest` when the target is:

- service implementation logic
- business rules that do not need HTTP
- direct method calls with DTO input and printed result output

## ApiTest Pattern

Follow this flow:

1. Inject `MockMvc`.
2. Log in in `@BeforeEach`.
3. Extract and store the token.
4. Call the target API with the token in the header.
5. Print the response string with `System.out.println`.

Use this shape:

```java
@SpringBootTest
@AutoConfigureMockMvc
public class ApiTest {

    @Autowired
    private MockMvc mockMvc;

    private String token;

    /**
     * 初始化登录，获取 token
     */
    @BeforeEach
    void login() throws Exception {
        String loginJson = "{ \"username\": \"admin\", \"password\": \"123456\" }";

        MvcResult result = mockMvc.perform(
                post("/login")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(loginJson)
        ).andReturn();

        String response = result.getResponse().getContentAsString();
        System.out.println("登录返回: " + response);

        token = response;
    }

    /**
     * 示例接口测试
     */
    @Test
    void testXXXApi() throws Exception {
        String reqJson = "{ \"id\": 1 }";

        MvcResult result = mockMvc.perform(
                post("/xxx/api")
                        .header("Authorization", token)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(reqJson)
        ).andReturn();

        String response = result.getResponse().getContentAsString();
        System.out.println("接口返回: " + response);
    }
}
```

Adjust only the endpoint, login payload, header name, and token parsing to match the real project.

## ServiceTest Pattern

Follow this flow:

1. Inject the target service.
2. Build the DTO inline.
3. Call the service method directly.
4. Convert the result to a JSON string.
5. Print the JSON string with `System.out.println`.

Use this shape:

```java
@SpringBootTest
public class ServiceTest {

    @Autowired
    private XxxService xxxService;

    /**
     * 示例业务测试
     */
    @Test
    void testXXXService() {
        XxxDTO dto = new XxxDTO();
        dto.setId(1L);

        Object result = xxxService.xxxMethod(dto);
        System.out.println("返回结果: " + JSON.toJSONString(result));
    }
}
```

If `JSON.toJSONString` is not available, use the project's existing JSON serializer, but still print a string rather than the object itself.

## Edit Strategy

- Search the repo for existing `ApiTest` and `ServiceTest` first.
- If both classes already exist, append new test methods to them only.
- If they do not exist yet, create only these two classes and no others.
- Keep imports minimal and consistent with the project.
- Add a short method comment for each new test and for login setup when present.
- Avoid assertion-heavy ceremony unless the project already depends on it. Printing the returned data is required.

## Output Rules

- Return code that can run directly after adjusting project-specific endpoint paths, DTO names, and service names.
- Prefer complete test methods over partial snippets.
- Keep method bodies flat and easy to scan.
- Always print response strings or JSON strings.
- Never output `System.out.println(result);`
