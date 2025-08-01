# Practical introduction to building remote MCP servers

<img width="825" height="468" alt="image" src="https://github.com/user-attachments/assets/741a558a-9d3b-4d00-9a99-1a270a979ccd" />
<img width="819" height="463" alt="image" src="https://github.com/user-attachments/assets/255810a4-67f1-496d-8148-e07b4f59f765" />
<img width="818" height="465" alt="image" src="https://github.com/user-attachments/assets/1656e6da-c15a-4d68-afd6-210d59c5bb4a" />
<img width="820" height="462" alt="image" src="https://github.com/user-attachments/assets/020947c4-afd1-443b-a1e4-2492ec544301" />
<img width="814" height="465" alt="image" src="https://github.com/user-attachments/assets/c2616458-faad-4b84-acd0-d8d406e9a69a" />
# GitHub MCP: From Local to Remote - Detailed Notes

## Presenters
- **Toby Padilla** (@toby) - Principal Product Manager, GitHub
- **Joe Zhou** (@mossaka) - Senior Software Engineer, Microsoft

---

## Executive Summary

This presentation covers GitHub's journey of evolving their Model Context Protocol (MCP) server from a local implementation to a remote one, including a practical demonstration of building a remote MCP server from scratch. The session highlights the benefits, challenges, and technical implementation details of both approaches.

---

## 1. MCP Background & Anthropic's Innovation

### What is MCP?
- **Model Context Protocol** - A standard for connecting AI models to external data sources and tools
- Created by Anthropic as more of a **social innovation than technical innovation**
- Built on pre-existing LLM function calling APIs that weren't gaining traction
- Anthropic's key contribution: **building community and momentum** around a standard

### Anthropic's Bootstrap Strategy
- Created **20-30 reference MCP servers** to kickstart the ecosystem
- GitHub MCP server was one of the original reference implementations
- **GitHub MCP server became one of the most popular** from the original batch
- This popularity led to GitHub taking ownership and further development

---

## 2. GitHub's MCP Server Evolution

### Initial State (Anthropic's Version)
- Local MCP server implementation
- Popular but had limitations in user experience
- Served as foundation for GitHub's enhanced version

### GitHub's First Iteration
- **Rewrote in Golang** for better performance
- Maintained as **drop-in replacement** for Anthropic's original
- Still operated as a **local MCP server**
- Used original codebase as reference to ensure compatibility

### User Experience Problems with Local Servers

#### Authentication Issues
- Required users to create **Personal Access Tokens (PATs)**
- Complex permission setup with multiple checkboxes
- Users often unsure about required permissions
- **Security concerns**: Users might grant excessive permissions
- Not user-friendly for non-technical users

#### Distribution and Updates
- Distributed via **Docker containers**
- Required Docker installation on user machines
- **Update notification problems**: Users didn't know about new releases
- Users stuck on old versions (thousands still using outdated versions)
- Manual update process: either new binary installation or Docker pull commands

---

## 3. The Remote MCP Solution

### Collaboration Catalyst
- **Microsoft and Anthropic partnership** intensified
- Multiple team members joined **MCP steering committee**
- Key contributor: **Den Delamarski** (OAuth expert)
- Collaboration with **Aaron from Okta** and community members

### Technical Breakthrough
- **Enhanced MCP protocol** with OAuth improvements
- Eliminated need for PAT tokens
- Enabled **web-based OAuth flow** - more secure and user-friendly
- **Immediate feature rollout** - SaaS-like experience

### Benefits of Remote MCP Servers
1. **Better Authentication**: Standard OAuth flow instead of PATs
2. **Automatic Updates**: Features appear immediately for all users
3. **No Local Installation**: No Docker or binary installation required
4. **Centralized Management**: Single hosted service for all users
5. **Better Security**: Proper OAuth scopes and permissions

---

## 4. Technical Implementation Demo

### Development Setup
Joe demonstrated building a remote MCP server using:

#### Package Manager
- **UV (Python package manager)** - chosen for efficiency
- Alternative to pip
- Commands:
  ```bash
  uv init [project-name]
  cd [project-name]
  uv run hello.py
  ```

#### Core Dependencies
```bash
uv add mcp  # Official Python MCP SDK
uv add pydantic  # For schema validation
```

### Basic MCP Server Structure

#### Simple Hello World (9 lines of code)
```python
from mcp import FastMCP

mcp = FastMCP("Demo Server")

@mcp.tool()
def hello_world(name: str) -> str:
    """Say hello to someone"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run()
```

#### Key Components
- **FastMCP class**: High-level MCP interface
- **@mcp.tool() decorator**: Converts Python functions to MCP tools
- **Automatic documentation**: Docstrings become tool descriptions
- **Type hints**: Automatic parameter validation

### Database Implementation
Created `db.py` with SQLite-based todo management:
- **CRUD operations**: Create, Read, Update, Delete
- **Todo schema**: ID, title, completion status
- **Database initialization and management**
- **Prepared for business logic integration**

### Local Server Testing
- **MCP Inspector UI**: Browser-based testing interface
- **Standard IO transport**: For local development
- Command: `uv run mcp`
- **Interactive tool testing**: Add todos, list items, mark complete

### VS Code Integration
Configuration in `mcp.json`:
```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "uv",
      "args": ["run", "mcp", "src/tools.py"],
      "transport": "stdio"
    }
  }
}
```

Integration with GitHub Copilot:
- **Agentic mode** with GPT-4o
- Natural language commands to interact with tools
- Example: "List all my todo items", "Mark this one as complete"

---

## 5. Converting to Remote (Streamable HTTP)

### Conversion Process
Converting from stdio to remote required **only 2 lines of code change**:

```python
# From local stdio server
if __name__ == "__main__":
    mcp.run()

# To remote HTTP server  
if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

### Remote Server Benefits
- **HTTP endpoint**: Accessible at `localhost:8000`
- **Inspector compatibility**: Same testing interface via HTTP
- **Logging and monitoring**: Full request/response logging
- **Debugging capabilities**: Complete operation audit trail
- **Multi-user support**: Single server, multiple clients

### Testing Remote Server
- **MCP Inspector**: Connect via HTTP instead of stdio
- URL format: `http://localhost:8000/mcp`
- **NPX command**: `npx @modelcontextprotocol/inspector`
- **Connection flexibility**: Connect/disconnect testing

---

## 6. Production Deployment (Azure Container Apps)

### Azure Developer CLI (AZD)
Setup process:
```bash
# Install AZD
# Login to Azure
azd auth login

# Deploy everything with one command
azd up
```

### Infrastructure as Code
- **Bicep language**: Infrastructure definition
- **Resource definitions**: Container registry, monitoring, identities
- **Container Apps**: Managed container hosting service
- **Port configuration**: Set to 8000 (Python default)
- **Environment variables**: MCP_FAST_HOST for containerization

### Configuration Details
- **Host setting**: `0.0.0.0` for containerized deployment
- **Public endpoint**: Generated by Azure Container Apps
- **Global accessibility**: Internet-accessible endpoint
- **Automatic scaling**: Managed by Azure platform

---

## 7. Challenges and Considerations

### Technical Challenges

#### SDK Migration
- Used **unofficial Go SDK** before official version
- **Specification evolution**: Rapid changes in MCP protocol
- **Migration requirement**: Need to adopt official Go SDK
- **Timing issues**: SDK lag behind protocol updates

#### User Migration
- **Tens of thousands** of existing local server users
- **Manual migration required**: Remove local, install remote
- **Communication challenge**: YouTube videos, tweets, documentation
- **Incomplete adoption**: Many users still on old versions
- **Recommendation**: Consider starting with remote if that's the end goal

#### Architectural Complexity
Local vs Remote comparison:

**Local MCP Server:**
- Simple command-line tool
- Standard I/O communication
- One tool per user
- Single-user focus
- Minimal state management

**Remote MCP Server:**
- Application server architecture
- Handles **millions of users**
- **Session management** required
- **State persistence** across requests
- **Connection handling** (disconnects, reconnects)
- **Multi-tenancy** considerations

### Production Readiness Requirements

#### Security
- **Authentication and authorization** implementation
- **OAuth flow** integration
- **Rate limiting** for abuse prevention
- **Input validation** and sanitization

#### Operational Excellence
- **Sophisticated business logic**
- **Caching strategies** for performance
- **Health checks** for reliability
- **Monitoring and alerting**
- **Error handling and recovery**
- **Logging and audit trails**

#### Scalability
- **Load balancing** for high availability
- **Database optimization** for concurrent users
- **Resource management** and auto-scaling
- **Performance monitoring** and optimization

---

## 8. GitHub Copilot Agent Integration

### Real-World Usage
- **GitHub Copilot coding agent**: 20-minute autonomous issue resolution
- **100% GitHub interaction**: Through GitHub MCP server
- **Agent architecture**: MCP host embedded in agent
- **Extensibility**: Multiple MCP servers can be installed
- **File system integration**: Additional MCP servers for broader functionality

### Agent-to-Agent Communication
- **MCP-mediated communication**: Agents communicate through MCP
- **GitHub MCP server exposure**: Agents accessible via MCP
- **Issue assignment**: MCP server can assign issues to GitHub Copilot
- **Recursive pattern**: Agents using MCP to communicate with other agents using MCP
- **"Turtles all the way down"**: Nested MCP interactions

### Protocol Limitations
- **Long-running tool calls**: Current limitation being addressed
- **Protocol evolution**: Active development in MCP specification
- **Future capabilities**: Enhanced agent-to-agent communication

---

## 9. Resources and Further Learning

### Official Documentation
- **Python MCP SDK**: Provides FastMCP class and tooling
- **MCP Specifications**: Official protocol documentation
- **Specification Enhancement Proposals**: Future protocol development

### Microsoft Resources
- **Azure Container Apps**: Managed container hosting
- **Microsoft MCP efforts**: Related projects and initiatives
- **Security blog post**: "How to build secure and scalable remote MCP servers"

### Community Resources
- **Anthropic roadmap talk**: Previous day's presentation
- **Official Model Context Protocol GitHub**: Specifications and projects
- **LangGraph integration**: MCP adapter for agentic frameworks
- **Community adapters**: Various framework integrations

### Demo Repository
- **GitHub repository**: `mossaka/remote-mcp-python-demo`
- **Complete example**: Full source code for demonstration
- **Infrastructure templates**: Bicep files for Azure deployment
- **Documentation**: Setup and deployment instructions

---

## 10. Key Takeaways

### Strategic Decisions
1. **Remote-first approach**: Consider starting remote if that's the eventual goal
2. **User experience priority**: OAuth flow significantly better than PATs
3. **Community building**: Social innovation as important as technical innovation
4. **Standard adoption**: Benefit from established protocols and standards

### Technical Lessons
1. **Simplicity in conversion**: Local to remote can be straightforward with right abstractions
2. **Infrastructure as code**: Essential for reproducible deployments
3. **Monitoring importance**: Logging and debugging crucial for remote servers
4. **SDK maturity**: Consider stability of tooling and libraries

### Operational Insights
1. **Migration complexity**: User base migration more challenging than technical migration
2. **Production requirements**: Security, scalability, and reliability need careful planning
3. **Agent integration**: MCP enables powerful agent-to-agent communication patterns
4. **Continuous deployment**: Remote servers enable immediate feature rollout

This comprehensive overview demonstrates both the technical feasibility and strategic value of migrating from local to remote MCP servers, providing a roadmap for organizations considering similar transitions.
