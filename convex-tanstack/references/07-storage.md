# Part 7: File Storage

### Upload Pattern

```typescript
// Generate upload URL
export const generateUploadUrl = mutation({
  args: {},
  returns: v.string(),
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl()
  },
})

// Store file reference after upload
export const saveFile = mutation({
  args: {
    storageId: v.id("_storage"),
    fileName: v.string(),
  },
  returns: v.id("files"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("files", {
      storageId: args.storageId,
      fileName: args.fileName,
      uploadedAt: Date.now(),
    })
  },
})
```

### Get File URL

```typescript
export const getFileUrl = query({
  args: { storageId: v.id("_storage") },
  returns: v.union(v.string(), v.null()),
  handler: async (ctx, args) => {
    // Returns signed URL or null if file doesn't exist
    return await ctx.storage.getUrl(args.storageId)
  },
})
```

### Get File Metadata

```typescript
export const getFileMetadata = query({
  args: { storageId: v.id("_storage") },
  returns: v.union(
    v.object({
      _id: v.id("_storage"),
      _creationTime: v.number(),
      contentType: v.optional(v.string()),
      sha256: v.string(),
      size: v.number(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    // Query the _storage system table
    const metadata = await ctx.db.system.get(args.storageId)
    return metadata
  },
})
```

### Delete File

```typescript
export const deleteFile = mutation({
  args: { storageId: v.id("_storage") },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.storage.delete(args.storageId)
    return null
  },
})
```

### Frontend Upload Component

```typescript
import { useMutation } from 'convex/react'
import { api } from '~/convex/_generated/api'

function FileUpload() {
  const generateUploadUrl = useMutation(api.files.generateUploadUrl)
  const saveFile = useMutation(api.files.saveFile)

  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0]
    if (!file) return

    // 1. Get upload URL
    const uploadUrl = await generateUploadUrl()

    // 2. Upload file to Convex storage
    const result = await fetch(uploadUrl, {
      method: "POST",
      headers: { "Content-Type": file.type },
      body: file,
    })
    const { storageId } = await result.json()

    // 3. Save file reference
    await saveFile({ storageId, fileName: file.name })
  }

  return <input type="file" onChange={handleUpload} />
}
```

---

