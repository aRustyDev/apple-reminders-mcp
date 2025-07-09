# Prompt: Build an MVP macOS Reminders MCP Server

  ## Project Overview
  Create a minimal MCP (Model Context Protocol) server that provides access to macOS Reminders through the EventKit framework. The server should run as a
   background daemon and be written primarily in Swift.

  ## MVP Scope
  The MVP will support these core operations:
  - Create a new reminder with title and optional due date
  - List reminders from the default list
  - Mark a reminder as complete
  - Get all reminder lists

  ## Implementation Steps

  ### Step 1: Project Setup
  1. Create a new Swift package: `swift package init --type executable --name RemindersMLPServer`
  2. Add MCP server dependencies to Package.swift
  3. Create the basic project structure:
     RemindersMLPServer/
     ├── Sources/
     │   ├── RemindersMLPServer/
     │   │   ├── main.swift
     │   │   ├── MCPServer.swift
     │   │   ├── RemindersAPI.swift
     │   │   └── Models.swift
     ├── Package.swift
     └── LaunchDaemon/
         └── com.user.reminders-mcp.plist

  ### Step 2: Implement Reminders API Wrapper
  1. Create `RemindersAPI.swift` with EventKit integration
  2. Request calendar access permissions
  3. Implement core functions:
  - `createReminder(title: String, notes: String?, dueDate: Date?) -> Result<String, Error>`
  - `listReminders(listName: String?, includeCompleted: Bool) -> Result<[Reminder], Error>`
  - `completeReminder(id: String) -> Result<Bool, Error>`
  - `getLists() -> Result<[ReminderList], Error>`

  ### Step 3: Implement MCP Protocol
  1. Create `MCPServer.swift` to handle the MCP protocol
  2. Implement JSON-RPC 2.0 message handling
  3. Map MCP tool calls to RemindersAPI functions
  4. Handle stdio communication (stdin/stdout)

  ### Step 4: Create LaunchDaemon Configuration
  1. Create `com.user.reminders-mcp.plist` for launchd
  2. Configure it to:
  - Run at user login
  - Restart on failure
  - Log to ~/Library/Logs/RemindersMLPServer/
  - Use stdio for MCP communication

  ### Step 5: Handle Permissions
  1. Add Info.plist with NSRemindersUsageDescription
  2. Implement permission request flow
  3. Handle cases where permissions are denied

  ### Step 6: Build and Install
  1. Create build script that:
  - Compiles the Swift executable
  - Copies it to ~/Library/Application Support/RemindersMLPServer/
  - Installs the LaunchDaemon plist
  - Creates necessary directories

  ### Step 7: Create MCP Configuration
  1. Create the MCP server configuration file:
  ```json
  {
    "mcpServers": {
      "reminders": {
        "command": "~/Library/Application Support/RemindersMLPServer/reminders-mcp",
        "args": [],
        "env": {}
      }
    }
  }

  Step 8: Testing

  1. Create test script to verify:
    - Server starts and responds to MCP initialization
    - Each tool function works correctly
    - Error handling works properly
    - Daemon restarts on failure

  Key Technical Considerations

  EventKit Integration

  - Use EKEventStore for Reminders access
  - Handle async permission requests
  - Properly bridge Swift async/await with MCP's request/response model

  MCP Protocol Implementation

  - Implement proper JSON-RPC 2.0 spec
  - Handle tool discovery (initialize method)
  - Properly format tool schemas for Claude

  Daemon Management

  - Use launchd for process management
  - Implement proper signal handling (SIGTERM, SIGINT)
  - Add health check endpoint

  Success Criteria

  The MVP is complete when:
  1. Claude can create a reminder using natural language
  2. Claude can list existing reminders
  3. Claude can mark reminders as complete
  4. The server runs continuously in the background
  5. The server restarts automatically if it crashes

  Future Enhancements (Post-MVP)

  - Support for multiple reminder lists
  - Reminder search and filtering
  - Recurring reminders
  - Location-based reminders
  - Reminder priorities and tags
  - Better error messages and logging
