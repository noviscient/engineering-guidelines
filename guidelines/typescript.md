# TypeScript

## Table of Contents

- [Naming conventions](#naming-conventions)
- [Prefer ES6/ESNext syntax over ES5 syntax.](#prefer-es6esnext-syntax-over-es5-syntax)
- [Do not declare uninitialised variables.](#do-not-declare-uninitialised-variables)
- [Typing patterns](#typing-patterns)
- [File and folder organisation.](#file-and-folder-organisation)

## Naming conventions

- **Name exported symbols (ie. types and variables) discretely.** Anything that is exported should be clear about what it is, and not be confusable with anything else in the codebase.
	```ts
	// ❌ BAD
	// Is this a list of financial institutions? A list of users? A list of countries? I have to peek at the source file to find out, which is an unnecessary use of time and energy.
	export type List = {
		name: string,
		address: string,
		country: string,
	}[]

	// ✅ GOOD
	export type FinancialInstitutionList = {
		name: string,
		address: string,
		country: string,
	}[]

	// ✅ GOOD
	// not exported, so it's okay (if there is only one thing that "List" can refer to within the file)
	type List = {
		name: string,
		address: string,
		country: string,
	}[]
	```
- Names of indexes should be prefixed with `Idx` or `Index` if they are 0-based, `Num` or `Number` if 1-based.
	```ts
	// ❌ BAD
	// This is confusing because it's not clear whether the index is 0-based or 1-based.
	const getMonthName = (month: number) => {
		const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
		return monthNames[month]
	}

	// ❌ BAD
	// This is slightly better but not ideal as I have to check the definition of `Month` to know if it's 0-based or 1-based.
	type Month = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
	const getMonthName = (month: Month) => {
		const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
		return monthNames[month - 1]
	}

	// ✅ GOOD
	// This is clear because the index is prefixed with `num` and the list is 1-based. It is not necessary to implement both `Idx` and `Num` but it's clear when I'm using either function, whether the input should be 0-based or 1-based.
	type MonthIdx = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11
	type MonthNum = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
		const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
	const getMonthFromNum = (monthNum: MonthNum) => {
		return monthNames[monthNum - 1]
	}
	const getMonthFromIndex = (monthIndex: MonthIdx) => {
		return monthNames[monthIndex]
	}
	```
- If types are expected to have specific values, specify them in the type definition. This makes it easier to understand what the type is supposed to be.
	```ts
	// ❌ BAD
	export type PeriodOption = string
	export type MonthNum = number
	// ✅ GOOD
	export type PeriodOption = 'MTD' | '1M' | '3M' | '6M' | 'YTD' | '1Y' | '3Y' | 'MAX'
	export type MonthNum = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
	```
- Name a type as singular if it is singular, and prefix with `List` if it is a list of it. Plural should only be used if it is neither.
	```ts
	// ❌ BAD
	export type Drivers = {
		name: string,
		age: number,
		transmission: Transmissions[]
	}
	export enum Transmissions {
		automatic = 'automatic',
		manual = 'manual'
	}
	export type Driver = User[]
	// ✅ GOOD
	export type User = {
		name: string,
		age: number,
		transmission: Transmission[]
	}
	export type Transmission = 'automatic' | 'manual'
	export type UserList = User[]
	```
- Use `camelCase` for variable and function names. You may prefix with `_` to additionally inform that variables aren't meant to be used outside of the declared scope. Use `PascalCase` for types, enum and classes names.
- **Use `namespace` to group business logic or utilities** so that it is clear just from glancing at the code, where certain overly specific functions belong to (and so that those overly specific functions don't pollute the import scope). Namespaces should be in `camelCase` and should be in plural unless its object of concern is a singleton. Try not to prefix it with `NS`, `Namespace` or `Utils` as it being plural is clear enough.
	```ts
	// ❌ BAD
	export const getMonthName = (month: number) => {
		// ...
	}
	export const getMonthNum = (monthName: string) => {
		// ...
	}
	// ✅ GOOD
	export namespace months {
		export const getMonthName = (month: number) => {
			// ...
		}
		export const getMonthNum = (monthName: string) => {
			// ...
		}
	}
	```

## Prefer ES6/ESNext syntax over ES5 syntax.

- Use `const`; use `let` only if reassignment is necessary _and_ appropriate, and do not use `var`.
- Prefer `async`/`await`/`Promise`/`Thenable` over nested callbacks.
	```ts
	// ❌ BAD
	const doSomething = (callback: (result: string) => void) => {
		// ...
		callback(result)
	}
	doSomething(setTimeout(somethingElse, 1000))
	
	// ✅ GOOD
	const doSomething = async (): Promise<string> => {
		// ...
		return result
	}
	const wait = async (durationMS: number) => new Promise((resolve, reject) => setTimeout(resolve, durationMS))
	
	doSomething()
		.then(() => wait(1000))
		.then(somethingElse)
	```
- Where applicable, use `Promise.prototype.then()` and `Promise.prototype.catch()` over `try...catch` blocks.
	```ts
	// ❌ BAD
	try {
		const result = await doSomething()
	} catch (err) {
		// ...
	}
	
	// ✅ GOOD
	doSomething()
		.then((result) => {
			// ...
		})
		.catch((err) => {
			// ...
		})
	```
- Use `for...of` instead of `for...in` for iterating over arrays.
- For clarity, use `Object.entries`, `Object.values`, `Object.keys`, `for...of` and `Array.prototype.forEach` over `for...in` and `for (var i = 0; i < arr.length; i++)`. The final case may be applicable in some cases where `i` is of importance.
- Prefer `Array.prototype.map`, `Array.prototype.reduce` and `Array.prototype.filter` over re-creating an array separately if the map/reduce/filter function is clear-cut.
- Use nullish coalescing over "`||`" _where applicable_.
	```ts
	// ❌ BAD
	const x = y || 0
	// ✅ GOOD
	const x = y ?? 0
	```
- Use optional chaining over "`&&`" _where applicable_.
	```ts
	// ❌ BAD
	const x = y && y.z
	// ✅ GOOD
	const x = y?.z
	```
- Do not use `==` or `!=`. Use `===` and `!==` instead.
- Try to avoid manipulating objects using `Object.assign` as it interferes with clean type checking. Instead, use object spread syntax.
	```ts
	// ❌ BAD
	const x = Object.assign({}, y, { z: 1 })
	// ✅ GOOD
	const x = { ...y, z: 1 }
	```

## Do not declare uninitialised variables.

```ts
// ❌ BAD
let x: MonthYear // or `let x: MonthYear`
if (condition) {
	x = { month: 1, year: 2020 }
} else {
	x = { month: 2, year: 2020 }
}
// ✅ GOOD
const x: MonthYear = condition ? { month: 1, year: 2020 } : { month: 2, year: 2020 }
// or
const _getMonthYear = (): MonthYear => {
	if (condition) {
		return { month: 1, year: 2020 }
	} else {
		return { month: 2, year: 2020 }
	}
}
const x = _getMonthYear()
// or
// For cases where `x` may not be set:-
let x: MonthYear | undefined = undefined
if (condition) {
	x = { month: 1, year: 2020 }
} else (otherCondition) {
	x = { month: 2, year: 2020 }
}
if (!x) {
	// ...
}
```

## Typing patterns
- **Use `[]` instead of `Array`**. `[]` is more concise and easier to read. `Array` adds a nesting of type parameters.
- **Do not use `any` if the type can be known**. `any` is a wildcard type and should only be used if the type cannot be known. If the type can be known, then it should be specified.
- **Use `unknown` instead of `any` for values of unknown typing that will be consumed**. This ensures that the consumer will typecheck before consuming it.
- **Use commas instead of `;` for property separation.**
- **Either declare all properties in one line if it's short enough, or declare one on each line if it's long.** This makes it easier to read and compare properties.
	```ts
	// ❌ BAD
	type User = {
		id: string; name: string;
		age: number;
	}
	type UserDetailed = { id: string, name: string, /* 10 other properties here */ }
	// ✅ GOOD
	type User = { id: string, name: string, age: number }
	type User = {
		id: string,
		name: string,
		age: number,
	}
	type UserDetailed = {
		id: string,
		name: string,
		/* 10 other properties here */
	}
	```
- **Use `type` instead of `interface`**. `type` is more flexible and can be used to define union types (`string | number`), intersection types (`{ className: string } & { color: string }`), and primitive types.
	```ts
	// ❌ BAD
	interface User {
		id: UserId
		name: string;
		age: number;
	}
	// ✅ GOOD
	type User = {
		id: UserId,
		name: string,
		age: number,
	};
	type UserId = string
	```
- Use `PascalCase` for type names. Prefix them for specific use cases:-
	- If it is React component props, prefix with `P` followed by the component name in `PascalCase`.
		```tsx
		// ❌ BAD
		type Props = {
			name: string,
		}
		const MyComponent: React.FC<MyComponentProps> = ({ name }) => {
			return <div>{ name }</div>
		}
		// ✅ GOOD
		type PMyComponent = {
			name: string
		}
		const MyComponent: React.FC<PMyComponent> = ({ name }) => {
			return <div>{ name }</div>
		}
		```
	- If it is non-React component props but still a function's argument object made just for that function, prefix with `F` followed by the function name in `PascalCase`.
		```ts
		// ❌ BAD
		export type OAuthClientSetupFunctionProps = {
			name: string,
			clientID: string,
			callbackURL: string
		}
		export const createOAuthClient = (opts: OAuthClientSetupFunctionProps) => {
			// ...
		}
		// ✅ GOOD
		export type FCreateOAuthClient = {
			name: string,
			clientID: string,
			callbackURL: string
		}
		export const createOAuthClient = (opts: FCreateOAuthClient) => {
			// ...
		}
		```
	- (optional) if it is a state type for an atom that won't otherwise be used, you can prefix it with `S`.
		```ts
		// ❌ BAD
		export type FactsheetListDisplayState = {
			name: string,
			age: number
		}
		export const factsheetListDisplayAtom = atom<FactsheetListDisplayState>(...)
		
		// ✅ GOOD
		export type SFactsheetListDisplay = {
			name: string,
			age: number
		}
		export const factsheetListDisplayAtom = atom<SFactsheetListDisplay>(...)
		```
- **Use string literal types over `enum`.** They offer more options for transmutation.
	```ts
	// ❌ BAD
	export enum Transmissions {
		automatic = 'automatic',
		manual = 'manual'
	}
	export const TransmissionChoiceLabels: Record<Transmission, string> = {
		[Transmissions.automatic]: 'Automatic',
		[Transmissions.manual]: 'Manual'
	}
	// ✅ GOOD
	export const Transmissions = ['automatic', 'manual']
	export type Transmission = typeof Transmissions[number]
	// or export type Transmission = 'automatic' | 'manual'
	export const TransmissionChoiceLabels: Record<Transmission, string> = {
		automatic: 'Automatic',
		manual: 'Manual'
	}
	```
- **If using `enum`, use `snake_case` or `SCREAMING_SNAKE_CASE` for enum options.** This is to prevent confusion with `PascalCase` for types.
	```ts
	// ❌ BAD
	export enum Transmissions {
		Automatic = 'automatic',
		Manual = 'manual'
	}
	// ✅ GOOD
	export enum Transmissions {
		automatic = 'automatic',
		manual = 'manual'
	}
	```
- (situationally optional) **For typing item maps where it is otherwise unclear what the keys are, consider describing the keys in the type annotation.**
	```ts
	// ❌ BAD
	// is this mapped by factsheet ID or index?
	type FactsheetScoreMap = Record<string, number>
	
	// ✅ GOOD
	type FactsheetScoreMap = { [factsheetID: string]: number }
	```
- **Write functions and return Promises with CONSIDERATE return types in mind.**

- **If a written function's return type is likely to be consumed and its return type is not easily inferrable, annotate the return type in the function.** If the return type is understandably nothing, irrelevant (ie. return type will NEVER be used) or is something simple + can be inferred in a glance, then it is not necessary to annotate the return type.
	```ts
	// ❌ BAD
	const createFactsheet = async (opts: CreateFactsheetOpts) => {
		// ...
		return doSomething(opts)
			.then((result) => {
				return result
			})
			.catch((err) => {
				return err.toString()
			})
	}
	createFactsheet().then((iDontKnowWhatsThis) => {
		stopLoading()
		// ???
		if (typeof iDontKnowWhatsThis === 'object') {
			// ...maybe it succeeded?
		} else {
			console.error(iDontKnowWhatsThis)
			toast.danger('Could not create Factsheet, I don\'t know why.')
		}
	})

	// ✅ GOOD
	const createFactsheet = async (opts: CreateFactsheetOpts): Promise<{ok: true, fs: Factsheet} | {ok: false, err: string}> => {
		return doSomething(opts)
			.then((result) => {
				return { ok: true as const, fs: result }
			})
			.catch((e) => {
				const err = errUtils.extractErrString(e)
				return { ok: false as const, err }
			})
	}
	createFactsheet().then((result) => {
		if (result.ok) {
			toastSuccess('Factsheet created!')
			handleNewFactsheetAdded(result.fs)
		} else {
			toast.danger(`Could not create Factsheet: ${result.err}`)
		}
	}).finally(() => {
		stopLoading()
	})
	```


## File and folder organisation.

- Business logic is organised by feature into child directories and subdirectories within the `modules` parent directory.
- Utilities (defined as functions that are not specific to a feature in any application) are organised in files in the `utils` directories.
- Feature-specific utils are placed in `utils` directories within feature directories.
- Non-feature-specific React components are placed within the `components` directory.
- Feature-specific React components are placed within the `components` directory within feature directories.
- While creating React components, newly created utility and feature types and functions that _may_ be used elsewhere should not be placed in that React component's file, but placed in an adjacent file instead. React component files should **not** be imported when their components are not rendered.
	- This rule is to prevent circular import chains that can carelessly join together over time.
	- Exceptions can be made to this rule if it is and is likely forever that that file will not import other files.
- Name files appropriately.
	- Files primarily containing React components should be named `ComponentName.tsx`.
	- Files primarily containing React hooks should be named `useHookName.ts`.
	- Files primarily containing a collection of utility types and functions can be named anything.
	- Files primarily containing a namespace of utility types and functions should be named accordingly, ie. `namespace.tsx`.
	- Files primarily containing state atoms should be named `somethingAtom.ts`.
- React component files should _only_ host the component described in its filename, and optionally the following if applicable. Anything else should be placed in another file (such as its own file).
	- Child components that exist _only_ to be a part of the main component.
	- Other utility functions/hooks/loaders/etc that exist _only_ to be a part of / support the main component.