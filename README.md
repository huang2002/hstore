# hstore

> A storage lib.

## TOC

- [Introduction](#introduction)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Links](#links)

## Introduction

`hstore` is a storage lib which has a custom type system and is rather lightweight(<10KB after minification WITHOUT zipping). With your store declared, type checking and conflict detecting will be available out of the box.

## Usage

### npm

1. Use npm to install it as a dependency:

    ```bash
    npm install hstore
    ```

2. Import the exports of this lib:

    ```js
    import { /* ... */ } from "hstore";
    // or
    const { /* ... */ } = require("hstore");
    ```

3. Use them in your code.

### CDN

1. Include one of the following script tags in your HTML file:

    via jsdelivr:

    ```html
    <script type="text/javascript" crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/hstore@latest/dist/hstore.umd.min.js"></script>
    ```

    or via unpkg:

    ```html
    <script type="text/javascript" crossorigin="anonymous" src="https://unpkg.com/hstore@latest/dist/hstore.umd.min.js"></script>
    ```

2. Access the APIs via the `HS` global.

    ```js
    const { /* ... */ } = HS;
    ```

If you want a specified version, just replace `latest` with that in the url. By the way, it is recommended to use a specified version in production.

For more information about these two CDN sites, visit [www.jsdelivr.com](https://www.jsdelivr.com) and [unpkg.com](https://unpkg.com).

## API Reference

(The API reference is written in [TypeScript](https://www.typescriptlang.org/).)

```ts
/**
 * @desc The type of storage-like objects.
 */
interface StorageLike {

    /**
     * @desc Get the item value by giving the key to it.
     */
    getItem(key: string): string | null;

    /**
     * @desc Set the item value by giving the key to it.
     */
    setItem(key: string, value: string): void;

}

/**
 * @desc The type of store options. (These are all
 * partial properties on store instances, so you can
 * refer to the property details for more information.)
 */
interface StoreOptions<T> {
    defaultValue?: T | null;
    type?: Type<T> | null;
    delay?: number;
    storage?: StorageLike;
    lazyLoad?: boolean;
    strictLoad?: boolean;
    secure?: boolean;
    autoFix?: boolean;
    onInvalid?: StoreInvalidCallback<T> | null;
    onConflict?: StoreConflictCallback<T> | null;
    pathSeparator?: string;
}

/**
 * @desc The class for store instances.
 */
class Store<T = unknown> {

    /**
     * @desc The defaults of store options.
     */
    static defaults: StoreOptions<any>;

    /**
     * @desc The constructor which accepts a required name and optional options.
     */
    constructor(name: string, options?: StoreOptions<T>);

    /**
     * @desc The name which is used as the key to the store.
     */
    name: string;

    /**
     * @desc The default value of the store. If this is omitted but the `type` property
     * is given, `type.defaultValue` will be adopted.
     *
     */
    defaultValue: T | null;

    /**
     * @desc The type of the store. (See the `Type` interface and built-in types below.)
     * If this is omitted but the `defaultValue` is given, an inferred type generated by
     * `inferType`(see below) will be adopted.
     */
    type: Type<T> | null;

    /**
     * @desc The delay of saving in milliseconds. (If zero, save synchronously.)
     */
    delay: number;

    /**
     * @desc The storage to use.
     * @default localStorage
     */
    storage: StorageLike;

    /**
     * @desc Whether not to load immediately when the store is being created.
     * @default false
     */
    readonly lazyLoad: boolean;

    /**
     * @desc Whether to do type checking while loading the store from storage.
     * @default true
     */
    strictLoad: boolean;

    /**
     * @desc Whether to check conflicts while saving.
     * @default true
     */
    secure: boolean;

    /**
     * @desc Whether to try to fix invalid value automatically.
     * @default true
     */
    autoFix: boolean;

    /**
     * @desc The invalidation callback. (If not given, errors will be thrown instead.)
     * @param paths The invalid property paths. (Path `[]` means the root.)
     */
    onInvalid: ((this: Store<T>, paths: string[][]) => void) | null;

    /**
     * @desc The conflict callback. (If not given, errors will be thrown instead.)
     * @param newSource The current source in the storage.
     * @param oldSource The copy of the old source in the storage.
     * @example
     * ```js
     * onConflict(newSource, oldSource) {
     *     if (youWantTheNewSource) {
     *         this.load(newSource);
     *     } else {
     *         this.storage.setItem(this.name, oldSource);
     *     }
     * }
     * ```
     */
    onConflict: ((this: Store<T>, newSource: string | null, oldSource: string | null) => void) | null;

    /**
     * @desc The path separator to use.
     * @default '.'
     */
    pathSeparator: string;

    /**
     * @desc Manually save the store. (synchronously)
     * @returns Whether the saving is successful. (`false` if conflict occurs.)
     */
    save(): boolean;

    /**
     * @desc Reset the specific value.
     * @param selector A path string or a path array.
     * @returns Whether the reset is successful. (`false` if the default value can't be found.)
     * @example
     * ```js
     *     store.reset('foo.bar');
     *     store.reset(['foo', 'bar']);
     * ```
     */
    reset(selector: string | string[]): boolean;

    /**
     * @desc Load the store from source.
     * @param source The source string. If omitted, read the source from `storage`.
     * @returns Whether the loading is successful. (`false` on invalidation if it can't fixed.)
     */
    load(source?: string | null): boolean;

    /**
     * @desc Check whether the sources conflict.
     * @returns `true` if conflict; `false` otherwise.
     */
    checkConflict(): boolean;

    /**
     * @desc Get the specific store value by giving a property path string or
     * a path array. Invoke it without arguments to get the whole store value.
     */
    get(path?: string | string[]): unknown;

    /**
     * @desc Set the specific value by giving a path and a new value or
     * an updating callback which accepts the old value and returns a new one.
     * @returns Whether the operation is successful. (`false` on invalidation if it can't fixed.)
     */
    set(path: string | string[], patch: unknown | (this: Store<T>, oldValue: unknown) => unknown): boolean;

}

/**
 * @desc The type of validating results.
 */
interface ValidatingResult {

    /**
     * @desc Whether the value is valid.
     */
    valid: boolean;

    /**
     * @desc The path of invalid properties if there is any.
     */
    paths?: string[][];

}

/**
 * @desc The type of type class instances.
 */
interface Type<T> {

    /**
     * @desc The default value.
     */
    defaultValue: T;

    /**
     * @desc Validate the given result.
     */
    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Turn an object into a map of corresponding type class instances.
 */
type Types<T extends {}> = {
    [K in Extract<keyof T, string>]: Type<T[K]>;
};

/**
 * @desc The type class of any types. (always valid)
 */
class Any implements Type<any> {
    constructor(defaultValue?: any);
    defaultValue: any;
    validate(): ValidatingResult;
}

/**
 * @desc Create an any type instance.
 */
function any(defaultValue?: any): Any;

/**
 * @desc The type class of booleans.
 */
class Boolean implements Type<boolean> {
    defaultValue: boolean;
    constructor(defaultValue?: boolean);
    validate(value: unknown): ValidatingResult;
}

/**
 * @desc Create a boolean type instance.
 */
function boolean(defaultValue?: boolean): Boolean;

/**
 * @desc The type of string options. (See property details.)
 */
interface StringOptions {
    defaultValue?: string;
    minLength?: number;
    maxLength?: number;
    pattern?: RegExp | null;
}

/**
 * @desc The type class of strings.
 */
class String implements Type<string> {

    static defaults: StringOptions;

    constructor(options?: StringOptions);

    defaultValue: string;

    /**
     * @desc The minimum string length.
     * @default 0
     */
    minLength: number;

    /**
     * @desc The maximum string length.
     * @default Infinity
     */
    maxLength: number;

    /**
     * @desc The string pattern. (`null` means any pattern.)
     */
    pattern: RegExp | null;

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a string type instance.
 */
function string(options?: StringOptions): String;

/**
 * @desc The type of number options. (See property details.)
 */
interface NumberOptions {
    defaultValue?: number;
    min?: number;
    max?: number;
    integer?: boolean;
}

/**
 * @desc The type class of numbers.
 */
class Number implements Type<number> {

    static defaults: NumberOptions;

    constructor(options?: NumberOptions);

    defaultValue: number;

    /**
     * @desc The minimum limit.
     * @default -Infinity
     */
    min: number;

    /**
     * @desc The maximum limit.
     * @default Infinity
     */
    max: number;

    /**
     * @desc Whether the number must be an integer.
     * @default false
     */
    integer: boolean;

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a number type instance.
 */
function number(options?: NumberOptions | undefined): Number;

/**
 * @desc The type of nullable options. (See property details.)
 */
interface NullableOptions<T extends null | undefined> {
    defaultValue?: T;
    null?: boolean;
    undefined?: boolean;
}

/**
 * @desc The type class of nullable value.
 */
class Nullable<T extends null | undefined = null | undefined> implements Type<T> {

    static defaults: NullableOptions<null | undefined>;

    constructor(options?: NullableOptions<T>);

    defaultValue: T;

    /**
     * @desc Whether `null` is acceptable.
     * @default true
     */
    null: boolean;

    /**
     * @desc Whether `undefined` is acceptable.
     * @default true
     */
    undefined: boolean;

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a nullable type instance.
 */
function nullable: <T extends null | undefined = null | undefined>
    (options?: NullableOptions<T>): Nullable<T>;

/**
 * @desc The type class of dictionaries.
 */
class Dictionary<T extends {} = any> implements Type<T> {

    /**
     * @desc The property types. (`null` or `undefined` means no limit.)
     * @example
     * ```js
     * dictionary.types = {
     *     foo: HS.string(),
     *     bar: HS.number(),
     * };
     * ```
     */
    readonly types?: Types<T> | null;

    constructor(types?: Types<T> | null);

    /**
     * @desc The default value. (If not provided, one will be created from `types`.)
     */
    defaultValue: T;

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a dictionary type instance.
 */
function dictionary<T extends {} = any>(types?: Types<T> | null): Dictionary<T>;

/**
 * @desc The type of list options. (See property details.)
 */
interface ListOptions<T> {
    defaultValue?: T[];
    type: Type<T>;
}

/**
 * @desc The type class of lists.
 */
class List<T = unknown> implements Type<T[]> {

    static defaults: ListOptions<unknown>;

    constructor(options?: ListOptions<T>);

    defaultValue: T[];

    /**
     * @desc The type of list elements.
     * @default Any
     */
    type: Type<T>;

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a list type instance.
 */
function list<T = unknown>(options?: ListOptions<T>): List<T>;

/**
 * @desc The type of union options. (See property details.)
 */
interface UnionOptions<T> {
    defaultValue?: T;
    types: Type<T>[];
}

/**
 * @desc The type class of union.
 */
class Union<T = unknown> implements Type<T> {

    constructor(options?: UnionOptions<T>);

    /**
     * @desc The default value. (If not provided, one will be created from `types`.)
     */
    defaultValue: T;

    /**
     * @desc The types included in the union.
     */
    types: Type<T>[];

    validate(value: unknown): ValidatingResult;

}

/**
 * @desc Create a union type instance.
 */
function union<T = unknown>(options?: UnionOptions<T>): Union<T>;

/**
 * @desc Get the type nested in the given type by giving the path to it.
 * (Its ancestors must be dictionary types.)
 */
function getTypeByPath(type: Type<unknown>, path: string[]): Type<unknown>;

/**
 * @desc Infer type from the given value.
 */
function inferType(value: unknown): Type<unknown>;

```

## Links

- [Contributing Guide](./CONTRIBUTING.md)
- [Changelog](./CHANGELOG.md)
- [License (MIT)](./LICENSE)
