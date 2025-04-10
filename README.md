# Databricks Genie API MCP Server

This project implements a Model Context Protocol (MCP) server that exposes Databricks Genie API capabilities as tools. It allows you to integrate Databricks' no-code AI/BI assistant features with other applications through a standardized interface, enabling powerful natural language querying of your Databricks data.

For a detailed explanation and example use cases, please see the accompanying [blog post](../genie_mcp_blog.md).

## Features

- Expose Databricks Genie API functions as MCP tools.
- Enable natural language querying of Databricks data.
- Start and manage Genie conversations.
- Create and retrieve messages.
- Execute and fetch SQL query results generated by Genie.
- Secure authentication with Databricks.

## Prerequisites

- Python 3.10+
- Databricks workspace with Genie access and System Tables enabled (if using them).
- Databricks Assistant enabled.
- CAN USE permission on a Pro or Serverless SQL warehouse.
- Access to Unity Catalog data relevant to your Genie Space.
- An MCP-compatible client application, such as [Claude Desktop](https://www.anthropic.com/claude-on-desktop).

## Setup Instructions

1.  **Clone the Repository** (if you haven't already)
    ```bash
    # Add clone command if needed
    ```

2.  **Navigate to the Server Directory**
    ```bash
    cd genie_api 
    ```

3.  **Install Dependencies**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Authentication**

    Set the required environment variables for the Databricks SDK to connect to your workspace. Create a `.env` file in this directory (`genie_api/`) or set them globally:

    ```bash
    # --- .env file content ---

    # For PAT authentication (recommended for development)
    # DATABRICKS_HOST=https://your-workspace.cloud.databricks.com
    # DATABRICKS_TOKEN=your-personal-access-token

    # Or for OAuth with service principal (recommended for production)
    DATABRICKS_HOST=https://your-workspace.cloud.databricks.com
    DATABRICKS_CLIENT_ID=your-client-id
    DATABRICKS_CLIENT_SECRET=your-client-secret

    # --- end .env file content ---
    ```
    *Ensure the `.env` file is added to your `.gitignore!*

5.  **Run the Server Locally**
    ```bash
    python server.py
    ```
    The server will start and listen for connections via standard input/output (stdio).

## Using with Claude Desktop

This MCP server is designed to be used with MCP clients like Claude Desktop. Follow these steps to connect:

1.  **Install Claude Desktop:** Download and install from the [official website](https://www.anthropic.com/claude-on-desktop).
2.  **Configure Claude Desktop:**
    *   Open Claude Desktop settings (Menu Bar -> Claude -> Settings...).
    *   Go to Developer -> Edit Config.
    *   Add the following entry to the `mcpServers` object in the `claude_desktop_config.json` file, adjusting paths as needed:

    ```json
    {
      "mcpServers": {
        "databricks-genie": {
          "command": "python", // Or python3, or the full path to your python executable
          "args": [
            "/full/absolute/path/to/your/project/genie_api/server.py" 
          ],
          "workingDirectory": "/full/absolute/path/to/your/project/genie_api/" 
        }
        // ... potentially other servers ...
      }
    }
    ```
    *   **Important:** Use the **full absolute path** to `server.py` and the `genie_api` directory.
    *   Ensure the Python `command` is accessible by Claude Desktop.
    *   The `workingDirectory` ensures the server can find `auth.py` and your `.env` file.
3.  **Restart Claude Desktop:** Close and reopen the application.
4.  **Verify:** Click the hammer icon (Tools) in the chat input. You should see the `databricks-genie` tools listed (e.g., `start_conversation`, `create_message`).

Now you can ask Claude questions like "What was our DBU consumption last month?" or "Who accessed the PII table yesterday?", and it will use the tools provided by your local server.

For more details on configuring Claude Desktop, see the [MCP Quickstart for Claude Desktop Users](https://modelcontextprotocol.io/quickstart/user).

## Available Tools

- `start_conversation`: Start a new conversation in a Genie space.
- `create_message`: Create a new message in an existing conversation.
- `get_message`: Retrieve a message from a conversation.
- `get_message_attachment_query_result`: Get SQL query results from a message attachment.
- `execute_message_attachment_query`: Execute SQL for a message query attachment.
- `get_space`: Get details about a Genie space.
- `generate_download_full_query_result`: Initiate a full query result download.
- `poll_message_until_complete`: Poll a message until it reaches a terminal state.

## Troubleshooting

- **Authentication Issues**: Verify Databricks credentials (`.env` file or environment variables) and required permissions in Databricks.
- **Connection Problems (Claude Desktop)**:
    - Ensure absolute paths in `claude_desktop_config.json` are correct.
    - Verify the `command` points to a valid Python interpreter.
    - Check Claude Desktop logs (`~/Library/Logs/Claude/` or `%APPDATA%\Claude\logs`). Look for `mcp.log` and `mcp-server-databricks-genie.log`.
    - Try running `python server.py` manually in your terminal from the `genie_api` directory to check for errors.
- **SQL Execution Errors**: Ensure the service principal or user has CAN USE on a SQL warehouse and access to the relevant Unity Catalog data.

## Security Considerations

- **Credentials:** Never hardcode credentials. Use environment variables (`.env`) or secure credential stores. Ensure `.env` is in your `.gitignore`. Use OAuth with service principals for production.
- **MCP Security:** This server runs locally with your user's permissions and Databricks credentials. Clients like Claude Desktop **MUST** obtain user consent before executing tools ([MCP Security Specification](https://modelcontextprotocol.io/specification/2025-03-26/index)).
- **Production Deployment:** Running this server for broader use requires a secure hosting strategy. **Do not simply expose this local server.** Work with security/DevOps to determine appropriate hosting, network controls, and potentially MCP server-level authentication. The hosting environment needs secure access to Databricks credentials (e.g., instance profiles, managed secrets). See the [blog post](../genie_mcp_blog.md) for more discussion.
- **Input Sanitization:** Trust the Databricks SDK/API for input handling passed via tools.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This software is provided under a specific license granted by Databricks, Inc. Please see the [LICENSE](LICENSE) file for the full terms and conditions governing your use of this software. 
