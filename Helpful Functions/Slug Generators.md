## Node Crypto
Generate a unique slug using node crypto module
```ts
import { randomUUID } from "node:crypto"

export function slugGenerator(text: string) {
  return `${text
    .toLowerCase()
    .replace(/[+%()=/|!@#$%^&*~:"',.`]/g, "")
    .replace(/\s+/g, "-")}-${randomUUID().substring(0, 6)}`
}
```
