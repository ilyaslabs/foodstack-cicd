# FoodStack CI/CD Workflows

This repository contains reusable GitHub Actions workflows for Java Maven projects in the FoodStack ecosystem. The workflows handle pull request validation and Maven Central deployment for Java libraries and applications.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Essential Setup
- Set Java 21 as the default JDK: `sudo update-alternatives --config java` (select Java 21)
- Set JAVA_HOME: `export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64`
- Verify setup: `java --version && mvn --version` (should show Java 21)

### Build and Test Commands
- `mvn clean` - Clean build artifacts (takes ~1.5 seconds)
- `mvn compile` - Compile source code (takes ~2 seconds) 
- `mvn test -B` - Run tests in batch mode (takes ~10 seconds). NEVER CANCEL. Set timeout to 3+ minutes for larger projects.
- `mvn clean deploy -B -U -DskipTests -Drevision=VERSION` - Deploy to Maven Central (will fail without proper repository configuration)

### Workflow Validation
- **ALWAYS** validate workflow YAML syntax: `yamllint .github/workflows/` (takes <1 second)
- Fix common yamllint issues:
  - Add `---` document start marker
  - Fix truthy values (`on:` vs `"on":`)
  - Add newline at end of file
- **NEVER** commit workflows with yamllint errors

### Installation Requirements
If Java 21 is not available:
```bash
sudo apt-get update && sudo apt-get install -y openjdk-21-jdk
sudo update-alternatives --config java  # Select Java 21
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

## Validation

### Mandatory Validation Steps
1. **ALWAYS** run `yamllint .github/workflows/` before committing workflow changes
2. **ALWAYS** verify Java 21 setup with `java --version && mvn --version`
3. **ALWAYS** test Maven commands in a sample project to verify they work
4. For workflow changes, create a test Maven project: `mvn archetype:generate -DgroupId=com.ilyaslabs.foodstack -DartifactId=test-project -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`

### Expected Behavior
- `mvn test -B` succeeds in a properly configured Maven project
- `mvn clean deploy` fails with "repository element was not specified" error (this is expected without Maven Central configuration)
- yamllint may show warnings about document start and truthy values (acceptable)
- yamllint should NOT show errors (fix these before committing)

### Testing Scenarios
When making workflow changes:
1. Create test Maven project in `/tmp`
2. Run the exact commands from the workflow
3. Verify expected success/failure patterns
4. Time the operations to update timeout recommendations

## Common Tasks

### Available Workflows

#### pr-checks.yml
- **Purpose**: Validates pull requests by running Maven tests
- **Command**: `mvn test -B`
- **Java Version**: Configurable (default: 21)
- **Timeout**: Set to 5+ minutes for safety. NEVER CANCEL.
- **Expected**: SUCCESS for valid Java projects with tests

#### release.yml
- **Purpose**: Deploys artifacts to Maven Central on release
- **Trigger**: Release created or workflow_call
- **Commands**: `mvn clean deploy -B -U -DskipTests -Drevision=${VERSION}`
- **Requirements**: GPG keys, OSSRH credentials
- **Timeout**: Set to 10+ minutes for deployment. NEVER CANCEL.
- **Expected**: FAILURE in test environment (missing credentials)

### Timing Expectations
- **NEVER CANCEL**: All Maven operations may take longer in CI environments
- yamllint validation: <1 second
- Maven clean: ~1.5 seconds  
- Maven compile: ~2 seconds
- Maven test: ~10 seconds (simple projects), up to 5+ minutes (complex projects)
- Maven deploy: ~2-10 minutes depending on artifact size
- **Set timeouts with 3x buffer**: Always use 3x the expected time for timeout values

### Repository Structure
```
.github/
├── workflows/
│   ├── pr-checks.yml      # Pull request validation
│   └── release.yml        # Maven Central deployment
└── copilot-instructions.md
.gitignore                 # Standard ignores
```

### Common Commands Output

#### `ls -la` (repository root)
```
.github/
.gitignore
```

#### `yamllint .github/workflows/` (typical output with warnings)
```
::group::.github/workflows/pr-checks.yml
::warning file=.github/workflows/pr-checks.yml,line=1,col=1::1:1 [document-start] missing document start "---"
::warning file=.github/workflows/pr-checks.yml,line=3,col=1::3:1 [truthy] truthy value should be one of [false, true]
::endgroup::

::group::.github/workflows/release.yml
::warning file=.github/workflows/release.yml,line=1,col=1::1:1 [document-start] missing document start "---"
::warning file=.github/workflows/release.yml,line=3,col=1::3:1 [truthy] truthy value should be one of [false, true]
::endgroup::
```

#### Environment validation
```bash
$ export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
$ java --version
openjdk 21.0.8 2025-07-15

$ mvn --version  
Apache Maven 3.9.11
Java version: 21.0.8, vendor: Ubuntu
```

## Troubleshooting

### Common Issues
- **"mvn: command not found"**: Install Maven with `sudo apt-get install maven`
- **"JAVA_HOME not set"**: Export JAVA_HOME as shown in setup
- **Maven using wrong Java version**: Set JAVA_HOME and verify with `mvn --version`
- **yamllint errors**: Fix syntax issues before committing
- **Deploy failures**: Expected without proper Maven Central configuration

### Workflow Debugging
- Check workflow syntax with yamllint first
- Verify Java/Maven setup matches workflow requirements
- Test commands locally before pushing workflow changes
- Expected deploy failure: "repository element was not specified"

### Performance Notes
- First Maven run downloads dependencies (takes longer)
- Subsequent runs use local cache (faster)
- Always allow extra time in CI environments
- **CRITICAL**: Never cancel long-running builds - they may take 10+ minutes