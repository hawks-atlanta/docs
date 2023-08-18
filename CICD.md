# CI/CD

```mermaid
mindmap
((Repository))
	[Push]
		[main]
			Deploy Production
			Release
			Coverage
		[dev]
			Tagging
			Deploy Development
	[Pull Request]
		[dev]
			Run tests
```

## Continuous integration

```mermaid
mindmap
((Repository))
	[Push]
		[main]
			Release
			Coverage
		[dev]
			Tagging
	[Pull request]
		[dev]
			Test
```

### Test

| Feature     | Value                             |
| ----------- | --------------------------------- |
| Executes    | On **PR** to `dev`                |
| Permissions | **Read only** repository contents |

```mermaid
stateDiagram-v2
	direction LR
	state Build {
		direction LR
        [*] --> BuildBinary
        BuildBinary --> [*]
    }
    state Test {
		direction LR
        [*] --> DockerCompose
        DockerCompose --> UnitTest
        UnitTest --> [*]
    }
	[*] --> Build
	Build --> Test
	Test --> [*]
```

### Tagging

| Feature     | Value                              |
| ----------- | ---------------------------------- |
| Executes    | On **Pushes** to `dev`             |
| Permissions | **Read/Write** repository contents |

```mermaid
stateDiagram-v2
	direction LR
	
	state Tagging {
		direction LR
        [*] --> CheckoutDev
        CheckoutDev --> StandardVersion
        StandardVersion --> Push
        Push --> [*]
    }
    
    [*] --> Tagging
    Tagging --> [*]
```

### Release

| Feature     | Value                                                        |
| ----------- | ------------------------------------------------------------ |
| Executes    | On **Pushes** to `main`                                      |
| Permissions | **Read only** repository contents, **Read/Write** releases, **Read/Write** packages |

```mermaid
stateDiagram-v2
	direction LR
	
	state Versioning {
		direction LR
		VersionPy: Read tag version
		[*] --> VersionPy
		VersionPy --> [*]
	}
	
	state Build {
		direction LR
		docker: Build Docker image
		registry: Upload docker image
		[*] --> docker
		docker --> registry
		registry --> [*]
	}
	
	state CreateRelease {
		direction LR
		createRelease: Create Release
		[*] --> createRelease
		createRelease --> [*]
	}
	
	[*] --> Versioning
	Versioning --> CreateRelease
	CreateRelease --> Build
	Build --> [*]
```

### Coverage

```mermaid
stateDiagram-v2
	direction LR
	state Coverage {
        direction LR
        EnvironmentUp: Environment Up
        coverage: Coverage
        Push: Push converage.txt
        EnvironmentDown: Environment Down
        [*] --> EnvironmentUp
        EnvironmentUp --> coverage
        coverage --> Push
        Push --> EnvironmentDown
        EnvironmentDown --> [*]
	}
	
	[*] --> Coverage
	Coverage --> [*]
```

## Continuous deployment

```mermaid
mindmap
((Repository))
	[Push]
        [dev]
        	Deploy Development
        [main]
        	Deploy Production
```

