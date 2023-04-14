---
layout: post
title:  "Python assert statement and the -O flag: Security Best Practices – Part I"
---

In this article, we will explore the use of the assert statement in Python and the implications of running Python scripts with the -O (optimize) flag, and in the second part, we’ll go through a real-world scenario where it is possible to bypass security restrictions implemented on what appears to be protected APIs in an open source project.

Although the assert statement is a useful tool for debugging and testing, it should not be relied upon for security or performance-critical code. By the end of this article, you will understand the purpose and proper usage of the assert statement and how the -O flag can trigger some critical security issues that its severity depends on how it is implemented.

## Understanding the Assert Statement

The assert statement is a debugging aid in Python that tests a condition and triggers an error if the condition is not met. It is used to catch programming errors early, making it easier to identify and fix issues during development.

``` assert condition, error_message ```

Here, the condition is the expression to be evaluated, and error_message is an optional message that will be displayed if the assertion fails.

Example:

```
def divide(x, y):
    assert y != 0, "Division by zero is not allowed."
    return x / y

result = divide(10, 2)  # This will work fine.
result = divide(10, 0)  # This will raise an AssertionError with the message "Division by zero is not allowed."
```

### What is the -O Flag?

The -O (optimize) flag is a command line option used when running Python scripts. It enables basic optimizations and changes the behavior of the interpreter slightly to improve performance. 

When running a script with the -O flag, the ``` __debug__ ``` global variable is set to False, and all assert statements are removed from the code, effectively disabling them.

To run the python script with the -O flag, use the following command:

``` python -O assert.py ```


### Test it yourself.

The goal of the example that we’ll see now is to access the secret flag stored in the secure_data function by exploiting the insecure use of the assert statement.

Create a file named insecure_assert.py with the following content:

```
import base64

def is_authorized(username, password):
    return username == "admin" and password == "secure_password"

def secure_data(username, password):
    assert is_authorized(username, password), "Unauthorized access!"
    flag = "Rm9sbG93IG1lIG9uIHR3aXR0ZXIgaHR0cHM6Ly90d2l0dGVyLmNvbS9hZXNzYWRlaw"
    return base64.b64decode(flag).decode()

username = input("Enter your username: ")
password = input("Enter your password: ")
print(secure_data(username, password))

```

Run the script with the command below.

``` python -O insecure_assert.py ```

When the script is running with the -O flag, the assert statement will be ignored, bypassing the access control check and allowing unauthorized users to access the secret flag.

## Best Practices and Common Pitfalls

The assert statement is intended for debugging and testing purposes. It should not be used in production code, as it can be disabled with the -O flag. Use exception handling and error checking techniques, such as try and except, to handle errors in your production code.

Using assert statements can make your code more robust during development, but it’s essential to consider the trade-offs. When using assert statements, you may introduce additional overhead, making your code slower. Additionally, if you rely too heavily on assert statements, you may create a false sense of security, neglecting proper error handling and validation in your code.

Logging is an essential tool for understanding the behavior of your code, particularly when debugging complex issues. By using Python’s built-in logging module, you can create detailed logs of your code’s execution, making it easier to identify and fix issues. Additionally, using tracebacks can help you pinpoint the location of an error in your code, allowing you to resolve the issue more quickly.

## Conclusion

In conclusion, the assert statement can be a valuable tool for debugging and testing during development, but it should be used with caution. Keep in mind that the -O flag can disable assert statements, rendering them unsuitable for security or performance-critical code. By following best practices for error handling, input validation, and testing, you can build robust and secure applications in Python. Always be mindful of the trade-offs and implications of using assert statements and the -O flag, and choose the right tools and techniques for each situation in your development process.

In our upcoming article, we will showcase a real-world scenario in which we discovered several projects employing the aforementioned techniques to validate security implications using the assert statement. We will discuss how we were able to circumvent the security check implementation on some protected APIs by exploiting the limitations of the assert statement when it’s used alongside the -O flag.

This case study will provide valuable insights into the risks associated with relying on assert statements for security purposes and emphasize the importance of adopting proper security measures and best practices in API protection.

### References:

https://snyk.io/blog/the-dangers-of-assert-in-python/
https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement
https://deepsource.io/blog/python-security-pitfalls/