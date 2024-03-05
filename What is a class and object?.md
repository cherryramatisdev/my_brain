First let's compare the original definition you probably saw from your university teacher and them we'll step into a new view of the world about these definitions shall we?

The original definition is that a class abstracts/represents an entity from the real world, with it's characteristics and behaviors, a classic example would be:

```typescript
class Dog {
	private _name: String

	constructor(name: string) {
		this._name = name
	}

	get name(): string {
		return this._name
	}

	public bark(): string {
		return "Au!"
	}
}
```