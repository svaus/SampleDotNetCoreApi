# SampleDotNetCoreApi
Sample DotNet Core Api to setup the project

```
dotnet new webapi
```

## Add Serilog Logging to project 

Add below packages to solution/project

```
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Settings.Configuration
```

Modify the Program.cs file as below. Added method UseSerilog which is injecting log configuration from configuration file i.e. appsettings.json 

```
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .UseSerilog((hostingContext, loggerConfiguration) => {
                    loggerConfiguration
                    .ReadFrom
                    .Configuration(hostingContext.Configuration)
                    .Enrich.FromLogContext()
                    .WriteTo.Console();
                });
```


Now add the below section in appsettings.json file at root level of appsetting json : 

```
"Serilog": {
      "Using": [ "Serilog.Sinks.Console" ],
      "MinimumLevel": "Debug",
      "WriteTo": [
        { "Name": "Console" }        
      ],
      "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ],
      "Properties": {
      "Application": "SampleDotNetCoreApi"
      }
    }
```


In configure method of Startup.cs class add below code in the beginning and also inject ILoggerFactory. This ILoggerFactory is injected by AspNetCore dependency injection.

```
loggerFactory.AddSerilog();
```

### Add logging to Rolling File using Serilog

If you want to do logging to file, Serilog provides sink for RollingFile.

```
dotnet add package Serilog.Sinks.RollingFile
```

And to appsettings.json file add the section for RollingFile. There is no code required for this. Serilog will read the configuration and apply the settings. The only thing which is required is adding the package.

```
"WriteTo": [
        { "Name": "Console" },
        {
        "Name": "RollingFile",
          "Args": {
          "pathFormat": "logs\\log-{Date}.txt",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level}] {Message}{NewLine}{Exception}"
          }
        }
      ],
```