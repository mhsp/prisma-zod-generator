# MHSP Prisma Zod Generator

This is a fork of [prisma-zod-generator by Omar Dulaimi](https://github.com/omar-dulaimi/prisma-zod-generator) with some modifications to fit the needs of MHSP.

Automatically generate [Zod](https://github.com/colinhacks/zod) schemas from your [Prisma](https://github.com/prisma/prisma) Schema, and use them to validate your API endpoints or any other use you have. Updates every time `npx prisma generate` runs.

## Table of Contents

- [Supported Prisma Versions](#supported-prisma-versions)
- [Usage](#usage)
- [Customizations](#customizations)
- [Additional Options](#additional-options)

# Supported Prisma Versions

### Prisma 4

- 0.3.0 and higher

### Prisma 2/3

- 0.2.0 and lower

# Usage

1- Add the generator to your Prisma schema

```prisma
generator zod {
  provider = "prisma-zod-generator"
}
```

2- Enable strict mode in `tsconfig` as it is required by Zod, and considered a Typescript best practice

```json
{
  "compilerOptions": {
    "strict": true
  }
}

```

3- Running `npx prisma generate` for the following schema.prisma

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String
  content   String?
  published Boolean  @default(false)
  viewCount Int      @default(0)
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  Int?
  likes     BigInt
}
```

will generate the following files

![Zod Schemas](https://raw.githubusercontent.com/omar-dulaimi/prisma-zod-generator/master/zodSchemas.png)

4- Use generated schemas somewhere in your API logic, like middleware or decorator

```ts
import { PostCreateOneSchema } from './prisma/generated/schemas/createOnePost.schema';

app.post('/blog', async (req, res, next) => {
  const { body } = req;
  await PostCreateOneSchema.parse(body);
});
```

# Customizations

## Skipping entire models

```prisma
/// @@Gen.model(hide: true)
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
}
```

# Additional Options

| Option              | Description                                                                     | Type      | Default       |
|---------------------|---------------------------------------------------------------------------------| --------- | ------------- |
| `output`            | Output directory for the generated zod schemas                                  | `string`  | `./generated` |
| `isGenerateSelect`  | Enables the generation of Select related schemas and the select property        | `boolean` | `false`       |
| `isGenerateInclude` | Enables the generation of Include related schemas and the include property      | `boolean` | `false`       |
| `isGenerateShallow` | Enables the generation of Shallow input types (no creation of linked resources) | `boolean` | `false`       |

Use additional options in the `schema.prisma`

```prisma
generator zod {
  provider          = "prisma-zod-generator"
  output            = "./generated-zod-schemas"
  isGenerateSelect  = true
  isGenerateInclude = true
  isGenerateShallow = true
}
```
