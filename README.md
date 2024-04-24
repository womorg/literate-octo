# Validata

Type safe data validation and sanitization.

See also

- [@womorg/literate-octo-koa](https://www.npmjs.com/package/@womorg/literate-octo-koa) for more usage in [Koa](https://www.npmjs.com/package/koa)
- [@womorg/literate-octo-express](https://www.npmjs.com/package/@womorg/literate-octo-express) for more usage in [Express](https://www.npmjs.com/package/express)

## Getting started

```bash
npm i @womorg/literate-octo
```

## Basic usage

```typescript
import { asString, isObject, isString, maybeString } from '@womorg/literate-octo';

interface Sample {
  maybeString: string | undefined;
  myString: string;
  numericString: string;
}

const sample = isObject<Sample>({
  maybeString: maybeString(), // will allow string data type or sanitize to undefined
  myString: isString(), // will allow only string data type
  numericString: asString(), // will allow string or attempt to convert to string
});

console.log(
  JSON.stringify(
    sample.process({
      maybeString: 123,
      myString: 123,
      numericString: 123,
    })
  )
);

/*
FAIL: Outputs:
{"issues":[{"path":["maybeString"],"value":123,"reason":"incorrect-type","info":{"expectedType":"string"}},{"path":["myString"],"value":123,"reason":"incorrect-type","info":{"expectedType":"string"}}]}
*/

console.log(
  JSON.stringify(
    sample.process({
      myString: '123',
      numericString: 123,
    })
  )
);

/*
SUCCESS: Outputs:
{"value":{"myString":"123","numericString":"123"}}
*/
```

## API

Checks:

- isAny
- Array
  - isArray
  - maybeArray
  - asArray
  - maybeAsArray
- Boolean
  - isBoolean
  - maybeBoolean
  - asBoolean
  - maybeAsBoolean
- Date
  - isDate
  - maybeDate
  - asDate
  - maybeAsDate
- Enum
  - isEnum
  - maybeEnum
  - asEnum
  - maybeAsEnum
- Number
  - isNumber
  - maybeNumber
  - asNumber
  - maybeAsNumber
- Object
  - isObject
  - maybeObject
  - asObject
  - maybeAsObject
- Record
  - isRecord
  - maybeRecord
  - asRecord
  - maybeAsRecord
- String
  - isString
  - maybeString
  - asString
  - maybeAsString
- Tuple
  - isTuple
  - maybeTuple
- Url
  - isUrl
  - maybeUrl
  - asUrl
  - maybeAsUrl
- isNullable
- asNullable

Types

- TypeOf

Work is done by a typed `ValueProcessor`, as returned by`isObject<T>()` or `asNumber()`.

```typescript
interface ValueProcessor<T> {
  process(value: unknown): Result<T>;
}
```

The `process()` method returns a `Result<T>`.The `Result` is either a list of issues
(meaning validation failures) or the accepted value (it may be coerced/altered from the original).

```typescript
type Result<T> = ValueResult<T> | IssueResult;

interface ValueResult<T> {
  value: T;
}

interface IssueResult {
  issues: Issue[];
}
```

## Naming conventions

### `is...` e.g. `isNumber`

- if the value is of the type it will be accepted
- `null` or `undefined` cause an issue
- otherwise it will cause an issue

### `maybe...` e.g. `maybeNumber`

- if the value is of the type it will be accepted
- `null` or `undefined` it will sanitized to undefined
- otherwise it will cause an issue

### `as...` e.g. `asNumber`

- if the value is of the type it will be accepted
- `null` or `undefined` converted to default, if provided, or cause an issue
- if the value can be converted to the type, it will be converted and used
- if the value is cannot be converted the default will be used if provided
- otherwise it will cause an issue

### `maybeAs...` e.g. `maybeAsNumber`

- if the value is of the type it will be accepted
- `null` or `undefined` converted to default, if provided, or sanitized to undefined
- if the value can be converted to the type it will be converted and used
- if the value is cannot be converted the default will be used if provided
- otherwise it will cause an issue
  // \* otherwise it will be sanitized to undefined

## Checks

### `isArray`, `maybeArray`, `asArray`, `maybeAsArray`

Usage:

```typescript
isArray(itemProcessor, options);
maybeArray(itemProcessor, options);
asArray(itemProcessor, options);
maybeAsArray(itemProcessor, options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `coerceMaxLength? number` - if there are more items than this, some will be removed
- `maxLength?: number` - if there are more items than this, it's an error `max-length`
- `minLength?: number` - if there are less items than this, it's an error `min-length`
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

Example:

```typescript
isArray<number>(isNumber({ max: 20, min: 10 }), { coerceMaxLength: 7 });
```

### `isBoolean`, `maybeBoolean`, `asBoolean`, `maybeAsBoolean`

Usage:

```typescript
isBoolean(options);
maybeBoolean(options);
asBoolean(options);
maybeAsBoolean(options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

### `isDate`, `maybeDate`, `asDate`, `maybeAsDate`

Usage:

```typescript
isDate(options);
maybeDate(options);
asDate(options);
maybeAsDate(options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `format` - custom date format used in conversion from `string` to `Date` see [Luxon formatting](https://moment.github.io/luxon/docs/manual/formatting)
- `maxFuture?: Duration` - if the value is after this duration into the future, it's an error `max-future`
- `maxPast?: Duration` - if the value is before this duration into the past, it's an error `max-past`
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

### `isEnum`, `maybeEnum`, `asEnum`, `maybeAsEnum`

Usage:

```typescript
isEnum(Enum);
maybeNumber(Enum);
asNumber(Enum);
maybeAsNumber(Enum);
```

Example:

```typescript
// String based Enum
enum EnumOne {
  A = 'A',
  B = 'B',
  C = 'C',
}
isEnum(EnumOne); // Allows "A", "B", "C"

// Number based Enum
enum EnumTwo {
  A,
  B,
  C,
}
isEnum(EnumTwo); // Allows 0, 1, 2

// Converting to an Enum using it's key or value
asEnum(EnumTwo); // Allows 1, 2, 3, "A", "B", "C"
asEnum(EnumTwo).process('A'));       // { value: 0 }
asEnum(EnumTwo).process(0));         // { value: 0 }
asEnum(EnumTwo).process(EnumOne.A)); // { value: 0 }
```

### `isNumber`, `maybeNumber`, `asNumber`, `maybeAsNumber`

Usage:

```typescript
isNumber(options);
maybeNumber(options);
asNumber(options);
maybeAsNumber(options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `coerceMin?: number` - if the value is less than this, it will be set to this value
- `coerceMax?: number` - if the value is more than this, it will be set to this value
- `max?: number` - if the value is than this, it's an error `max`
- `min?: number` - if the value is than this, it's an error `min`
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

### `isObject`, `maybeObject`, `asObject`, `maybeAsObject`

Usage:

```typescript
isObject(contract, options);
maybeObject(contract, options);
asObject(contract, options); // will parse string JSON as object
maybeAsObject(contract, options); // will parse string JSON as object
// where `contract` is Record<string, ValueProcessor>
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

Example:

```typescript
interface Sample {
  myString: string;
  maybeString: string | undefined;
  numericString: string;
}

const check = isObject<Sample>({
  maybeString: maybeString(), // if these don't match the interface TypeScript will error
  myString: isString(),
  numericString: asString(),
});
```

### `isRecord`, `maybeRecord`, `asRecord`, `maybeAsRecord`

Usage:

```typescript
isRecord<V>(check, options);
maybeRecord<V>(check, options);
asRecord<V>(check, options);
maybeAsRecord<V>(check, options);
// where `check` is ValueProcessor<V>, and Record<string, V> is the type to be processed
```

Options:

- `keyRegex?: RegExp` - regular expression to check each key name, or it's an error `key-regex`
- `maxKeys?: number` - if the number of keys in the object is more than this, it's an error `max-keys`
- `minKeys?: number` - if the number of keys in the object is more than this, it's an error `max-keys`
- `validator?: (value: Record<string, V>, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

Example:

```typescript
const check = isRecord(isString());
check.process({ foo: 'bar' });
```

### `isString`, `maybeString`, `asString`, `maybeAsString`

Usage:

```typescript
isString(options);
maybeString(options);
asString(options);
maybeAsString(options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `limitLength?: number` - if the length of the string is more than this, it will be truncated to this length
- `padStart?: StringPadding` - pad the start of the string up to given value
- `padEnd?: StringPadding` - pad the end of the string up to given value
- `trim?: 'start' | 'end' | 'both' | 'none'` - removes the leading and/or trailing white space and line terminator characters from the string
- `regex?: RegExp` - regular expression that must be matched, or it's an error `regex`
- `maxLength?: number` - if the length of the string is more than this, it's an error `max-length`
- `minLength?: number` - if the length of the string is less than this, it's an error `min-length`
- `format:? StringFormatCheck` - extension point for string format checking, if check fails it's an issue `format` with `info.expectedFormat` set
- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

StringPadding:

- `length: number` - will pad up until this length
- `padWith: string` - the value to pad with

StringFormat:

- `StringFormat.ULID()` - https://github.com/ulid/spec
- `StringFormat.UUID()` - https://www.ietf.org/rfc/rfc4122.txt
- `StringFormat.password(requirements: PasswordRequirements)` - Password format with minimum requirements

Example:

```typescript
const check = isString({
  limitLength: 6,
  padStart: { length: 6, padWith: '-' },
});
```

```typescript
const check = isString({
  format: StringFormat.ULID(),
});
```

```typescript
const check = isString({
  format: StringFormat.password({
    minLength: 10, // default=8
    numberChars: 2, // default=1
    lowerCaseChars: 2, // default=1
    upperCaseChars: 2, // default=1
    specialChars: 0, // default=1
  }),
});
```

```typescript
// change case
import { pascalCase } from 'change-case';

const check = isString({
  transform: pascalCase,
});
```

```typescript
const check = isString({
  maxLength: 10,
  minLength: 8,
  regex: /^[A-Z]+$/,
});
```

```typescript
import validator from 'validator';

const check = isString({
  validator: validator.isEmail,
  validatorOptions: { allow_display_name: true },
});
```

### `isTuple`, `maybeTuple`

Usage:

```typescript
isTuple(options);
maybeTuple(options);
```

Options:

- `validator?: (value: T, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

Example:

```typescript
type MyTuple = [number, string];
const check = isTuple([isNumber({ max: 9, min: 3 }), isString({ regex: /^\w+$/ })]);
```

### `isUrl`, `maybeUrl`, `asUrl`, `maybeAsUrl`

Working with Node's URL object

Usage:

```typescript
isUrl(options);
maybeUrl(options);
asUrl(options);
maybeAsUrl(options);
```

Options:

- `converter?: (value: unknown, options?: any) => T | undefined` - custom converter function, if not defined or `undefined` is returned then built in conversions will be run
- `convertOptions` - options to pass to the _converter_
- `setProtocol?: string` - will coerce the protocol to the given value, if present
- `protocol?: string` - given URL must have this protocol, or it's an error `invalid-protocol`
- `validator?: (value: URL, options?: any) => boolean` - custom validation function; if false is returned it's an error
- `validatorOptions?: any` - options to pass to the _validator_

Example:

```typescript
const check = asUrl({
  protocol: 'https',
});
```

### `isNullable`

Any other check can be wrapped into `isNullable` to accept `null`.

Example:

```typescript
const check = isNullable(isString({ min: 3 }));
```

### `asNullable`

Any other check can be wrapped into `asNullable` to accept `null`.

Options:

- `default` - can be `null` or return type or a function with return type of the wrapped check

Example:

```typescript
const check = asNullable(isString({ min: 3 }));
const check = asNullable(isString({ min: 3 }), { default: null });
const check = asNullable(isString({ min: 3 }), { default: 'text' });
const check = asNullable(isString({ min: 3 }), { default: () => 'text' });
```

## Types

Types can be extracted from a `ValueProcessor` or a `Contract`-like pure object.

```typescript
const sampleContract = {
  maybeString: maybeString(),
  myString: isString(),
  numericString: asString(),
};
const sample = isObject(sampleContract);

// both are same as
export type SampleContract = TypeOf<typeof sample>;
export type Sample = TypeOf<typeof sample>;
// interface Sample {
//   myString: string;
//   maybeString: string | undefined;
//   numericString: string;
// }
```
