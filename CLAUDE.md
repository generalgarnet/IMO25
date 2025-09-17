# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Package Installation
```bash
pip install -r requirements.txt
```

### Running Single Agents
```bash
# Google Gemini 2.5 Pro agent (default)
python code/agent.py problems/problem_file.txt --log output.log

# OpenAI GPT-5 agent
python code/agent_oai.py problems/problem_file.txt --log output.log

# XAI Grok-4 agent
python code/agent_xai.py problems/problem_file.txt --log output.log
```

### Parallel Execution
```bash
# Run 10 agents in parallel
python code/run_parallel.py problems/problem_file.txt -n 10

# Run with custom timeout and log directory
python code/run_parallel.py problems/problem_file.txt -n 20 -t 300 -d logs/custom

# Run with different agent types
python code/run_parallel.py problems/problem_file.txt -a code/agent_oai.py -n 10
```

### Result Extraction
```bash
# Extract final result from JSON/JSONL files
python code/res2md.py logs/results.jsonl
```

## Architecture Overview

This is an AI agent system for solving International Mathematical Olympiad (IMO) problems using multiple LLM providers. The system demonstrates gold-medal performance on IMO 2025 problems.

### Core Components

1. **Agent Implementations** (`code/`):
   - `agent.py`: Google Gemini 2.5 Pro agent (primary implementation)
   - `agent_oai.py`: OpenAI GPT-5 agent
   - `agent_xai.py`: XAI Grok-4 agent
   - All agents follow the same interface and behavior patterns

2. **Execution System**:
   - `run_parallel.py`: Coordinates multiple parallel agents for increased success probability
   - Each agent runs independently with configurable timeouts
   - Real-time progress monitoring and result aggregation

3. **Utilities**:
   - `res2md.py`: Extracts final structured results from JSON/JSONL log files

### Agent Architecture

Each agent implements an iterative improvement cycle:

1. **Initial Solution Generation**: Uses structured mathematical reasoning prompts
2. **Self-Improvement**: Agent refines its own solution
3. **Verification**: Independent verification subsystem checks mathematical correctness
4. **Iterative Refinement**: Up to 30 iterations based on verification feedback

Key architectural features:
- **Memory System**: State persistence and recovery via JSON files
- **Dual Verification**: Primary verification plus secondary confirmation
- **Convergence Criteria**: 5 consecutive correct verifications or 10 failures
- **Logging**: Dual output to console and files with timestamps

### API Integration

Each agent uses different LLM providers but follows the same pattern:
- **Gemini**: Google Generative Language API v1beta
- **OpenAI**: Standard OpenAI API with GPT-5
- **XAI**: XAI API with Grok-4 models

All API calls include:
- Temperature=0.1 for consistency
- Structured prompt engineering for mathematical reasoning
- Error handling and retry logic

### File Structure

```
IMO25/
├── code/                    # Core implementation
│   ├── agent.py            # Gemini agent
│   ├── agent_oai.py        # OpenAI agent
│   ├── agent_xai.py        # XAI agent
│   ├── run_parallel.py     # Parallel execution
│   └── res2md.py           # Result extraction
├── problems/               # IMO problem files (txt format)
├── run_logs/               # Gemini execution logs
├── run_logs_gpt5/          # OpenAI execution logs
├── run_logs_grok4/         # XAI execution logs
└── requirements.txt        # Python dependencies
```

## Environment Setup

Required environment variables:
```bash
export GOOGLE_API_KEY=your_google_api_key    # For Gemini agent
export OPENAI_API_KEY=your_openai_api_key    # For OpenAI agent
export XAI_API_KEY=your_xai_api_key          # For XAI agent
```

## Key Development Patterns

### Agent Implementation
All three agents (`agent.py`, `agent_oai.py`, `agent_xai.py`) share identical interfaces and core logic. The only differences are:
- API endpoint configuration
- Authentication method
- Model-specific parameters

### Prompt Engineering
The system uses sophisticated prompt templates:
- **Step 1 Prompt**: Primary mathematical reasoning instruction
- **Self-Improvement Prompt**: Solution refinement guidance
- **Correction Prompt**: Iterative improvement based on verification feedback
- **Verification System Prompt**: Independent verification criteria

### Error Handling
- API call timeouts and retries
- State recovery from memory files
- Graceful degradation on provider failures
- Detailed logging for debugging

### Performance Considerations
- Parallel execution scales solution probability
- Memory system allows interruption and resumption
- Configurable timeouts prevent hanging
- Worker limits control resource usage

## Working with the Codebase

### Adding New Agent Types
1. Copy existing agent implementation
2. Modify API endpoint and authentication
3. Update `run_parallel.py` to recognize new agent type
4. Test with existing problem files

### Problem Format
Problems are stored as plain text files in `problems/` directory. Follow existing format for consistency.

### Testing Approach
- Use `run_parallel.py` with small agent counts (3-5) for testing
- Verify with multiple problem types
- Check log outputs for proper formatting
- Test memory recovery functionality