---
layout: post
title:  "Jsonize with custom field name in TypeScript"
author: "Tuna"
comments: false
category: Coding
tags: typescript, javascript, json
image: 
excerpt: |
  A step by step discovery of how to customize JSON serialization and deserialization in TypeScript.
excerpt_separator: <!--more-->
lang: en
sticky: false
hidden: false
draft: false
---

## Introduction


As some of you may know, I created an ASCII drawing app called [MonoSketch][git-monosketch]. Initially developed in Kotlin/JS, the app’s architecture served us well for the early iterations. However, the UI became a bottleneck whenever I wanted to add more features. This led to the decision to rewrite MonoSketch in TypeScript.

Rewriting in TypeScript, with [Svelte][svelte] for the UI, offered several benefits: it made UI development easier, better integration with FrontEnd web technologies, and made adding new features simpler. However, one key challenge I faced was handling JSON serialization and deserialization in TypeScript, which is used for data persistence.

MonoSketch saves drawings as a custom format `.mono` file, which is essentially a JSON file with a custom structure:
- Field names are obfuscated for shorter file sizes (e.g., `text` is serialized as `t`).
- Some value types are serialized in a custom format (e.g., a `Point(left, right)` object is serialized as a single string in the format `"{left}|{right}"`).

These customizations were straightforward to implement in Kotlin, thanks to [**Kotlin’s serialization library**][kotlin-serialization]. However, TypeScript lacks a built-in mechanism for customizing JSON field names and serialization rules. Although there is a library called [class-transformer][class-transformer] that offers similar features to Kotlin's serialization, I found it too complex for MonoSketch and was lazy to learn it. This led me to develop a custom solution for JSON serialization and deserialization in TypeScript, which I’ll share in this post.

#### Key Requirements
First, let’s dive into the requirements that shaped the design of the solution.

When designing a solution for MonoSketch, I identified three primary requirements that the system needed to fulfill:

1. Custom Field Names
2. Custom Serialization and Deserialization of Values
3. Support for Nested Objects


Let's address each requirement in detail.

## Custom Field Names

This requirement is simple: we need to map the field names in the TypeScript class to the field names in the JSON file. For example, the extra value of a text shape looks like this:
```typescript
extra = {
  textHorizontalAlign: 0,
  textVerticalAlign: 0,
  // ...
}
```
will be serialized as:
```jsonc
{
  "tha": 0,
  "tva": 0,
  //...
}
```

A simple solution is we will add 2 methods like these:
```typescript
class TextShapeExtra {
  textHorizontalAlign: number = 0;
  textVerticalAlign: number = 0;

  toJson(): any {
    return {
      tha: this.textHorizontalAlign,
      tva: this.textVerticalAlign
    }
  }

  static fromJson(json: any): TextShapeExtra {
    const extra = new TextShapeExtra();
    extra.textHorizontalAlign = json.tha;
    extra.textVerticalAlign = json.tva;
    return extra;
  }
}
```

This solution is simple and straightforward, but it has a few drawbacks:

- It requires manual mapping of each field, which can be tedious for classes with many fields.
- It is error-prone, as a typo in the field name can lead to runtime errors.
- It is not scalable, as adding or removing fields requires updating the `toJson()` and `fromJson()` methods.
- It is not reusable, as the mapping logic is tightly coupled with the class implementation.

**Solution: Decorators**

In Kotlin, we can use the [`@SerialName`][kotlin-serial-name] annotation to customize field names. So, the solution I came up with for TypeScript was to create a decorator that mimics this behavior. The decorator would allow us to specify custom field names in the class definition, which would be used during serialization and deserialization.


```typescript
class TextShapeExtra {
  @SerialName("tha")
  textHorizontalAlign: number = 0;
  @SerialName("tva")
  textVerticalAlign: number = 0;
}
```
`@SerialName` decorator is defined as follows:
```typescript
function SerialName(name: string) {
  return function(target: any, propertyKey: string | symbol) {
    if (!target.constructor.serialNames) {
      target.constructor.serialNames = {};
    }
    target.constructor.serialNames[propertyKey] = name;
  };
}
```
The `SerialName` decorator stores the custom field names in a static property called `serialNames` within the class. For the `TextShapeExtra` class, the `serialNames` property would look like this:
```typescript
TextShapeExtra.serialNames = {
  textHorizontalAlign: "tha",
  textVerticalAlign: "tva"
}
```

This property will be used to map the field names during serialization and deserialization.

With this decorator, we can now serialize and deserialize objects with custom field names without having to write custom mapping logic for each class. Of course, `toJson` and `fromJson` methods haven't yet been implemented, but we will cover them in the later section.

## Custom Serialization and Deserialization of Values

The second requirement is to support custom serialization and deserialization of values. In the `Point` sample above, we need to serialize a `Point(left, right)` object as a string in the format `"{left}|{right}"`. This is a common requirement in many serialization libraries, but TypeScript does not provide built-in support for this feature.

Let's expand the `TextShapeExtra` class to include a `Point` object:

```typescript
class TextShapeExtra {
  @SerialName("tha")
  textHorizontalAlign: number = 0;
  @SerialName("tva")
  textVerticalAlign: number = 0;
  @SerialName("p")
  position: Point = new Point(0, 0);
}
```

After serialization, the `TextShapeExtra` object should look like this:
```jsonc
{
  "tha": 0,
  "tva": 0,
  "p": "0|0"
}
```

To achieve this, we apply the same approach as we did for custom field names: create a decorator that allows us to specify custom serialization and deserialization logic for a field.

```typescript
function Serialize(serializer: (value: any) => any, deserializer: (value: any) => any) {
  return function(target: any, propertyKey: string | symbol) {
    if (!target.constructor.serializers) {
      target.constructor.serializers = {};
      target.constructor.deserializers = {};
    }
    target.constructor.serializers[propertyKey] = serializer;
    target.constructor.deserializers[propertyKey] = deserializer;
  };
}
```

Sample usage:
```typescript
class TextShapeExtra {
  @SerialName("tha")
  textHorizontalAlign: number = 0;
  @SerialName("tva")
  textVerticalAlign: number = 0;
  @SerialName("p")
  @Serialize(
    (value: Point) => `${value.left}|${value.right}`,
    (value: string) => {
      const (left, top) = value.split("|");
      return new Point(parseInt(left), parseInt(top));
    }
  )
  position: Point = new Point(0, 0);
}
```

Similar to the `SerialName` decorator, the `Serialize` decorator stores the serialization and deserialization logic in the `serializers` and `deserializers` properties of the class.

With this decorator, we can now serialize and deserialize objects with custom serialization and deserialization logic without having to write custom mapping logic for each class.

## Support for Nested Objects

Finally, we need to support nested objects in the serialization and deserialization process. This means that if an object contains another object, the nested object should also be serialized and deserialized according to the custom field names and serialization logic.

The text shape extra belongs to a text shape:
```typescript
class TextShape {
  @SerialName("i")
  id: string = "";
  @SerialName("t")
  text: string = "";
  @SerialName("e")
  extra: TextShapeExtra = new TextShapeExtra();
}
```

After serialization, the `TextShape` object should look like this:
```jsonc
{
  "i": "",
  "t": "",
  "e": {
    "tha": 0,
    "tva": 0,
    "p": "0|0"
  }
}
```

To handle this, we have two options:
1. Use the `Serialize` decorator to specify custom serialization and deserialization logic for nested objects.
2. Recursively serialize and deserialize nested objects.

The first option is straightforward and allows us to customize the serialization and deserialization logic for nested objects. However, it requires us to manually specify the serialization and deserialization logic for each nested object, which can be tedious and error-prone due to repeated code.

The second option is more complex but offers a more generic solution. We can recursively serialize and deserialize nested objects by checking if a field is an object and applying the serialization and deserialization logic accordingly.

To do this, we need the 3rd decorator, but this time, not for the field, but for the class itself. The code is a bit long:

```typescript
function Jsonizable(constructor: Function) {
  if (!constructor.serializers) {
    constructor.serializers = {};
  }
  if (!constructor.serialNames) {
    constructor.serialNames = {};
  }

  const serialNames = constructor.serialNames;
  const serializers = constructor.serializers;
  const deserializers = constructor.deserializers;

  constructor.prototype.toJson = function() {
    const json: any = {};
    const instance = this;
    for (const key in instance) {
      if (!instance.hasOwnProperty(key)) {
        continue;
      }
      // If the key is not defined in serialNames, use the key itself
      const serializedKey = serialNames[key] ?? key;

      // 1st: Check if the field has a serializer
      // 2nd: Check if the field has a toJson method
      // 3rd: Use the value directly
      if (serializers[key]) {
        json[serializedKey] = serializers[key](instance[key]);
      } else if (instance[key].toJson) {
        json[serializedKey] = instance[key].toJson();
      } else {
        json[serializedKey] = instance[key];
      }
    }
    return json;
  };

  constructor.fromJson = function(data: any) {
    const instance = new constructor();
    for (const key of Object.keys(instance)) {
      // If the key is not defined in serialNames, use the key itself
      const serializedKey = serialNames[key] ?? key;
      const value = data[serializedKey];
      if (value === undefined) {
        continue;
      }

      const field = instance[key];
      // 1st: Check if the field has a deserializer
      // 2nd: Check if the field has a fromJson method
      // 3rd: Use the value directly
      if (deserializers[key]) {
        instance[key] = deserializers[key](value);
      } else if (field && field.constructor && field.constructor.fromJson) {
        instance[key] = field.constructor.fromJson(value);
      } else {
        instance[key] = value;
      }
    }

    return instance;
  };
}
```

This decorator adds `toJson` and `fromJson` methods to the class and class's static api respectively, which recursively serializes and deserializes the object and its nested objects. 

- `toJson` method iterates over each field in the object, checks if the field has a custom serializer, and applies the serialization logic accordingly. If the field is an object, it recursively calls the `toJson` method of the nested object. 
- `fromJson` method does the reverse, iterating over each field in the JSON data, checking if the field has a custom deserializer, and applying the deserialization logic accordingly. If the field is an object, it recursively calls the `fromJson` method of the nested object.

With this decorator, we can now serialize and deserialize objects with nested objects without having to write custom serialization and deserialization logic for each nested object.

> **Note**: This solution does not automatically support arrays or polymorphism. To handle these cases, we can use the `@Serialize` decorator to specify custom serialization and deserialization logic for arrays and polymorphic objects.

## Combining everything together

**Definition side:**

```typescript
@Jsonizable
class TextShapeExtra {
  @SerialName("tha")
  textHorizontalAlign: number = 0;
  @SerialName("tva")
  textVerticalAlign: number = 0;
  @SerialName("p")
  @Serialize(
    (value: Point) => `${value.left}|${value.right}`,
    (value: string) => {
      const (left, top) = value.split("|");
      return new Point(parseInt(left), parseInt(top));
    }
  )
  position: Point = new Point(0, 0);
}

@Jsonizable
class TextShape {
  @SerialName("i")
  id: string = "";
  @SerialName("t")
  text: string = "";
  @SerialName("e")
  extra: TextShapeExtra = new TextShapeExtra();
}
```

**Caller side:**

```typescript
const textShape = new TextShape(...);
const json = textShape.toJson();

const textShape2 = TextShape.fromJson(json);
```

**Setting up the Project**

One thing I haven't mentioned is that decorators are not natively supported by TypeScript and are still considered an [experimental feature][foot-note-ts-decorator]. To enable their usage, the `experimentalDecorators` flag must be set to true in the `tsconfig.json` file. This configuration is similar to what [`class-transformer` requires][class-transformer-documentation]:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

## End Notes

In MonoSketch Kotlin version, except for [Compose HTML][compose-html] for UI, the only library is the serialization. Now, after implementing the custom solution in TypeScript, MonoSketch TypeScript version uses Svelte as the only 3rd party library. How crazy I am!

[git-monosketch]: https://github.com/tuanchauict/MonoSketch
[kotlin-serialization]: https://kotlinlang.org/docs/serialization.html
[class-transformer]: https://github.com/typestack/class-transformer
[svelte]: https://svelte.dev/
[kotlin-serial-name]: https://kotlinlang.org/api/kotlinx.serialization/kotlinx-serialization-core/kotlinx.serialization/-serial-name/
[compose-html]: https://github.com/JetBrains/compose-multiplatform?tab=readme-ov-file#compose-html
[foot-note-ts-decorator]: https://www.typescriptlang.org/tsconfig/#experimentalDecorators
[class-transformer-documentation]: https://github.com/typestack/class-transformer/blob/a073b5ea218dd4da9325fe980f15c1538980500e/docs/pages/01-getting-started.md#installation
