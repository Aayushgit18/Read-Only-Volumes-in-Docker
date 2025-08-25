# Read-Only-Volumes-in-Docker

## 🧩 What’s the Problem?

By default, when you bind mount your **local project folder** into a container using:

```bash
-v "$(pwd)":/app
```

* It **overwrites** `/app` in the container
* It's **read-write** by default, so:

  * The container can **read** and **write** to this folder
  * That means your app could accidentally **change local source code** from inside the container

That’s **not safe** or intended for most dev setups.

---

## ✅ Solution: Use a Read-Only Bind Mount

To **protect your source code**, you can mark the bind mount as **read-only**:

```bash
-v "$(pwd)":/app:ro
```

This tells Docker:

> “Mount my local folder into the container, but only allow reading from it — not writing.”

This ensures:

* Local file changes still reflect in the container ✅
* The container **can’t write back** to the source files ❌

---

## ⚠️ But... What About Writable Folders Inside `/app`?

In your project, some folders **need to be writable** by the container:

| Folder              | Why it needs write access                       |
| ------------------- | ----------------------------------------------- |
| `/app/feedback`     | Stores submitted feedback (user-generated data) |
| `/app/temp`         | Temporary file storage during processing        |
| `/app/node_modules` | Must exist from image; shouldn’t be overwritten |

If you just mount the full project folder as **read-only**, you would block writing to these folders — and your app would break.

---

## 🛠️ Fix: Override Read-Only Bind Mount with Specific Writable Volumes

Docker allows **more specific volume paths** to override less specific ones.

### Final `docker run` command:

```bash
docker run -d -p 3000:80 --name feedback-app \
-v feedback:/app/feedback \            # Named volume (writable) for persistent feedback
-v /app/temp \                         # Anonymous volume (writable) for temporary files
-v /app/node_modules \                 # Anonymous volume to preserve node_modules
-v "$(pwd)":/app:ro \                  # Bind mount (read-only) for source code
feedback-app
```

> 🔁 Docker gives priority to the more specific path (`/app/temp`, `/app/feedback`, etc.) — so these remain writable even though `/app` is mounted read-only.

---

## 🔐 Benefits of This Pattern

| ✅ Feature                           | Description                                      |
| ----------------------------------- | ------------------------------------------------ |
| Protects source code                | Prevents accidental overwriting by app logic     |
| Still allows code auto-reload       | Source files still sync into container           |
| Maintains writable app data folders | App can write to `/app/feedback` and `/app/temp` |
| Efficient volume use                | Anonymous volumes reduce container storage load  |

---

## 🔄 TL;DR: Volume Strategy for Local Dev

| Folder              | Mount Type       | Flags        | Purpose                                 |
| ------------------- | ---------------- | ------------ | --------------------------------------- |
| `/app`              | Bind Mount       | `:ro`        | Read-only access to local source files  |
| `/app/feedback`     | Named Volume     | (default RW) | Persistent data from user submissions   |
| `/app/temp`         | Anonymous Volume | (default RW) | Temporary processing files              |
| `/app/node_modules` | Anonymous Volume | (default RW) | Keeps dependencies inside the container |

---

## 📦 Quick Reference: Volume Syntax Recap

```bash
# Anonymous Volume
-v /app/temp

# Named Volume
-v feedback:/app/feedback

# Bind Mount (Read-only)
-v "$(pwd)":/app:ro
