# Plan: Update ChatGPT to Use New OpenAI Models

## Current State Analysis

### Files Identified
1. **[modules/botpress-module/src/actions/call_chat_gpt_api.js](modules/botpress-module/src/actions/call_chat_gpt_api.js)** - Main API implementation
2. **[modules/botpress-module/src/views/full/AiTask.tsx](modules/botpress-module/src/views/full/AiTask.tsx)** - UI component with model dropdown
3. **[modules/botpress-module/src/backend/skills/ai_task.ts](modules/botpress-module/src/backend/skills/ai_task.ts)** - Backend skill integration

### Current Implementation Issues
1. **Fictional Models**: The code references non-existent models:
   - `gpt-5`, `gpt-5-mini`, `gpt-5-nano` - These don't exist in OpenAI's API
   - These are treated as reasoning models with `reasoning_effort` parameter

2. **Outdated OpenAI SDK**: Using `openai@^3.2.1` (deprecated)
   - Uses old API: `OpenAIApi` and `createChatCompletion()`
   - Modern version is v4+ with different API structure

3. **Mixed Model Types**:
   - GPT models (standard chat models)
   - Reasoning models (o1-preview, o1-mini, o3-mini)
   - Gemini models (Google AI)
   - Claude models (placeholder, not implemented)

### Actual OpenAI Model Lineup (as of early 2025)
**Standard Chat Models:**
- `gpt-4o` - Latest GPT-4 Omni model
- `gpt-4o-mini` - Smaller, faster GPT-4 Omni
- `gpt-4-turbo` - GPT-4 Turbo
- `gpt-4` - Base GPT-4
- `gpt-3.5-turbo` - GPT-3.5 Turbo

**Reasoning Models (o-series):**
- `o1` - Latest reasoning model (released Dec 2024)
- `o1-preview` - Preview version
- `o1-mini` - Smaller reasoning model
- `o3-mini` - Newer mini reasoning model (if available)

**Note:** GPT-5 does not exist yet. The o1 models are OpenAI's latest advanced reasoning models.

## Implementation Plan

### Phase 1: Update OpenAI SDK
**Goal:** Upgrade from deprecated openai v3 to latest v4

**Changes needed:**
1. Update [modules/botpress-module/package.json](modules/botpress-module/package.json:27)
   - Change `"openai": "^3.2.1"` to `"openai": "^4.0.0"`

2. Update [call_chat_gpt_api.js](modules/botpress-module/src/actions/call_chat_gpt_api.js)
   - Replace deprecated imports and API calls
   - Old: `const { Configuration, OpenAIApi } = require("openai")`
   - New: `const OpenAI = require("openai")`
   - Replace `openai.createChatCompletion()` with new SDK method

### Phase 2: Fix Model Names in API Implementation
**File:** [call_chat_gpt_api.js](modules/botpress-module/src/actions/call_chat_gpt_api.js)

**Changes:**
1. **Remove fictional models** (lines 17, 44):
   - Remove default to `gpt-5` (line 17)
   - Change default to `gpt-4o` (actual latest model)
   - Remove `gpt-5`, `gpt-5-mini`, `gpt-5-nano` logic (line 44)

2. **Update reasoning model detection** (line 44-46):
   - Replace fictional models with actual reasoning models
   - Update condition to check for: `o1`, `o1-preview`, `o1-mini`, `o3-mini`
   - Keep `reasoning_effort` parameter for these models

3. **API Key Security** (line 9-10):
   - **CRITICAL**: Hardcoded API key exposed in code!
   - Move to environment variable: `process.env.OPENAI_API_KEY`

### Phase 3: Update UI Model Dropdown
**File:** [AiTask.tsx](modules/botpress-module/src/views/full/AiTask.tsx)

**Changes:**
1. **Update default model** (line 20):
   - Change from `gpt-5` to `gpt-4o`

2. **Fix dropdown options** (lines 72-81):
   - Remove: `gpt-5`, `gpt-5-mini`, `gpt-5-nano`
   - Keep existing real models: `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-4`, `gpt-3.5-turbo`
   - Consider adding: `o1` (latest reasoning model)

3. **Update reasoning model detection** (lines 29, 32):
   - Add `o1` to the reasoning models list
   - Update condition: `['o1', 'o1-preview', 'o1-mini', 'o3-mini'].includes(e.target.value)`

4. **Update info message** (line 147):
   - Update text to reflect actual reasoning models

### Phase 4: Verification
**Files to verify:**
1. [ai_task.ts](modules/botpress-module/src/backend/skills/ai_task.ts) - No changes needed (passes through data)
2. Bundle files will auto-regenerate on build

### Phase 5: Testing Considerations
**Post-implementation tests:**
1. Test standard GPT models (gpt-4o, gpt-4o-mini)
2. Test reasoning models with reasoning_effort parameter
3. Test temperature parameter (should NOT apply to reasoning models)
4. Verify Gemini models still work
5. Check error handling

## Security Concerns
**CRITICAL:** API key is hardcoded in [call_chat_gpt_api.js:9-10](modules/botpress-module/src/actions/call_chat_gpt_api.js:9)
- This key should be rotated immediately
- Move to environment variable
- Never commit API keys to version control

## Dependencies to Install
After updating package.json:
```bash
cd modules/botpress-module
yarn install
```

## Summary of Changes
| File | Lines Changed | Type of Change |
|------|---------------|----------------|
| package.json | 1 | Dependency upgrade |
| call_chat_gpt_api.js | ~15-20 | API modernization, model fixes, security |
| AiTask.tsx | ~10 | UI model options update |
| **Total** | **~30 lines** | **3 files** |

## Questions for User
Before proceeding with implementation:

1. **Model Selection**: Do you want to include the new `o1` model (latest reasoning model from Dec 2024)?

2. **Deprecated Models**: Should we remove old models like `gpt-3.5-turbo` and `gpt-4` (non-turbo), or keep them for backward compatibility?

3. **Gemini Models**: The Gemini version numbers (2.5-pro, 2.5-flash-lite, etc.) also seem non-standard. Should we verify/update these as well?

4. **Claude Models**: These are marked "coming soon". Should we implement Claude integration now, or leave as placeholder?

5. **API Key**: The OpenAI API key is currently hardcoded. Should we:
   - Move it to environment variable (recommended)
   - Move it to a config file
   - Use the existing pattern (not recommended for security)

## Risk Assessment
- **Low Risk**: Model name updates in UI
- **Medium Risk**: SDK upgrade (API changes)
- **High Risk**: API key security issue (should be addressed immediately)

## Estimated Complexity
**Overall: Medium**
- SDK upgrade requires careful testing
- Multiple files need coordinated changes
- Security fix is critical but straightforward
