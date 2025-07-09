# Prompt: Build an MVP macOS Reminders MCP Server with LaunchAgent Architecture

  ## Project Overview
  Create a minimal MCP (Model Context Protocol) server that provides access to macOS Reminders through EventKit. The system consists of:
  - A LaunchAgent running the MCP server in the background
  - A menubar app for status monitoring and control
  - A CLI tool to start/manage the agent

  ## Architecture Components
  1. **MCP Server** (Swift) - Handles MCP protocol and EventKit access
  2. **LaunchAgent** - Manages background execution
  3. **Menubar App** (Swift/AppKit) - Shows status and provides controls
  4. **CLI Tool** (Swift) - Manages agent lifecycle

  ## MVP Features
  - Create reminders with title and optional due date
  - List reminders from default list
  - Mark reminders as complete
  - Show server status in menubar
  - Start/stop/restart server via CLI or menubar

  ## Implementation Steps

  ### Step 1: Project Structure Setup
  RemindersMLPServer/
  ├── Package.swift
  ├── Sources/
  │   ├── RemindersMLPServer/      # MCP Server
  │   │   ├── main.swift
  │   │   ├── MCPServer.swift
  │   │   ├── RemindersAPI.swift
  │   │   ├── Models.swift
  │   │   └── Info.plist
  │   ├── RemindersMLPMenubar/     # Menubar App
  │   │   ├── AppDelegate.swift
  │   │   ├── StatusItemController.swift
  │   │   ├── Info.plist
  │   │   └── RemindersMenubar.entitlements
  │   └── RemindersMLPCLI/         # CLI Tool
  │       └── main.swift
  ├── LaunchAgent/
  │   └── com.user.reminders-mcp.plist
  └── Scripts/
      ├── install.sh
      └── build.sh

  ### Step 2: Implement MCP Server Core

  #### 2.1 Create `Models.swift`
  ```swift
  import Foundation
  import EventKit

  struct MCPRequest: Codable {
      let jsonrpc: String
      let method: String
      let params: [String: Any]?
      let id: Int
  }

  struct MCPResponse: Codable {
      let jsonrpc: String = "2.0"
      let result: Any?
      let error: MCPError?
      let id: Int
  }

  struct MCPError: Codable {
      let code: Int
      let message: String
  }

  struct Tool: Codable {
      let name: String
      let description: String
      let inputSchema: [String: Any]
  }

  2.2 Create RemindersAPI.swift

  import EventKit

  class RemindersAPI {
      private let eventStore = EKEventStore()
      private var authorized = false

      func requestAccess() async throws {
          // Request access to Reminders
          authorized = try await eventStore.requestAccess(to: .reminder)
          if !authorized {
              throw RemindersError.accessDenied
          }
      }

      func createReminder(title: String, notes: String?, dueDate: Date?) async throws -> String
      func listReminders(includeCompleted: Bool) async throws -> [[String: Any]]
      func completeReminder(id: String) async throws -> Bool
      func getLists() async throws -> [[String: Any]]
  }

  2.3 Create MCPServer.swift

  class MCPServer {
      private let remindersAPI = RemindersAPI()
      private var inputBuffer = ""

      func start() async {
          // Request Reminders access on startup
          try? await remindersAPI.requestAccess()

          // Set up stdio for JSON-RPC communication
          FileHandle.standardInput.readabilityHandler = { [weak self] handle in
              let data = handle.availableData
              self?.handleInput(data)
          }

          // Send initialization capability
          sendCapabilities()

          // Keep running
          RunLoop.main.run()
      }

      private func handleInput(_ data: Data)
      private func processRequest(_ request: MCPRequest) async
      private func sendResponse(_ response: MCPResponse)
      private func sendCapabilities()
  }

  Step 3: Implement Menubar App

  3.1 Create AppDelegate.swift

  import Cocoa

  @main
  class AppDelegate: NSObject, NSApplicationDelegate {
      var statusItemController: StatusItemController!

      func applicationDidFinishLaunching(_ notification: Notification) {
          statusItemController = StatusItemController()
      }
  }

  3.2 Create StatusItemController.swift

  import Cocoa

  class StatusItemController {
      private let statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
      private var agentMonitor: AgentMonitor!

      init() {
          setupStatusItem()
          setupMenu()
          agentMonitor = AgentMonitor { [weak self] status in
              self?.updateStatus(status)
          }
      }

      private func setupStatusItem() {
          if let button = statusItem.button {
              button.image = NSImage(systemSymbolName: "checklist", accessibilityDescription: "Reminders MCP")
          }
      }

      private func setupMenu() {
          let menu = NSMenu()
          menu.addItem(withTitle: "Status: Unknown", action: nil, keyEquivalent: "")
          menu.addItem(NSMenuItem.separator())
          menu.addItem(withTitle: "Start", action: #selector(startAgent), keyEquivalent: "")
          menu.addItem(withTitle: "Stop", action: #selector(stopAgent), keyEquivalent: "")
          menu.addItem(withTitle: "Restart", action: #selector(restartAgent), keyEquivalent: "")
          menu.addItem(NSMenuItem.separator())
          menu.addItem(withTitle: "Quit", action: #selector(NSApplication.terminate(_:)), keyEquivalent: "q")
          statusItem.menu = menu
      }

      @objc private func startAgent()
      @objc private func stopAgent()
      @objc private func restartAgent()
  }

  Step 4: Implement CLI Tool

  4.1 Create RemindersMLPCLI/main.swift

  import Foundation

  enum Command: String {
      case start, stop, restart, status, install
  }

  class RemindersMLPCLI {
      static func main() {
          let args = CommandLine.arguments
          guard args.count > 1,
                let command = Command(rawValue: args[1]) else {
              printUsage()
              exit(1)
          }

          switch command {
          case .start:
              startAgent()
          case .stop:
              stopAgent()
          case .restart:
              restartAgent()
          case .status:
              printStatus()
          case .install:
              installAgent()
          }
      }

      static func startAgent() {
          // Load LaunchAgent
          let result = Process.run("/bin/launchctl", arguments: ["load", "-w", launchAgentPath])
          print(result.success ? "Started Reminders MCP server" : "Failed to start")
      }

      static func stopAgent() {
          // Unload LaunchAgent
          let result = Process.run("/bin/launchctl", arguments: ["unload", launchAgentPath])
          print(result.success ? "Stopped Reminders MCP server" : "Failed to stop")
      }
  }

  RemindersMLPCLI.main()

  Step 5: Create LaunchAgent Configuration

  5.1 Create com.user.reminders-mcp.plist

  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
      <key>Label</key>
      <string>com.user.reminders-mcp</string>
      <key>ProgramArguments</key>
      <array>
          <string>/usr/local/bin/reminders-mcp-server</string>
      </array>
      <key>KeepAlive</key>
      <true/>
      <key>RunAtLoad</key>
      <false/>
      <key>StandardOutPath</key>
      <string>/tmp/reminders-mcp.log</string>
      <key>StandardErrorPath</key>
      <string>/tmp/reminders-mcp.error.log</string>
  </dict>
  </plist>

  Step 6: Handle Permissions and Entitlements

  6.1 Create Info.plist for MCP Server

  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
      <key>NSRemindersUsageDescription</key>
      <string>Reminders MCP Server needs access to your reminders to provide MCP functionality.</string>
      <key>CFBundleIdentifier</key>
      <string>com.user.reminders-mcp-server</string>
  </dict>
  </plist>

  6.2 Embed Info.plist in Binary

  Update Package.swift build settings:
  .executableTarget(
      name: "RemindersMLPServer",
      dependencies: [],
      linkerSettings: [
          .unsafeFlags([
              "-sectcreate", "__TEXT", "__info_plist", "Sources/RemindersMLPServer/Info.plist"
          ])
      ]
  )

  Step 7: Build and Installation Scripts

  7.1 Create build.sh

  #!/bin/bash
  # Build all components
  swift build -c release

  # Create installation directories
  mkdir -p /usr/local/bin
  mkdir -p ~/Library/LaunchAgents
  mkdir -p /Applications

  # Copy binaries
  cp .build/release/RemindersMLPServer /usr/local/bin/reminders-mcp-server
  cp .build/release/RemindersMLPCLI /usr/local/bin/reminders-mcp

  # Build menubar app (requires Xcode)
  xcodebuild -project RemindersMenubar.xcodeproj -scheme RemindersMenubar -configuration Release
  cp -R build/Release/RemindersMenubar.app /Applications/

  # Install LaunchAgent
  cp LaunchAgent/com.user.reminders-mcp.plist ~/Library/LaunchAgents/

  7.2 Create install.sh

  #!/bin/bash
  # Run build
  ./build.sh

  # Set permissions
  chmod +x /usr/local/bin/reminders-mcp-server
  chmod +x /usr/local/bin/reminders-mcp

  # Create MCP configuration
  mkdir -p ~/.config/claude-code
  cat > ~/.config/claude-code/mcp-servers.json << EOF
  {
    "mcpServers": {
      "reminders": {
        "command": "/usr/local/bin/reminders-mcp-server",
        "args": [],
        "env": {}
      }
    }
  }
  EOF

  echo "Installation complete!"
  echo "Run 'reminders-mcp start' to start the server"
  echo "The menubar app is in /Applications/RemindersMenubar.app"

  Step 8: Testing Plan

  1. Permission Test: Verify EventKit access request works
  2. MCP Protocol Test: Test with Claude Code
  3. Agent Lifecycle Test: Start/stop/restart functionality
  4. Menubar Test: Status updates and controls
  5. Integration Test: Create, list, and complete reminders through Claude

  Step 9: First Run Experience

  1. User runs reminders-mcp install
  2. User runs reminders-mcp start or opens menubar app
  3. System prompts for Reminders access
  4. Server starts and shows "Running" in menubar
  5. Claude Code can now use reminders tools

  Success Criteria

  - ✅ MCP server responds to Claude's tool calls
  - ✅ Can create, list, and complete reminders
  - ✅ Menubar shows server status
  - ✅ CLI can control agent lifecycle
  - ✅ Survives system restart (when enabled)
  - ✅ Handles permission denial gracefully

  Key Implementation Notes

  - Use async/await for EventKit operations
  - Handle JSON-RPC parsing errors gracefully
  - Log errors to file for debugging
  - Update menubar icon based on server status
  - Cache EventKit authorization status
