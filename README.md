# Solving reCAPTCHA in C++ 

reCAPTCHA is a widely-used tool to prevent bots from engaging in abusive activities on websites. However, developers may need to bypass these challenges for legitimate purposes. In this guide, we'll walk through how to solve reCAPTCHA programmatically using C++ by interacting with the [CapSolver API](https://docs.capsolver.com/guide/api-server.html?utm_source=github&utm_medium=repo&utm_campaign=recaptchac++). We'll be using the `cpr` library for HTTP requests and `jsoncpp` for JSON parsing.

Additionally, we'll discuss how tools like [undetected](https://github.com/ultrafunkamsterdam/undetected-chromedriver) and [Playwright](https://playwright.dev/) can be used to manage reCAPTCHA in a more sophisticated way when combined with your C++ applications. Let's get started!

## Prerequisites

Before starting, ensure you have the following libraries installed:

- **cpr**: A C++ HTTP library.
- **jsoncpp**: A C++ library for JSON parsing.

You can install these libraries using [vcpkg](https://github.com/microsoft/vcpkg):

```bash
vcpkg install cpr jsoncpp
```

For handling browser automation and evasion techniques, you might consider using [Playwright](https://playwright.dev/) in combination with C++ bindings or a wrapper to handle reCAPTCHA challenges in a more undetected manner.

## Project Setup

1. **Create a New C++ Project**: Start by creating a new C++ project. Ensure you include the necessary headers for `cpr` and `jsoncpp`.

    ```cpp
    #include <iostream>
    #include <cpr/cpr.h>
    #include <json/json.h>
    ```

2. **Include Playwright (Optional)**: If your use case involves web automation, integrating Playwright could be beneficial. However, since Playwright has limited support directly in C++, you might consider using Python with `pybind11` or another interface for cross-language functionality.

## Step-by-Step Implementation

We'll define two main functions:

1. **createTask**: This function creates a reCAPTCHA task.
2. **getTaskResult**: This function retrieves the result of the created task.

### Step 1: Creating the reCAPTCHA Task

```cpp
#include <iostream>
#include <cpr/cpr.h>
#include <json/json.h>

std::string createTask(const std::string& apiKey, const std::string& websiteURL, const std::string& websiteKey) {
    Json::Value requestBody;
    requestBody["clientKey"] = apiKey;
    requestBody["task"]["type"] = "ReCaptchaV2Task";
    requestBody["task"]["websiteURL"] = websiteURL;
    requestBody["task"]["websiteKey"] = websiteKey;

    Json::StreamWriterBuilder writer;
    std::string requestBodyStr = Json::writeString(writer, requestBody);

    cpr::Response response = cpr::Post(
        cpr::Url{"https://api.capsolver.com/createTask"},
        cpr::Body{requestBodyStr},
        cpr::Header{{"Content-Type", "application/json"}}
    );

    Json::CharReaderBuilder reader;
    Json::Value responseBody;
    std::string errs;
    std::istringstream s(response.text);
    std::string taskId;

    if (Json::parseFromStream(reader, s, &responseBody, &errs)) {
        if (responseBody["errorId"].asInt() == 0) {
            taskId = responseBody["taskId"].asString();
        } else {
            std::cerr << "Error: " << responseBody["errorCode"].asString() << std::endl;
        }
    } else {
        std::cerr << "Failed to parse response: " << errs << std::endl;
    }

    return taskId;
}
```

### Step 2: Retrieving the reCAPTCHA Result

```cpp
#include <iostream>
#include <cpr/cpr.h>
#include <json/json.h>

std::string getTaskResult(const std::string& apiKey, const std::string& taskId) {
    Json::Value requestBody;
    requestBody["clientKey"] = apiKey;
    requestBody["taskId"] = taskId;

    Json::StreamWriterBuilder writer;
    std::string requestBodyStr = Json::writeString(writer, requestBody);

    while (true) {
        cpr::Response response = cpr::Post(
            cpr::Url{"https://api.capsolver.com/getTaskResult"},
            cpr::Body{requestBodyStr},
            cpr::Header{{"Content-Type", "application/json"}}
        );

        Json::CharReaderBuilder reader;
        Json::Value responseBody;
        std::string errs;
        std::istringstream s(response.text);

        if (Json::parseFromStream(reader, s, &responseBody, &errs)) {
            if (responseBody["status"].asString() == "ready") {
                return responseBody["solution"]["gRecaptchaResponse"].asString();
            } else if (responseBody["status"].asString() == "processing") {
                std::cout << "Task is still processing, waiting for 5 seconds..." << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(5));
            } else {
                std::cerr << "Error: " << responseBody["errorCode"].asString() << std::endl;
                break;
            }
        } else {
            std::cerr << "Failed to parse response: " << errs << std::endl;
            break;
        }
    }

    return "";
}
```

### Step 3: Bringing It All Together

```cpp
int main() {
    std::string apiKey = "YOUR_API_KEY";
    std::string websiteURL = "https://example.com";
    std::string websiteKey = "SITE_KEY";

    std::string taskId = createTask(apiKey, websiteURL, websiteKey);
    if (!taskId.empty()) {
        std::cout << "Task created successfully. Task ID: " << taskId << std::endl;
        std::string recaptchaResponse = getTaskResult(apiKey, taskId);
        std::cout << "reCAPTCHA Response: " << recaptchaResponse << std::endl;
    } else {
        std::cerr << "Failed to create task." << std::endl;
    }

    return 0;
}
```

### Advanced Use Case: Integrating Playwright and Undetected

If your project requires advanced browser automation or undetected solutions, you might consider combining this approach with Playwright or undetected Chromedriver.

- **Playwright**: Ideal for managing complex scenarios involving dynamic content or sophisticated CAPTCHA challenges.
- **Undetected**: Helps to bypass bot detection mechanisms that may interfere with solving reCAPTCHA challenges.

Although directly integrating these with C++ might require additional effort, using a cross-language solution (e.g., Python + C++ via `pybind11`) could be a powerful approach.

### Conclusion

This guide demonstrated how to solve reCAPTCHA in C++ using the [CapSolver API](https://docs.capsolver.com/guide/api-server.html?utm_source=github&utm_medium=repo&utm_campaign=recaptchac++). By following these steps, you can integrate reCAPTCHA solving into your C++ applications. For more advanced use cases, consider combining this approach with tools like Playwright or undetected for robust and stealthy CAPTCHA handling.
