# Blazor Web Worker / Ignite UI initializer repro

A minimal .NET 11 standalone Blazor WebAssembly app using **Ignite UI Blazor Lite 1.0.1** and the unmodified `blazorwebworker` template.

`Client` references the Ignite UI package and `Worker`. **`Worker` does not reference Ignite UI.** The page renders an Ignite UI Combo Box and automatically calls the worker's generated `Greet` method.

## Run

Requires the .NET 11 SDK.

```powershell
dotnet run --project Client\BlazorWebWorkerIgniteUIRepro.Client.csproj --no-launch-profile -- --urls http://localhost:5000
```

Open `http://localhost:5000` and the browser developer console. Reload if the console was not open during startup.

## Actual behavior

The Combo Box initializes in the page, but the worker also imports the client app's Ignite UI library initializer and its DOM-dependent JavaScript dependencies. The browser logs:

```text
MONO_WASM: Failed to import library initializer '[object Object]': ReferenceError: document is not defined
```

The exception originates in `_content/IgniteUI.Blazor/lit-html.js`. The worker still returns `Hello, World!`: the runtime catches and logs the import failure rather than failing worker startup. A debugger configured to break on JavaScript exceptions can stop at the underlying exception.

## Why this happens

The generated worker imports the client app's `dotnet.js` and calls `dotnet.create()`. That reuses the client's boot configuration, including its `resources.modulesAfterRuntimeReady` UI initializers. Selecting a worker assembly with `getAssemblyExports` does not isolate runtime initialization.

The worker files are unchanged from `dotnet new blazorwebworker`; there is no workaround in this repro. No Aspire, Syncfusion, custom initializer, or additional UI library is involved.

## Expected behavior

A supported, discoverable way to start a computation-only worker without importing DOM-dependent UI initializers inherited from the client app, while preserving worker-compatible runtime initializers.

See the [.NET 11 Web Worker guidance](https://learn.microsoft.com/aspnet/core/blazor/blazor-with-dotnet-on-web-workers?view=aspnetcore-11.0).
