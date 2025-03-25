

## How does .NET Core achieve cross-platform compatibility?
- **1) Cross-Platform Runtime (CoreCLR)**: .NET Core uses a lightweight & modular runtime called CoreCLR, which is designed to run on various OS. It abstract platform-specifc details so same .NET code can run on any platform.  
- **2) Core API (Platform-specific Implementations)**: .Net core internally include platform-specific implementation of libraries & API (like file handling, networking, or threading) which implement differently depending on OS. This makes these libraries to expose ***abstract layer*** which perform platform-independent as internally handled by CoreCLR. Example:-  
  - Windows uses backslashes (\\) for file paths, while Linux and macOS use forward slashes (/) but .Net Core provides cross-platform APIs (e.g., `System.IO.Path`) to handle these file paths.
  - Windows manages threads and processes differently from Linux or macOS, but in .Net Core, threading and process APIs like `System.Threading.Thread` or `System.Diagnostics.Process` abstract over these platform-specific differences. However, internally, platform-specific code ensures the correct behavior on each platform.
- **3) Open-Source and Modular Design**: .Net core in open-source, as its source-code available on GitHub. It is also modular so we can include only parts of .net core that our application need.  
- **4) .NET Core CLI**: .Net Core CLI allows developers to create, build, run, and deploy applications from the command line, regardless of their OS. The same commands work the same way on Windows, macOS, and Linux.
- **5) Docker & Container support**: - .Net core is compatible with Docker & Containerized environment, which make it consistent to deploy application in different environments (cloud, on-premise, & different OSes).
- **6) Platform Detection at Runtime**: It provides APIs to detect platform at runtime, which allow to write platform-specific code where needed. Ex: `System.Runtime.InteropServices.RuntimeInformation`
```csharp
if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
{
    // Windows-specific logic
}
else if (RuntimeInformation.IsOSPlatform(OSPlatform.Linux))
{
    // Linux-specific logic
}
```
