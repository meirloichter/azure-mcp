# Contributing to Azure MCP

There are many ways to contribute to the Azure MCP project: reporting bugs, submitting pull requests, and creating suggestions.
After cloning and building the repo, check out the [github project](https://github.com/orgs/Azure/projects/812/views/13) and [issues list](https://github.com/Azure/azure-mcp/issues).  Issues labeled [help wanted](https://github.com/Azure/azure-mcp/labels/help%20wanted) are good issues to submit a PR for.  Issues labeled [good first issue](https://github.com/Azure/azure-mcp/labels/good%20first%20issue) are great candidates to pick up if you are in the code for the first time.
>[!IMPORTANT]
If you are contributing significant changes, or if the issue is already assigned to a specific milestone, please discuss with the assignee of the issue first before starting to work on the issue.

## Prerequisites

1. Install either the stable or Insiders release of VS Code.
   * [Stable release](https://code.visualstudio.com/download)
   * [Insiders release](https://code.visualstudio.com/insiders)
2. Install [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) and [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) extensions.
3. Install [Node.js](https://nodejs.org/en/download) 20 or later
   * Ensure `node` and `npm` are in your path
4. Install [PowerShell](https://learn.microsoft.com/powershell/scripting/install/installing-powershell) 7.0 or later.
   * Optional for coding but required for running build, test and resource deployment scripts

### Project Structure

The project is organized as follows:
- `src/` - Main source code
  - `Areas/{Area}/` - Service specific code
    - `Commands/` - Command implementations
    - `Models/` - Service specific models
    - `Services/` - Service implementations and interfaces
    - `Options/` - Service specific command options
  - `Commands/` - Command base and helper classes
  - `Models/` - Common models and base classes
  - `Services/` - Common services
  - `Options/` - Command option definitions
- `tests/` - Test files
  - `Areas/{Area}/` - Service specific tests
    - `UnitTests/` - Unit tests require no authentication or test resources
    - `LiveTests/` - Live tests depend on Azure resources and authentication
- `docs/` - Documentation

### Adding a New Command

1. Create a new issue in this repository with:
   - Title: "Add command: azmcp [service] [resource] [operation]"
   - Description: Detailed explanation of what the command will do

2. Set up your development environment:
   - Open VS Code Insiders
   - Open the Copilot Chat view
   - Select "Agent" mode

3. Use Copilot to generate the command:
   ```
   Execute in Copilot Chat:
   "create [service] [resource] [operation] command using #new-command.md as a reference"
   ```

4. Follow the implementation guidelines in [src/Docs/new-command.md](https://github.com/Azure/azure-mcp/blob/main/src/Docs/new-command.md)

5. Add documentation for new command:
   - [azmcp-commands.md](https://github.com/Azure/azure-mcp/blob/main/docs/azmcp-commands.md)
   - [README.md](https://github.com/Azure/azure-mcp/blob/main/README.md)

6. Add CODEOWNERS entry for new command:
   - [CODEOWNERS](https://github.com/Azure/azure-mcp/blob/main/.github/CODEOWNERS) [(example PR)](https://github.com/Azure/azure-mcp/commit/08f73efe826d5d47c0f93be5ed9e614740e82091)

7. Create a Pull Request:
   - Reference the issue you created
   - Include tests in the `/tests` folder
   - Ensure all tests pass
   - Add sample prompts to `/e2eTests/e2eTestPrompts.md`
   - Follow the code style requirements

## Development Process

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Write or update tests
5. Test Locally
6. Submit a pull request

## Testing

Command authors must provide both of the following test types:
- End-to-end test prompts
- Unit tests

### End-to-end test

End-to-end tests are currently performed manually. Command authors must thoroughly test each command to ensure correct tool invocation and results. At least one prompt per tool is required and should be added to `/e2eTests/e2eTestPrompts.md`.

### Unit tests

Unit tests live under the `/tests` folder. To run tests:

```pwsh
# Run tests with coverage
./eng/scripts/Test-Code.ps1
```

Unit test requirements:
- Each command should have unit tests
- Tests should cover success and error scenarios
- Mock external service calls
- Test argument validation

### Testing Local Build with VS Code

To run the Azure MCP server from source for local development:

#### 1. Build the Server

Navigate to the MCP server source directory and build the project using the .NET CLI:

```bash
dotnet build
```

#### 2. Configure mcp.json in IDE

Update your mcp.json to point to the locally built azmcp executable. This setup uses stdio as the communication transport.

```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start"
      ]
    }
  }
}
```

Optional `--namespace` and `--mode` parameters can be used to configure different server modes:

**Default Mode** (no additional parameters):
```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start"
      ]
    }
  }
}
```

**Namespace Mode** (expose specific services):
```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start",
        "--namespace",
        "storage",
        "--namespace",
        "keyvault"
      ]
    }
  }
}
```

**Namespace Proxy Mode** (collapse tools by namespace - useful for VS Code's 128 tool limit):
```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start",
        "--mode",
        "namespace"
      ]
    }
  }
}
```

**Single Tool Proxy Mode** (single "azure" tool with internal routing):
```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start",
        "--mode",
        "single"
      ]
    }
  }
}
```

**Combined Mode** (filter namespaces with proxy mode):
```json
{
  "servers": {
    "azure-mcp-server": {
      "type": "stdio",
      "command": "<absolute-path-to>/azure-mcp/src/bin/Debug/net9.0/azmcp[.exe]",
      "args": [
        "server",
        "start",
        "--namespace",
        "storage",
        "--namespace",
        "keyvault",
        "--mode",
        "namespace"
      ]
    }
  }
}
```

> **Note:** Replace `<absolute-path-to>` with the full path to your built executable.
> On **Windows**, use `azmcp.exe`.
> On **macOS/Linux**, use `azmcp` (without the `.exe` extension).

> **Note:** For namespace mode, replace `<service-name>` with available top level command groups.
> Run `azmcp -h` to review available services. Examples include `storage`, `keyvault`, `cosmos`, `monitor`, etc.
>
> **Server Modes:**
>
> * **Default Mode**: No additional parameters - exposes all tools individually
> * **Namespace Mode**: `--namespace <service-name>` - expose specific services (can use multiple `--namespace` parameters)
> * **Namespace Mode**: `--mode namespace` - collapse tools by namespace (useful for VS Code's 128 tool limit)
> * **Single Tool Mode**: `--mode single` - single "azure" tool with internal routing
> * **Combined Mode**: Both `--namespace` and `--mode` can be used together to filter namespaces and use mode

#### 3. Start from IDE or Tooling

With the configuration in place, you can launch the MCP server directly from your IDE or any tooling that uses `mcp.json`.

### Configuring External MCP Servers

The Azure MCP Server supports connecting to external MCP servers through an embedded `registry.json` configuration file. This enables the server to act as a proxy, aggregating tools from multiple MCP servers into a single interface. The registry follows the same configuration schema as VS Code's `mcp.json`.

#### Registry Configuration

External MCP servers are defined in the embedded resource file `src/Areas/Server/Resources/registry.json`. This file contains server configurations that support both SSE (Server-Sent Events) and stdio transport mechanisms, following the standard MCP configuration format.

The registry structure follows this format:

```json
{
  "servers": {
    "server-id": {
      "url": "https://example.com/api/mcp",
      "description": "Description of what this server provides"
    },
    "another-server": {
      "type": "stdio",
      "command": "path/to/executable",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      },
      "description": "Another MCP server using stdio transport"
    }
  }
}
```

#### Transport Types

**SSE (Server-Sent Events) Transport:**

* Use the `url` property to specify the endpoint
* Supports HTTP-based communication with automatic transport mode detection
* Best for web-based MCP servers and remote endpoints

**Stdio Transport:**

* Use `type: "stdio"` with the `command` property
* Supports launching external processes that communicate via standard input/output
* Use `args` array for command-line arguments
* Use `env` object for environment variables
* Best for local executables, command-line tools, and local MCP servers

#### Server Discovery and Namespace Filtering

External servers are automatically discovered when the Azure MCP Server starts. They can be filtered using the same namespace mechanisms as built-in commands:

```bash
# Include only specific external servers
azmcp server start --namespace documentation --namespace another-server

# Use namespace mode to group tools exposed by external servers
azmcp server start --mode namespace
```

#### Adding New External MCP Servers

To add a new external MCP server to the registry:

1. Edit `src/Areas/Server/Resources/registry.json`
2. Add your server configuration under the `servers` object using VS Code's MCP configuration schema
3. Use a unique identifier as the key
4. Provide either a `url` for SSE transport or `type: "stdio"` with `command` for stdio transport
5. Include a descriptive `description` field
6. Rebuild the project to embed the updated registry

#### Example External Servers

The current registry includes:

* **documentation**: Microsoft Learn documentation search via SSE transport
* Additional external servers can be added following the same pattern as VS Code's mcp.json

External servers integrate seamlessly with the Azure MCP Server's tool aggregation, appearing alongside native Azure commands in the unified tool interface. This allows you to combine local MCP servers, remote MCP endpoints, and Azure-specific tools in a single interface.

### Live Tests

> ⚠️ If you are a Microsoft employee with Azure source permissions then please review our [Azure Internal Onboarding Documentation](https://aka.ms/azmcp/intake).  Team members can run live tests by adding this comment to the PR `/azp run azure - mcp` to start the run.

Before running live tests,
- [Install Azure Powershell](https://learn.microsoft.com/powershell/azure/install-azure-powershell)
- [Install Azure Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/install#install-manually)
- Login to Azure-Powershell
  - [`Connect-AzAccount`](https://learn.microsoft.com/powershell/azure/authenticate-interactive?view=azps-13.4.0)
- Deploy test resources
    ```pwsh
    ./eng/scripts/Deploy-TestResources.ps1
    ```

    | Parameter           | Type     | Description                                                                                                                   |
    |---------------------|----------|-------------------------------------------------------------------------------------------------------------------------------|
    | `Areas`             | string[] | Reduce the scope of your deployment to specific areas.  e.g. `-Areas Storage, KeyVault`                                       |
    | `SubscriptionId`    | string   | Deploy to a specific subscription, otherwise, for internal users, the subscription will be defaulted to a known subscription. |
    | `ResourceGroupName` | string   | Set the resource group name. Defaults to "{username}-mcp{hash(username)}".                                                    |
    | `BaseName`          | string   | Set the base name for all of the resources. Defaults to "mcp{hash}".                                                          |
    | `Unique`            | switch   | Make `{hash}` in the resource group name and base name unique per invocation. Defaults to a hash of your username             |
    | `DeleteAfterHours`  | int      | Change the timespan used to set  the DeleteAfter tag. Defaults to 12 hours.                                                   |


After deploying test resources, you should have a `.testsettings.json` file with your deployment information in the root of the repo.

You can run live tests with:
```pwsh
./eng/scripts/Test-Code.ps1 -Live
```

`Test-Code.ps1` supports the `-Areas` parameter as well
```pwsh
./eng/scripts/Test-Code.ps1 -Live -Areas Storage, KeyVault
```

### `npx` live tests

You can set the `TestPackage` parameter in `.testsettings.json` to have live tests run `npx` targeting an arbitrary Azure MCP package.

```json
{
  "TenantId": "a20062a8-ff76-41c2-8a6d-5e843da7b051",
  "TenantName": "Your Tenant",
  "SubscriptionId": "cd27afdc-9976-4f08-96e9-cad120a91560",
  "SubscriptionName": "Your Subscription",
  "ResourceGroupName": "rg-abcdefg",
  "ResourceBaseName": "t1234567890",
  "TestPackage": "@azure/mcp@0.0.10"
}
```

To run live tests against the local build of an npm module, as opposed to the bin folder's azmcp executable, run:
```pwsh
./eng/scripts/Build-Local.ps1
```

This will produce .tgz files in the `.dist` directory and set the `TestPackage` parameter in the `.testsettings.json` file.

```json
  "TestPackage": "file://D:\\repos\\azure-mcp\\.dist\\wrapper\\azure-mcp-0.0.12-alpha.1746488279.tgz"
```

### Debugging live tests

This section assumes that the necessary Azure resources for live tests are already deployed and that the `.testsettings.json` file with deployment information is located at the root of the local repository clone. Refer to the previous section for instructions if these steps have not been completed.

To debug the Azure MCP Server (`azmcp`) when running live tests in VS Code, follow these steps

1. Build the package with debug symbols by running `./eng/scripts/Build-Local.ps1 -DebugBuild`.
2. Set a breakpoint in a command file (e.g., [`KeyValueListCommand.ExecuteAsync`](https://github.com/Azure/azure-mcp/blob/4ed650a0507921273acc7b382a79049809ef39c1/src/Commands/AppConfig/KeyValue/KeyValueListCommand.cs#L48)).
3. In VS Code, navigate to a test method (e.g., [`AppConfigCommandTests::Should_list_appconfig_kvs()`](https://github.com/Azure/azure-mcp/blob/4ed650a0507921273acc7b382a79049809ef39c1/tests/Client/AppConfigCommandTests.cs#L56)), add a break point to `CallToolAsync` call in the test method then right-click select **Debug Test** . This will launch `azmcp` as a separate .NET process and trigger the breakpoint in the test method.
4. Find the `azmcp` process ID

```shell
pgrep -fl azmcp
```

```powershell
Get-Process | Where-Object { $_.ProcessName -like "*azmcp*" } | Select-Object Id, ProcessName, Path
```

5. Open the Command Palette (`Cmd+Shift+P` on Mac, `Ctrl+Shift+P` on Windows/Linux), select **Debug: Attach to .NET 5+ or .NET Core process**, and enter the `azmcp` process ID.

6. Hit F5 to "Continue" debugging, the debugger should attach to `azmcp` and hit the breakpoint in command file.


## Code Style

To ensure consistent code quality, code format checks will run during all PR and CI builds.

To catch format errors early, run `dotnet format` before submitting.

- Follow C# coding conventions
- No comments in implementation code (code should be self-documenting)
- Use descriptive variable and method names
- Follow the exact file structure and naming conventions
- Use proper error handling patterns
- XML documentation for public APIs
- Follow Model Context Protocol (MCP) patterns

## AOT Compatibility Analysis

The AOT compatibility analysis helps identify potential issues that might prevent the Azure MCP Server from working correctly when compiled with AOT or when trimming is enabled.

### Running the Analysis

To run the AOT compatibility analysis locally:

```pwsh
./eng/scripts/Analyze-AOT-Compact.ps1
```

The HTML report will be generated at `.work/aotCompactReport/aot-compact-report.html` and automatically opened in your default browser.

To output the report to console, run the analysis with `-OutputFormat Console` argument.

AOT compatibility warnings typically indicate:

- Use of reflection without proper annotations
- Serialization of types that might be trimmed
- Dynamic code generation
- Use of `RequiresUnreferencedCodeAttribute` methods without proper precautions

### Installing Git Hooks

You can install our pre-push hook to catch code format issues by automatically running `dotnet format` before each `git push`.

- `./eng/scripts/Install-GitHooks.ps1` - Installs the pre-push hook into your local repo
- `./eng/scripts/Remove-GitHooks.ps1` - Disables any git hooks in your local repo

## Model Context Protocol (MCP)

The Azure MCP Server implements the [Model Context Protocol specification](https://modelcontextprotocol.io). When adding new commands:

- Follow MCP JSON schema patterns
- Implement proper context handling
- Use standardized response formats
- Handle errors according to MCP specifications
- Provide proper argument suggestions

## Pull Request Process

1. Update documentation reflecting any changes
2. Add or update tests as needed
3. Reference the original issue
4. Wait for review and address any feedback

## Builds and releases (Internal)

The internal pipeline [azure-mcp](https://dev.azure.com/azure-sdk/internal/_build?definitionId=7571) is used for all official
releases and CI builds. On every merge to main, a build will run and will produce a dynamically named prerelease
package on the public dev feed, e.g. [@azure/mcp@0.0.10-beta.4799791](https://dev.azure.com/azure-sdk/public/_artifacts/feed/azure-sdk-for-js/Npm/@azure%2Fmcp/overview/0.0.10-beta.4799791).

Only manual runs of the pipeline sign and publish packages.  Building `main` or `hotfix/*` will publish to `npmjs.com`, all other refs will publish to the [public dev feed](https://dev.azure.com/azure-sdk/public/_artifacts/feed/azure-sdk-for-js).

Packages published to the npmjs.com will always use the `@latest` [dist-tag](https://docs.npmjs.com/downloading-and-installing-packages-locally#installing-a-package-with-dist-tags).

Packages published to the dev feed will use:
- `@latest` for the latest official/release build
- `@dev` for the latest CI build of main
- `@pre` for any arbitrary pipeline run or feature branch build

### PR Validation

To run live tests for a PR, inspect the PR code for any suspicious changes, then add the comment `/azp run azure - mcp` to the pull request.  This will queue a PR triggered run which will build, run unit tests, deploy test resources and run live tests.

If you would like to see the product of a PR as a package on the dev feed, after thoroughly inspecting the change, create a branch in the main repo and manually trigger an [azure - mcp](https://dev.azure.com/azure-sdk/internal/_build?definitionId=7571) pipeline run against that branch. This will queue a manually triggered run which will build, run unit tests, deploy test resources, run live tests, sign and publish the packages to the dev feed.

Instructions for consuming the package from the dev feed can be found in the "Extensions" tab of the pipeline run page.

## Questions and Support

- Create an issue for questions
- Tag @jongio for specific help or clarification
- Join our community discussions

## Additional Resources

- [Azure MCP Documentation](https://github.com/Azure/azure-mcp/blob/main/README.md)
- [Command Implementation Guide](https://github.com/Azure/azure-mcp/blob/main/src/Docs/new-command.md)
- [VS Code Insiders Download](https://code.visualstudio.com/insiders/)
- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)

## Code of Conduct

This project adheres to a standard code of conduct. By participating, you are expected to uphold this code.

## License

By contributing, you agree that your contributions will be licensed under the project's license.
