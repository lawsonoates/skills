# tsconfig

Use this for root and every package:

```json
{
	"$schema": "https://json.schemastore.org/tsconfig",
	"extends": "@tsconfig/bun/tsconfig.json",
	"compilerOptions": {
		"types": ["bun"]
	}
}
```