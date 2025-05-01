# MetaGPT Technical Architecture

This document outlines the technical architecture of MetaGPT, a multi-agent framework that simulates a software development company with different AI roles collaborating to develop software from simple requirements.

## 1. Overall Architecture

The MetaGPT framework is designed to simulate a software development company where multiple AI agents with different roles (e.g., Product Manager, Architect, Engineer) collaborate to develop software from a simple requirement. The system uses a modular, agent-based architecture where each role has specific responsibilities and communicates with other roles through a shared environment.

```mermaid
flowchart TB
    subgraph Input
        Requirement[User Requirement]
    end

    subgraph MetaGPT["MetaGPT Framework"]
        Team["Team"]
        Environment["Environment"]
        subgraph Roles["Roles / Agents"]
            TeamLeader["Team Leader"]
            PM["Product Manager"]
            Architect["Architect"]
            Engineer["Engineer"]
            DataAnalyst["Data Analyst"]
        end
        LLM["LLM Provider"]
        Actions["Actions"]
        Memory["Memory"]
    end

    subgraph Output
        Code["Generated Code"]
        Documents["Documents<br/>(PRD, Design Doc, etc.)"]
    end

    Requirement --> Team
    Team -- "Creates & Manages" --> Roles
    Team -- "Initializes" --> Environment
    Roles -- "Communicate via" --> Environment
    Environment -- "Routes messages" --> Roles
    Roles -- "Perform" --> Actions
    Actions -- "Query" --> LLM
    Roles -- "Store/Retrieve" --> Memory
    Actions --> Code
    Actions --> Documents
    
    classDef primary fill:#f9f,stroke:#333,stroke-width:2px;
    classDef secondary fill:#bbf,stroke:#333,stroke-width:1px;
    classDef tertiary fill:#dfd,stroke:#333,stroke-width:1px;
    
    class Team,Environment primary;
    class Roles,LLM,Actions,Memory secondary;
    class TeamLeader,PM,Architect,Engineer,DataAnalyst tertiary;
```

## 2. Core Components

### 2.1 Team

The `Team` class is the central orchestrator that manages a collection of AI agents (roles) and the environment in which they operate. It serves as the main entry point for users to create and manage a virtual software development team.

Key responsibilities:
- Hiring roles (agents) with different specializations
- Managing the team's budget and resources
- Initializing the environment for agent communication
- Starting projects based on user requirements
- Coordinating the execution of the multi-agent workflow

### 2.2 Environment

The environment is a shared space that facilitates communication between different roles. It maintains the message history, manages communication channels, and ensures proper message routing between agents.

Key responsibilities:
- Maintaining the agent registry
- Routing messages between agents
- Managing the project context
- Tracking the state of the multi-agent system
- Archiving the project history

### 2.3 Roles / Agents

Roles represent different specializations found in a typical software development company. Each role has specific responsibilities, goals, and capabilities.

Main roles include:
- **Team Leader**: Coordinates the team and assigns tasks to appropriate roles
- **Product Manager**: Creates product requirements documents (PRD)
- **Architect**: Designs the system architecture
- **Engineer**: Implements the code based on requirements and design
- **Data Analyst**: Handles data-related tasks and analysis

### 2.4 Actions

Actions represent specific tasks that roles can perform. Each role has a set of actions it can execute based on its responsibilities.

Common actions include:
- WritePRD: Creates a product requirement document
- WriteDesign: Creates a system design document 
- WriteCode: Implements code based on requirements and design
- CodeReview: Reviews and suggests improvements to code
- RunTests: Creates and executes tests

### 2.5 Memory

Memory allows agents to store and retrieve information from previous interactions, which is essential for maintaining context and coherence in the development process.

## 3. Sequence Diagram

The following sequence diagram illustrates the main flows and interactions within the MetaGPT system when a new software development project is initiated:

```mermaid
sequenceDiagram
    participant User
    participant Team
    participant Environment
    participant TeamLeader
    participant PM as ProductManager
    participant Architect
    participant Engineer
    
    User->>Team: generate_repo(idea)
    Team->>Team: invest(budget)
    Team->>Environment: add_roles([TeamLeader, ProductManager, Architect, Engineer, ...])
    Team->>Environment: publish_message(idea)
    
    Environment->>TeamLeader: route_message(idea)
    TeamLeader->>Environment: publish_message(to=ProductManager)
    Environment->>PM: route_message(requirement)
    
    PM->>PM: WritePRD.run()
    PM->>Environment: publish_message(PRD)
    Environment->>TeamLeader: route_message(PRD)
    TeamLeader->>Environment: publish_message(to=Architect)
    Environment->>Architect: route_message(PRD)
    
    Architect->>Architect: WriteDesign.run()
    Architect->>Environment: publish_message(Design)
    Environment->>TeamLeader: route_message(Design)
    TeamLeader->>Environment: publish_message(to=Engineer)
    Environment->>Engineer: route_message(Design)
    
    Engineer->>Engineer: WriteCode.run()
    Engineer->>Environment: publish_message(Code)
    Environment->>TeamLeader: route_message(Code)
    
    Team->>Environment: archive()
    Team-->>User: return results
```

## 4. Data Models

The following class diagrams show the key data models and their relationships in the MetaGPT framework:

### 4.1 Core Classes

```mermaid
classDiagram
    class Team {
        +env: Environment
        +investment: float
        +idea: string
        +hire(roles: list[Role])
        +invest(investment: float)
        +run_project(idea: string)
        +run(n_round: int, idea: string)
    }
    
    class Environment {
        +roles: dict
        +history: History
        +context: Context
        +add_role(role: Role)
        +add_roles(roles: list[Role])
        +get_role(role_name: string)
        +publish_message(message: Message)
        +run()
        +archive()
    }
    
    class Role {
        +name: string
        +profile: string
        +goal: string
        +constraints: string
        +actions: list[Action]
        +set_actions(actions: list[Action])
        +watch(actions: list[Action])
        +observe()
        +think()
        +act()
        +run()
    }
    
    class Action {
        +name: string
        +desc: string
        +context: Context
        +llm: LLM
        +run()
    }
    
    class Message {
        +content: string
        +role: string
        +cause_by: string
        +send_to: set
        +meta_data: dict
    }
    
    Team -- Environment : contains
    Environment -- Role : manages multiple
    Role -- Action : performs
    Role -- Message : sends/receives
    Environment -- Message : routes
```

### 4.2 Specialized Roles

```mermaid
classDiagram
    class Role {
        +name: string
        +profile: string
        +goal: string
        +constraints: string
        +actions: list[Action]
        +set_actions(actions: list[Action])
        +watch(actions: list[Action])
        +observe()
        +think()
        +act()
        +run()
    }
    
    class TeamLeader {
        +name: string = "Mike"
        +profile: string = "Team Leader"
        +goal: string = "Manage a team to assist users"
        +publish_team_message()
    }
    
    class ProductManager {
        +name: string = "Alice"
        +profile: string = "Product Manager"
        +goal: string = "Create a Product Requirement Document"
        +constraints: string
    }
    
    class Architect {
        +name: string = "Bob"
        +profile: string = "Architect"
        +goal: string = "Design a concise, usable, complete software system"
        +constraints: string
    }
    
    class Engineer2 {
        +name: string = "Alex"
        +profile: string = "Engineer"
        +goal: string = "Implement the system based on the design"
        +constraints: string
    }
    
    class DataAnalyst {
        +name: string = "David"
        +profile: string = "Data Analyst"
        +goal: string = "Handle data-related tasks"
        +constraints: string
    }
    
    Role <|-- TeamLeader
    Role <|-- ProductManager
    Role <|-- Architect
    Role <|-- Engineer2
    Role <|-- DataAnalyst
```

### 4.3 Actions Hierarchy

```mermaid
classDiagram
    class Action {
        +name: string
        +desc: string
        +context: Context
        +llm: LLM
        +run()
    }
    
    class WritePRD {
        +run()
    }
    
    class WriteDesign {
        +run()
    }
    
    class WriteTasks {
        +run()
    }
    
    class WriteCode {
        +run()
    }
    
    class CodeReview {
        +run()
    }
    
    class RunTests {
        +run()
    }
    
    Action <|-- WritePRD
    Action <|-- WriteDesign
    Action <|-- WriteTasks
    Action <|-- WriteCode
    Action <|-- CodeReview
    Action <|-- RunTests
```

## 5. Implementation Details

### 5.1 Software Development Workflow

MetaGPT implements a standard software development workflow:

1. **Requirement Analysis**: The Product Manager receives the initial user requirement and creates a comprehensive PRD.
2. **System Design**: The Architect receives the PRD and creates a system architecture design.
3. **Task Planning**: Tasks are identified and distributed among the team.
4. **Implementation**: Engineers write the code according to the design and requirements.
5. **Quality Assurance**: Code reviews and testing ensure the quality of the implementation.
6. **Delivery**: The final product is delivered, including source code and documentation.

### 5.2 Multi-agent Communication

Agents communicate through a message-passing system:

1. Each role can publish messages to the environment
2. Messages include metadata about the sender, intended recipients, and context
3. The environment routes messages to the appropriate recipients
4. Roles observe incoming messages and react accordingly

### 5.3 LLM Integration

MetaGPT is designed to work with various LLM providers:

- OpenAI (GPT-3.5, GPT-4)
- Azure OpenAI
- Anthropic
- Groq
- Local LLMs

The framework abstracts away the specific LLM implementation details, making it easy to switch between different providers.

## 6. Extensibility

MetaGPT is designed to be highly extensible:

- **Custom Roles**: Users can create custom roles with specific responsibilities
- **Custom Actions**: New actions can be defined to extend the capabilities of roles
- **Custom Workflows**: The interaction pattern between roles can be modified
- **Plugin System**: Additional functionality can be added through plugins

## 7. Conclusion

The MetaGPT architecture provides a flexible and powerful framework for creating AI-driven software development teams. By decomposing the software development process into specialized roles and defining clear communication channels, the system can effectively generate complete software projects from simple requirements.

The modular design of the framework makes it adaptable to different domains and use cases, while the agent-based approach allows for natural collaboration and division of labor in complex tasks.