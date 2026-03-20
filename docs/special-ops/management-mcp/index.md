---
tags:
    - MCP
    - Copilot Studio
    - Microsoft Agent 365
difficulty: 3
---

# 🏗️ Microsoft MCP Management Server

<!-- markdownlint-disable-next-line MD033 -->
<p align="center"><img src="./assets/badge.png" alt="MCP Management Server Badge" width="220" /></p>

> **Difficulty**: ⭐⭐⭐ | **Time**: ~45 min | **Type**: Special Ops

Welcome, agent. Your mission — should you choose to accept it — is **Operation Tool Smith**: connect to the **Microsoft MCP Management Server** from Visual Studio Code and craft a custom MCP server — without writing a single line of code. You'll wire up a remote API, add tools, publish your server, and put it to work inside a Copilot Studio agent. 🔨

## ❓ What Is the MCP Management Server?

The MCP Management Server is the **command center for building custom MCP servers**. Think of it as a factory that produces other factories: instead of writing code to stand up an MCP server, you call tools on the Management Server and it creates, configures, and publishes servers for you.

It's an **API-first** build surface — no UI required, everything is driven by tool calls. That means you can work directly from Visual Studio Code (or any MCP-compatible client), composing workflows through conversational agent interactions or direct API calls.

### Why Does This Matter?

Organizations often need specialized MCP servers for industry-specific or line-of-business workflows. Building one from scratch requires code, infrastructure, and deployment pipelines. The Management Server eliminates this overhead:

- **Create** an MCP server instance in seconds
- **Add tools** sourced from 1,500+ connectors (ServiceNow, JIRA, SAP, and more), Microsoft Graph APIs, Dataverse custom APIs, or any REST endpoint
- **Publish** the server so agents across the organization can use it
- **Manage** servers over time — update tools, descriptions, or delete servers when no longer needed

It's the **Swiss Army knife of MCP server creation** — one tool to build them all.

### Key Terms

| Term | Definition |
|------|------------|
| **MCP Management Server** | An MCP server whose tools create, update, and delete *other* MCP servers and their tools |
| **Work IQ** | The intelligence layer that grounds Microsoft 365 Copilot and your agents in real-time organizational context |
| **Remote API** | Any external REST endpoint that can be registered and exposed as a tool on your MCP server |
| **Connector** | A pre-built integration to an external service (for example, SharePoint, ServiceNow) |
| **Custom API** | A Dataverse-registered API that can be exposed as a tool on your MCP server |
| **Tooling Gateway** | The Agent 365 control plane that enforces governance, permissions, and observability for all MCP tool calls |

## 🎯 What You'll Build

- A **custom MCP server** created entirely through the Management Server (no code)
- **Tools** on that server powered by the [Open-Meteo](https://open-meteo.com/) remote API — a free weather API that requires no API key
- A **Copilot Studio agent** wired up to your freshly minted MCP server

## ⚙️ Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/download) installed
- A [Microsoft 365 Copilot license](https://learn.microsoft.com/copilot/microsoft-365/microsoft-365-copilot-licensing)
- Access to the [Frontier preview program](https://adoption.microsoft.com/copilot/frontier-program/)
- A Power Platform environment with Copilot Studio access
- **Tenant administrator** permissions (required to publish custom MCP servers)

> [!IMPORTANT]
> Currently, only tenant administrators can publish custom MCP servers within a tenant. The product team is working to enable this for all developers. If you don't have admin permissions, you can still follow along through the creation steps — you just won't be able to publish.

> [!NOTE]
> You need your Power Platform **environment ID** for this lab. To find it, go to the [Power Platform admin center](https://admin.powerplatform.microsoft.com/), select your environment, and copy the environment ID from the URL or environment details. For more details, see [Determine your environment ID](https://learn.microsoft.com/power-platform/admin/determine-org-id-name).

## 📋 The Scenario

> [!NOTE]
> **Scenario:** Contoso Weather Services is a logistics company that needs real-time weather data integrated into their agent workflows. Dispatchers currently switch between multiple apps to check conditions before routing deliveries. You are building a custom MCP server that exposes weather tools so a Copilot Studio agent can check forecasts and current conditions on behalf of dispatchers — without anyone writing code.

## 🚀 Mission Tasks

### Task 1: Connect to the MCP Management Server in Visual Studio Code

First, you need to connect Visual Studio Code to the MCP Management Server so you can start issuing commands.

1. Open **Visual Studio Code**

1. Open the command palette by pressing `Cmd` + `Shift` + `P` (macOS) or `Ctrl` + `Shift` + `P` (Windows/Linux)

1. Type `MCP` and select **MCP: Add Server**

1. Select **HTTP** as the server type

1. Enter the following URL, replacing `{environmentID}` with your Power Platform environment ID:

    ```text
    https://agent365.svc.cloud.microsoft/mcp/environments/{environmentID}/servers/MCPManagement
    ```

    > [!NOTE]
    > You need your Power Platform **environment ID** for this lab. To find it, go to the [Power Platform admin center](https://admin.powerplatform.microsoft.com/), select your environment, and copy the environment ID from the URL or environment details. For more details, see [Determine your environment ID](https://learn.microsoft.com/power-platform/admin/determine-org-id-name).

1. Enter `Management-MCP` as the server name

1. Select **Global** so the server is available across all projects

1. When prompted, sign in with your Microsoft account credentials

    > [!TIP]
    > Make sure you sign in with the account that has tenant administrator permissions and a Microsoft 365 Copilot license.

1. Verify the connection by opening **GitHub Copilot** in agent mode and asking:

    ```text
    List all MCP servers in my environment
    ```

    The Management Server should respond by calling the `GetMCPServers` tool and returning a list of existing servers (which may be empty if this is a fresh environment).

    <!-- TODO: Add screenshot of GetMCPServers response -->

### Task 2: Create a Custom MCP Server

Now that you're connected, it's time to forge your custom server.

1. In **GitHub Copilot** (agent mode), send the following message:

    ```text
    Create a new MCP server called "ContosoWeather" with the display name "Contoso Weather Services" and description "Provides real-time weather data for logistics routing decisions"
    ```

    The agent will call the `CreateMCPServer` tool with the parameters you provided.

    <!-- TODO: Add screenshot of CreateMCPServer call -->

1. Verify the server was created by asking:

    ```text
    Get the details of the ContosoWeather MCP server
    ```

    The agent calls the `GetMCPServer` tool and you should see your new server's details, including its name, display name, and description.

    <!-- TODO: Add screenshot of GetMCPServer response -->

> [!NOTE]
> The server name must not contain whitespaces. Use PascalCase or camelCase for the server name, and use the display name for a human-readable label.

### Task 3: Add a Remote API Tool

Time to give your server some capabilities. You'll use `CreateToolWithRemoteAPI` to add an [Open-Meteo](https://open-meteo.com/) weather tool. Open-Meteo is a free, open-source weather API that requires **no API key** — perfect for this lab.

> [!NOTE]
> The Open-Meteo API endpoint for current weather looks like this:
>
> ```text
> https://api.open-meteo.com/v1/forecast?latitude=47.6&longitude=-122.3&current=temperature_2m,weather_code,wind_speed_10m,relative_humidity_2m&temperature_unit=fahrenheit&wind_speed_unit=mph&timezone=auto
> ```
>
> It returns JSON with current temperature, weather code, wind speed, and humidity for the given coordinates. No authentication needed.

1. Register the Open-Meteo API as a remote API and add it as a tool on your server. Ask:

    ```text
    Create a tool called "GetCurrentWeather" on the ContosoWeather server using a remote API. The remote API endpoint is https://api.open-meteo.com/v1/forecast. Description: "Gets the current weather conditions for a location given latitude and longitude. Use query parameters: latitude, longitude, current=temperature_2m,weather_code,wind_speed_10m,relative_humidity_2m, temperature_unit=fahrenheit, wind_speed_unit=mph, timezone=auto"
    ```

    The agent calls `CreateToolWithRemoteAPI` to register the Open-Meteo endpoint and add it as a tool on your server.

    <!-- TODO: Add screenshot of CreateToolWithRemoteAPI call -->

1. Add a second tool for the daily forecast:

    ```text
    Create a tool called "GetDailyForecast" on the ContosoWeather server using a remote API. The remote API endpoint is https://api.open-meteo.com/v1/forecast. Description: "Gets a 7-day weather forecast for a location given latitude and longitude. Use query parameters: latitude, longitude, daily=temperature_2m_max,temperature_2m_min,precipitation_sum,weather_code, temperature_unit=fahrenheit, timezone=auto"
    ```

    <!-- TODO: Add screenshot -->

1. Verify both tools were added:

    ```text
    Get all tools in the ContosoWeather MCP server
    ```

    The agent calls `GetTools` and you should see your two weather tools listed.

    <!-- TODO: Add screenshot of GetTools response -->

> [!NOTE]
> You can add tools from many different sources — connectors (1,500+), Microsoft Graph APIs, Dataverse custom APIs, SDK messages, and remote APIs. This mission uses a remote API, but the process is similar for other tool types. Check the [MCP Management Server reference](https://learn.microsoft.com/microsoft-agent-365/mcp-server-reference/mcpmanagement) for all available `CreateToolWith*` operations.

### Task 4: Publish the MCP Server

Your server has tools — now let's make it available to agents across your organization.

1. Ask the agent to publish:

    ```text
    Publish the ContosoWeather MCP server
    ```

    The agent calls the `PublishMCPServer` tool. Once published, the server becomes available in the MCP server catalog in Copilot Studio and Microsoft Foundry.

    <!-- TODO: Add screenshot of PublishMCPServer call -->

> [!WARNING]
> Publishing makes the server available to all agents in your tenant. Make sure you're comfortable with the tools exposed before publishing. You can use `BlockMCPServer` to block a published server if needed.

### Task 5: Use Your Custom MCP Server in Copilot Studio

Now it's time to see your creation in action. You'll connect a Copilot Studio agent to the server you just built.

1. Go to [Copilot Studio](https://copilotstudio.microsoft.com/)

1. Select the environment picker at the top right corner and select the same environment you used for the Management Server

1. Select **Agents** in the left navigation

1. Select the **Create blank agent** button

    <!-- TODO: Add screenshot -->

1. When provisioning is complete, select **Edit** in the details card and change the name to:

    ```text
    Weather Dispatcher
    ```

1. Add the following **Description**:

    ```text
    A logistics assistant that provides real-time weather data to help dispatchers make informed routing decisions.
    ```

1. Add the following **Instructions**:

    ```text
    You are a weather-aware logistics assistant for Contoso Weather Services. Help dispatchers check current weather conditions and forecasts to make smart routing decisions. When asked about weather, use the available weather tools to get real-time data. Present weather information clearly and concisely, highlighting any conditions that might affect deliveries (rain, snow, extreme heat, high winds). Always include the location and time context in your response.
    ```

1. Select **Save**

1. Select **Tools** in the top menu

1. Select the blue **Add a tool** button

1. Select **Model Context Protocol** to filter for MCP servers

1. Find your **Contoso Weather Services** server in the list and select it

    > [!TIP]
    > If you don't see your server, make sure you published it in Task 4 and that you're in the same environment.

    <!-- TODO: Add screenshot of finding the server -->

1. Select **Not connected** and then **Create new Connection**

1. Select **Create** and provide your credentials

1. Select **Add and configure** to add the tools to your agent

    <!-- TODO: Add screenshot of tools added -->

1. Select the **+ icon** in the **Test your agent** pane to start a new testing session

1. Send the following message:

    ```text
    What's the current weather in Seattle?
    ```

    The agent should use your `GetCurrentWeather` tool and return real-time weather data from Open-Meteo.

    <!-- TODO: Add screenshot of weather response -->

1. Try the forecast tool:

    ```text
    What's the 7-day weather forecast for Seattle?
    ```

    <!-- TODO: Add screenshot of forecast response -->

Congratulations — your custom MCP server is live and powering an agent! 🎉

### Task 6: Clean Up (Optional)

If you want to remove the server after testing:

1. Go back to **Visual Studio Code** and ask:

    ```text
    Delete the ContosoWeather MCP server
    ```

    The agent calls `DeleteMCPServer` to remove the server. This also removes all tools associated with it.

    > [!WARNING]
    > Deleting an MCP server is permanent. Any agents using this server will lose access to its tools.

## 🏅 Mission Accomplished

<!-- markdownlint-disable-next-line MD033 -->
<p align="center"><img src="./assets/badge.png" alt="MCP Management Server Badge" width="220" /></p>

Congrats, agent — **Operation Tool Smith** is complete! You've crafted a custom MCP server from the ground up using nothing but tool calls. Here's what you've mastered:

✅ **MCP Management Server connection**: Connected to the Management Server from Visual Studio Code and authenticated with your Microsoft account

✅ **Custom server creation**: Created an MCP server instance using the `CreateMCPServer` tool — no code, no infrastructure

✅ **Tool composition**: Added remote API tools to your server using `CreateToolWithRemoteAPI`, wiring up the Open-Meteo weather API without any authentication or code

✅ **Server publishing**: Published your server to make it available across your organization

✅ **Agent integration**: Connected a Copilot Studio agent to your custom server and tested it with real-time data

## 🏅 Claim Your Completion Badge

Simply submit the badge request form and answer all required questions:

[https://aka.ms/agent-academy-special-ops/management-mcp/form](https://aka.ms/agent-academy-special-ops/management-mcp/form)

Once your submission is reviewed, you will receive an email from Global AI Community with instructions to claim your badge.

> [!TIP]
> If you don't see the email, check your spam or junk folder.

## 📚 Tactical Resources

- 📖 [MCP Management Server reference](https://learn.microsoft.com/microsoft-agent-365/mcp-server-reference/mcpmanagement)
- 📖 [Work IQ MCP overview](https://learn.microsoft.com/microsoft-agent-365/tooling-servers-overview)
- 📖 [Build custom MCP servers with the Management Server](https://learn.microsoft.com/microsoft-agent-365/tooling-servers-overview#build-scenario-focused-custom-mcp-servers-with-the-microsoft-mcp-management-server)
- 📖 [Open-Meteo Weather API](https://open-meteo.com/en/docs) — Free weather API used in this lab (no API key required)
- 📖 [Model Context Protocol overview](https://modelcontextprotocol.io/introduction)

<!-- markdownlint-disable-next-line MD033 -->
<img src="https://m365-visitor-stats.azurewebsites.net/agent-academy/special-ops/management-mcp" alt="Analytics" />
