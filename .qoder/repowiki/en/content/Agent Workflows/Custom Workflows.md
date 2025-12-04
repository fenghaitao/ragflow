# Custom Workflows

<cite>
**Referenced Files in This Document**
- [agent/component/base.py](file://agent/component/base.py)
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py)
- [agent/canvas.py](file://agent/canvas.py)
- [agent/component/loop.py](file://agent/component/loop.py)
- [agent/component/switch.py](file://agent/component/switch.py)
- [agent/component/iteration.py](file://agent/component/iteration.py)
- [agent/component/variable_assigner.py](file://agent/component/variable_assigner.py)
- [agent/component/varaiable_aggregator.py](file://agent/component/varaiable_aggregator.py)
- [agent/component/loopitem.py](file://agent/component/loopitem.py)
- [agent/component/iterationitem.py](file://agent/component/iterationitem.py)
- [agent/component/exit_loop.py](file://agent/component/exit_loop.py)
- [agent/component/categorize.py](file://agent/component/categorize.py)
- [agent/component/message.py](file://agent/component/message.py)
- [agent/component/invoke.py](file://agent/component/invoke.py)
- [agent/templates/choose_your_knowledge_base_workflow.json](file://agent/templates/choose_your_knowledge_base_workflow.json)
- [agent/templates/customer_service.json](file://agent/templates/customer_service.json)
- [agent/test/dsl_examples/iteration.json](file://agent/test/dsl_examples/iteration.json)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Workflow Architecture Overview](#workflow-architecture-overview)
3. [Core Components](#core-components)
4. [Control Flow Components](#control-flow-components)
5. [Data Manipulation Components](#data-manipulation-components)
6. [Building Custom Workflows](#building-custom-workflows)
7. [Domain Model and Execution](#domain-model-and-execution)
8. [Advanced Patterns](#advanced-patterns)
9. [Debugging and Troubleshooting](#debugging-and-troubleshooting)
10. [Best Practices](#best-practices)

## Introduction

RAGFlow's custom workflow system provides a powerful visual programming interface for creating sophisticated AI agent interactions. The system is built around a directed acyclic graph (DAG) architecture where components represent individual processing units that communicate through variable passing and control flow mechanisms.

Workflows in RAGFlow enable complex multi-agent scenarios, iterative processing, conditional branching, and sophisticated data manipulation patterns. The system supports everything from simple linear processing to complex nested loops and parallel execution patterns.

## Workflow Architecture Overview

The workflow system is built on several key architectural principles:

```mermaid
graph TB
subgraph "Canvas Layer"
Canvas[Canvas Manager]
DSL[DSL Parser]
Variables[Variable System]
end
subgraph "Component Layer"
Base[Base Component]
Agents[Agent Components]
Tools[Tool Components]
Control[Control Flow]
end
subgraph "Execution Layer"
Scheduler[Execution Scheduler]
State[State Management]
Error[Error Handling]
end
Canvas --> DSL
DSL --> Base
Base --> Agents
Base --> Tools
Base --> Control
Canvas --> Scheduler
Scheduler --> State
State --> Error
```

**Diagram sources**
- [agent/canvas.py](file://agent/canvas.py#L40-L100)
- [agent/component/base.py](file://agent/component/base.py#L393-L450)

### Key Architectural Features

1. **Visual Programming Interface**: Workflows are defined through a drag-and-drop interface with JSON-based DSL representation
2. **Component-Based Design**: Each workflow element is a reusable component with standardized interfaces
3. **Variable System**: Components communicate through a flexible variable passing mechanism
4. **Execution Engine**: Asynchronous execution with proper state management and error handling
5. **Extensible Architecture**: Easy addition of new component types and capabilities

**Section sources**
- [agent/canvas.py](file://agent/canvas.py#L40-L100)
- [agent/component/base.py](file://agent/component/base.py#L393-L450)

## Core Components

### Base Component Architecture

All workflow components inherit from the `ComponentBase` class, which provides essential functionality for workflow execution:

```mermaid
classDiagram
class ComponentBase {
+string component_name
+Graph _canvas
+string _id
+ComponentParamBase _param
+invoke(**kwargs) dict
+output(var_nm) Any
+set_output(key, value) void
+get_input() dict
+set_input_value(key, value) void
+check_if_canceled(message) bool
+reset(only_output) void
}
class ComponentParamBase {
+dict inputs
+dict outputs
+int max_retries
+float delay_after_error
+string exception_method
+check() void
+update(conf, allow_redundant) self
}
class Agent {
+dict tools
+LLMBundle chat_mdl
+_invoke(**kwargs) Any
+_react_with_tools_streamly() Iterator
}
ComponentBase <|-- Agent
ComponentBase --> ComponentParamBase
```

**Diagram sources**
- [agent/component/base.py](file://agent/component/base.py#L393-L583)
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L81-L120)

### Component Lifecycle

Components follow a standardized lifecycle during workflow execution:

1. **Initialization**: Parameter validation and setup
2. **Input Resolution**: Variable substitution and input preparation
3. **Execution**: Component-specific processing logic
4. **Output Generation**: Result storage and variable updates
5. **Cleanup**: Resource cleanup and state reset

**Section sources**
- [agent/component/base.py](file://agent/component/base.py#L434-L447)

## Control Flow Components

### Loops and Iterations

RAGFlow provides sophisticated looping constructs for repetitive processing:

#### Loop Component
The `Loop` component manages iteration over variables with termination conditions:

```mermaid
flowchart TD
Start([Loop Start]) --> InitVars[Initialize Variables]
InitVars --> CheckCount{Max Loop Count?}
CheckCount --> |Reached| End([Loop Complete])
CheckCount --> |Continue| ProcessItem[Process Loop Item]
ProcessItem --> EvaluateCond[Evaluate Termination Condition]
EvaluateCond --> |Continue| Increment[Increment Counter]
Increment --> CheckCount
EvaluateCond --> |Terminate| End
```

**Diagram sources**
- [agent/component/loop.py](file://agent/component/loop.py#L43-L79)
- [agent/component/loopitem.py](file://agent/component/loopitem.py#L35-L163)

#### Iteration Component
The `Iteration` component processes arrays and collections:

```mermaid
sequenceDiagram
participant Canvas as Canvas
participant Iteration as Iteration
participant Item as IterationItem
participant Child as Child Components
Canvas->>Iteration : Initialize with array
Iteration->>Iteration : Validate array type
loop For each item
Iteration->>Item : Create new item instance
Item->>Item : Set item/index variables
Item->>Child : Execute child components
Child-->>Item : Return results
Item->>Iteration : Aggregate results
end
Iteration-->>Canvas : Return aggregated results
```

**Diagram sources**
- [agent/component/iteration.py](file://agent/component/iteration.py#L59-L72)
- [agent/component/iterationitem.py](file://agent/component/iterationitem.py#L35-L92)

### Conditional Branching

#### Switch Component
The `Switch` component provides conditional routing based on variable values:

```mermaid
flowchart TD
Input[Input Variables] --> EvalConditions[Evaluate Conditions]
EvalConditions --> LogicalOp{Logical Operator}
LogicalOp --> |AND| CheckAll[Check All Conditions]
LogicalOp --> |OR| CheckAny[Check Any Condition]
CheckAll --> AllTrue{All True?}
CheckAny --> AnyTrue{Any True?}
AllTrue --> |Yes| RouteTrue[Route to True Path]
AllTrue --> |No| RouteFalse[Route to False Path]
AnyTrue --> |Yes| RouteTrue
AnyTrue --> |No| RouteFalse
RouteTrue --> Output[Output Next Components]
RouteFalse --> ElsePath[Else/Default Path]
```

**Diagram sources**
- [agent/component/switch.py](file://agent/component/switch.py#L64-L98)

#### Categorize Component
The `Categorize` component uses LLM reasoning for classification:

```mermaid
sequenceDiagram
participant User as User Input
participant Cat as Categorize
participant LLM as LLM Model
participant Output as Output Routing
User->>Cat : Submit query
Cat->>Cat : Build classification prompt
Cat->>LLM : Send classification request
LLM-->>Cat : Return category prediction
Cat->>Cat : Count category occurrences
Cat->>Cat : Select dominant category
Cat->>Output : Route to appropriate components
```

**Diagram sources**
- [agent/component/categorize.py](file://agent/component/categorize.py#L99-L149)

**Section sources**
- [agent/component/switch.py](file://agent/component/switch.py#L64-L141)
- [agent/component/categorize.py](file://agent/component/categorize.py#L99-L149)

## Data Manipulation Components

### Variable Assignment

The `VariableAssigner` component provides comprehensive variable manipulation capabilities:

| Operation | Description | Type Safety |
|-----------|-------------|-------------|
| `overwrite` | Replace variable with new value | Yes |
| `set` | Set variable with parameter | Yes |
| `append` | Add item to list | Element type checking |
| `extend` | Merge arrays | Element type compatibility |
| `+=`, `-=` | Numeric operations | Number validation |
| `clear` | Reset to default | Type-specific clearing |

**Section sources**
- [agent/component/variable_assigner.py](file://agent/component/variable_assigner.py#L44-L192)

### Variable Aggregation

The `VariableAggregator` component enables group-based variable selection:

```mermaid
flowchart LR
subgraph "Group Selection"
G1[Group 1] --> V1[Variable 1]
G1 --> V2[Variable 2]
G1 --> V3[Variable 3]
end
subgraph "Selection Logic"
V1 --> Check{Available?}
V2 --> Check
V3 --> Check
Check --> |Yes| Select[Select First Available]
Check --> |No| Next[Check Next Variable]
end
Select --> Output[Output Group Value]
```

**Diagram sources**
- [agent/component/varaiable_aggregator.py](file://agent/component/varaiable_aggregator.py#L61-L85)

**Section sources**
- [agent/component/varaiable_aggregator.py](file://agent/component/varaiable_aggregator.py#L61-L85)

### Message Component

The `Message` component handles output formatting and templating:

```mermaid
flowchart TD
Input[Message Content] --> Template{Template Engine?}
Template --> |Jinja2| Jinja[Jinja2 Rendering]
Template --> |Plain| Plain[Direct Substitution]
Jinja --> Stream{Streaming?}
Plain --> Stream
Stream --> |Yes| StreamOutput[Stream Output]
Stream --> |No| StaticOutput[Static Output]
StreamOutput --> Convert{Format Conversion?}
StaticOutput --> Convert
Convert --> PDF[PDF Export]
Convert --> HTML[HTML Export]
Convert --> Markdown[Markdown Export]
Convert --> Docx[DOCX Export]
```

**Diagram sources**
- [agent/component/message.py](file://agent/component/message.py#L174-L267)

**Section sources**
- [agent/component/message.py](file://agent/component/message.py#L174-L267)

## Building Custom Workflows

### Step-by-Step Workflow Creation

#### 1. Define Inputs and Outputs

Every workflow begins with defining the input parameters and expected outputs:

```json
{
    "components": {
        "begin": {
            "obj": {
                "component_name": "Begin",
                "params": {
                    "inputs": {
                        "query": {
                            "name": "User Query",
                            "type": "string",
                            "optional": false
                        }
                    },
                    "outputs": {
                        "result": {
                            "type": "string",
                            "value": ""
                        }
                    }
                }
            }
        }
    }
}
```

#### 2. Connect Components

Establish connections between components using upstream/downstream relationships:

```json
{
    "components": {
        "begin": {
            "downstream": ["agent"],
            "upstream": []
        },
        "agent": {
            "downstream": ["message"],
            "upstream": ["begin"]
        }
    }
}
```

#### 3. Configure Component Parameters

Each component requires specific parameter configuration:

```json
{
    "agent": {
        "obj": {
            "component_name": "Agent",
            "params": {
                "llm_id": "deepseek-chat@DeepSeek",
                "sys_prompt": "You are a helpful assistant...",
                "max_rounds": 5,
                "temperature": 0.7
            }
        }
    }
}
```

### Creating a Basic Workflow

Here's a simple workflow that demonstrates the basic pattern:

```mermaid
sequenceDiagram
participant User as User Input
participant Begin as Begin Component
participant Agent as Agent Component
participant Message as Message Component
User->>Begin : Submit query
Begin->>Begin : Initialize workflow
Begin->>Agent : Pass query
Agent->>Agent : Process with LLM
Agent->>Message : Return response
Message->>Message : Format output
Message-->>User : Display result
```

**Diagram sources**
- [agent/templates/choose_your_knowledge_base_workflow.json](file://agent/templates/choose_your_knowledge_base_workflow.json#L14-L80)

**Section sources**
- [agent/templates/choose_your_knowledge_base_workflow.json](file://agent/templates/choose_your_knowledge_base_workflow.json#L14-L80)

### Advanced Workflow Patterns

#### Multi-Agent Collaboration

Complex workflows often involve multiple agents working together:

```mermaid
graph TB
subgraph "Intent Classification"
Categorize[Categorize Component]
end
subgraph "Agent Pool"
Casual[Casual Chat Agent]
Product[Product Info Agent]
Support[Support Agent]
end
subgraph "Response Aggregation"
Aggregator[Variable Aggregator]
FinalMessage[Final Message]
end
Categorize --> Casual
Categorize --> Product
Categorize --> Support
Casual --> Aggregator
Product --> Aggregator
Support --> Aggregator
Aggregator --> FinalMessage
```

**Diagram sources**
- [agent/templates/customer_service.json](file://agent/templates/customer_service.json#L167-L325)

#### Iterative Processing

Workflows can process data iteratively with nested components:

```mermaid
flowchart TD
Start([Start]) --> Decompose[Decompose Topic]
Decompose --> Iterate[Iterate Through Topics]
Iterate --> Search[Search Web]
Search --> Generate[Generate Report]
Generate --> Collect[Collect Results]
Collect --> MoreItems{More Items?}
MoreItems --> |Yes| Iterate
MoreItems --> |No| Aggregate[Aggregate Reports]
Aggregate --> End([Complete])
```

**Diagram sources**
- [agent/test/dsl_examples/iteration.json](file://agent/test/dsl_examples/iteration.json#L27-L81)

**Section sources**
- [agent/templates/customer_service.json](file://agent/templates/customer_service.json#L167-L325)
- [agent/test/dsl_examples/iteration.json](file://agent/test/dsl_examples/iteration.json#L27-L81)

## Domain Model and Execution

### State Management

The workflow execution engine maintains state through several mechanisms:

```mermaid
classDiagram
class Canvas {
+dict globals
+dict variables
+list history
+dict retrieval
+list memory
+run(**kwargs) AsyncGenerator
+get_variable_value(exp) Any
+set_variable_value(exp, value) void
}
class ComponentState {
+dict inputs
+dict outputs
+string error
+float elapsed_time
+bool canceled
}
class ExecutionEngine {
+list path
+ThreadPoolExecutor thread_pool
+run_batch(start, end) void
+process_components() void
+handle_errors() void
}
Canvas --> ComponentState
Canvas --> ExecutionEngine
```

**Diagram sources**
- [agent/canvas.py](file://agent/canvas.py#L282-L310)
- [agent/canvas.py](file://agent/canvas.py#L363-L400)

### Variable Scope and Resolution

Variables follow a hierarchical resolution pattern:

1. **Global Variables**: System-wide variables like `sys.query`
2. **Component Variables**: Per-component output variables
3. **Environment Variables**: Runtime environment values
4. **Nested Access**: Dot notation for complex object access

### Error Handling and Recovery

The system implements comprehensive error handling:

```mermaid
flowchart TD
Error[Component Error] --> Handler{Exception Handler?}
Handler --> |Comment| LogError[Log Error]
Handler --> |Default| DefaultValue[Use Default Value]
Handler --> |Goto| JumpTo[Jump to Specified Component]
Handler --> |None| PropagateError[Propagate Error]
LogError --> Continue[Continue Execution]
DefaultValue --> Continue
JumpTo --> Continue
PropagateError --> Stop[Stop Workflow]
```

**Diagram sources**
- [agent/component/base.py](file://agent/component/base.py#L565-L583)

**Section sources**
- [agent/canvas.py](file://agent/canvas.py#L282-L310)
- [agent/component/base.py](file://agent/component/base.py#L565-L583)

## Advanced Patterns

### Nested Loops and Conditional Logic

Complex workflows can combine multiple control flow patterns:

```mermaid
flowchart TD
OuterLoop[Outer Loop] --> InnerLoop{Inner Loop?}
InnerLoop --> |Yes| ConditionalBranch{Conditional Branch}
InnerLoop --> |No| ProcessItem[Process Item]
ConditionalBranch --> |Category A| AgentA[Agent A]
ConditionalBranch --> |Category B| AgentB[Agent B]
AgentA --> ProcessItem
AgentB --> ProcessItem
ProcessItem --> MoreItems{More Items?}
MoreItems --> |Yes| InnerLoop
MoreItems --> |No| MoreOuter{More Outer Items?}
MoreOuter --> |Yes| OuterLoop
MoreOuter --> |No| Finalize[Finalize Results]
```

### Parallel Execution

Some components can execute in parallel for improved performance:

```mermaid
sequenceDiagram
participant Canvas as Canvas
participant Parallel1 as Parallel Component 1
participant Parallel2 as Parallel Component 2
participant Parallel3 as Parallel Component 3
participant Aggregator as Result Aggregator
par Parallel Execution
Canvas->>Parallel1 : Start processing
Canvas->>Parallel2 : Start processing
Canvas->>Parallel3 : Start processing
end
Parallel1-->>Aggregator : Return result 1
Parallel2-->>Aggregator : Return result 2
Parallel3-->>Aggregator : Return result 3
Aggregator->>Aggregator : Combine results
Aggregator-->>Canvas : Final aggregated result
```

### Dynamic Component Creation

Advanced workflows can dynamically create and configure components during execution:

```mermaid
flowchart LR
Template[Component Template] --> Instantiate[Dynamic Instantiation]
Instantiate --> Configure[Runtime Configuration]
Configure --> Register[Register with Canvas]
Register --> Execute[Execute Component]
```

## Debugging and Troubleshooting

### Common Issues and Solutions

#### Infinite Loops

**Problem**: Workflow gets stuck in endless loops
**Solution**: Implement proper termination conditions and loop counters

```json
{
    "loop": {
        "maximum_loop_count": 100,
        "loop_termination_condition": [
            {
                "variable": "result",
                "operator": "not empty"
            }
        ]
    }
}
```

#### Variable Scope Problems

**Problem**: Variables not accessible across components
**Solution**: Use proper variable naming conventions and scope resolution

```javascript
// Correct: Using component reference
"{agent@response}"

// Incorrect: Missing component reference
"{response}"
```

#### Memory Leaks

**Problem**: Workflows consume excessive memory
**Solution**: Implement proper cleanup and state reset

```python
# Component cleanup pattern
def reset(self, only_output=False):
    # Clear component state
    self._param.outputs.clear()
    if not only_output:
        self._param.inputs.clear()
```

### Debugging Strategies

#### Enable Logging

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

#### Use Debug Mode

```json
{
    "debug_inputs": {
        "input_name": "debug_value"
    }
}
```

#### Monitor Execution

```mermaid
flowchart TD
Start([Workflow Start]) --> Monitor[Enable Monitoring]
Monitor --> Track[Track Component States]
Track --> Log[Log Execution Details]
Log --> Analyze[Analyze Performance]
Analyze --> Optimize[Optimize Workflow]
```

**Section sources**
- [agent/component/base.py](file://agent/component/base.py#L465-L476)

## Best Practices

### Workflow Design Principles

1. **Single Responsibility**: Each component should have a clear, focused purpose
2. **Loose Coupling**: Components should minimize dependencies on specific others
3. **Clear Interfaces**: Well-defined input/output contracts
4. **Error Resilience**: Comprehensive error handling and recovery
5. **Performance Awareness**: Consider execution time and resource usage

### Component Development Guidelines

#### Parameter Validation

```python
def check(self):
    self.check_positive_integer(self.max_retries, "Max retries must be positive")
    self.check_empty(self.llm_id, "LLM ID cannot be empty")
```

#### State Management

```python
def reset(self, only_output=False):
    # Reset only outputs for reuse
    if only_output:
        for key in self._param.outputs:
            self._param.outputs[key]["value"] = None
    else:
        # Full reset including inputs
        self._param.inputs.clear()
        self._param.outputs.clear()
```

#### Error Handling

```python
def _invoke(self, **kwargs):
    try:
        # Component logic
        result = self.process_data()
        self.set_output("result", result)
    except Exception as e:
        if self.get_exception_default_value():
            self.set_exception_default_value()
        else:
            self.set_output("_ERROR", str(e))
        logging.exception(e)
```

### Performance Optimization

#### Batch Processing

```mermaid
flowchart LR
Input[Large Dataset] --> Split[Split into Batches]
Split --> Batch1[Batch 1]
Split --> Batch2[Batch 2]
Split --> Batch3[Batch 3]
Batch1 --> Process[Process in Parallel]
Batch2 --> Process
Batch3 --> Process
Process --> Combine[Combine Results]
```

#### Caching Strategies

1. **Result Caching**: Store expensive computation results
2. **LLM Response Caching**: Cache LLM responses for repeated queries
3. **External API Caching**: Cache external service responses

### Testing and Validation

#### Unit Testing Components

```python
def test_component_execution():
    # Setup test environment
    canvas = create_test_canvas()
    component = create_test_component(canvas)
    
    # Test execution
    result = component.invoke(test_input)
    
    # Verify results
    assert result["expected_output"] == expected_result
```

#### Integration Testing

```python
def test_workflow_end_to_end():
    # Load workflow definition
    workflow = load_workflow("test_workflow.json")
    
    # Execute workflow
    result = execute_workflow(workflow, test_inputs)
    
    # Validate complete workflow behavior
    assert validate_workflow_results(result)
```

**Section sources**
- [agent/component/base.py](file://agent/component/base.py#L465-L476)
- [agent/component/base.py](file://agent/component/base.py#L565-L583)