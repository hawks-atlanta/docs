# CI/CD

## Continuous integration

```mermaid
mindmap
(Repository)
	[main]
		Push
			Release
	[dev]
		Pull Request
			Test
		Push
			Tagging
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
	BuildAndUpload: Build and Upload
	state BuildAndUpload {
		direction LR
		state fork_build <<fork>>
		state join_build <<join>>
		upx: Run UPX
		[*] --> fork_build
		fork_build --> Windows
		fork_build --> Linux
		fork_build --> FreeBSD
		fork_build --> OSX
		Windows --> join_build
		Linux --> join_build
		FreeBSD --> join_build
		OSX --> join_build
		join_build --> upx
		upx --> Upload
		Upload --> [*]
	}
	
	state CreateRelease {
		direction LR
		createRelease: Create Release
		[*] --> createRelease
		createRelease --> [*]
	}
	
	
	
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
	
	state fork_main <<fork>>
	state join_main <<join>>
	
	[*] --> fork_main
	fork_main --> Versioning
	fork_main --> Coverage
	Versioning --> CreateRelease
	CreateRelease --> BuildAndUpload
	BuildAndUpload --> join_main
	Coverage --> join_main
	join_main --> [*]
```

## Continuous deployment

```mermaid
mindmap
(Repository)
	Push
        dev
        	Deploy Development
        main
        	Deploy Production
```

