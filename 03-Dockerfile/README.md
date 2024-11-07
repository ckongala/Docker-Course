In Docker, both `CMD` and `ENTRYPOINT`, as well as `COPY` and `ADD` in a **Dockerfile**, serve important roles, but they have specific use cases and behaviors. Let’s break them down clearly:

### 1. **`CMD` vs `ENTRYPOINT`**

Both `CMD` and `ENTRYPOINT` define the command that will be executed when the Docker container starts. However, they behave differently in terms of flexibility and use cases.

#### **`CMD`**
- **Purpose**: The `CMD` directive is used to specify the **default command** that will be executed when a container starts.
- **Syntax**: It can be defined in 3 forms:
  1. **CMD ["executable", "param1", "param2"]** (exec form) — Preferred because it doesn’t invoke a shell and thus avoids issues with signal handling.
  2. **CMD ["param1", "param2"]** (as an entry point to a specific executable).
  3. **CMD ["executable", "param1", "param2"]** (shell form) — This will invoke a shell (`/bin/sh -c`), which can have issues with signal handling.
  
- **Behavior**:
  - It **can be overridden** when running a container. For example, you can override `CMD` by providing a command at the end of the `docker run` command:
    ```bash
    docker run <image> <override-command>
    ```

- **Use Case**: You use `CMD` when you want to provide a default command, but also allow the flexibility to override it with a different command if necessary.

#### **`ENTRYPOINT`**
- **Purpose**: The `ENTRYPOINT` directive sets the **command that is always executed** when the container starts. It is typically used to define the **main process** of the container, which should always be run.
- **Syntax**:
  1. **ENTRYPOINT ["executable", "param1", "param2"]** (exec form) — Preferred form.
  2. **ENTRYPOINT ["executable", "param1", "param2"]** (shell form) — Uses the shell to invoke the command.
  
- **Behavior**:
  - Unlike `CMD`, the command in `ENTRYPOINT` **cannot be easily overridden** by providing a command at the end of the `docker run` command. 
  - However, if you need to **override** the parameters or append commands, you can use `CMD` along with `ENTRYPOINT`.
  
- **Use Case**: `ENTRYPOINT` is used when you want the container to always execute a specific process (e.g., a server or application). If you need to pass additional arguments to the command, combine `ENTRYPOINT` and `CMD`.

#### **Difference in behavior**:
- If both `ENTRYPOINT` and `CMD` are provided, `CMD` provides **default arguments** to the `ENTRYPOINT` command.
- If `ENTRYPOINT` is not defined, `CMD` will act as the full command to run.
  
**Example:**

```dockerfile
FROM ubuntu

# Entry point is always 'echo'
ENTRYPOINT ["echo"]

# Default arguments are passed from CMD
CMD ["Hello", "World"]

# When running the container, you get: 'Hello World'
```

If you run the container with `docker run <image> Goodbye`, it will output `Goodbye` instead of the default `Hello World` because the `ENTRYPOINT` remains `echo` but the `CMD` is overridden.

---

### 2. **`COPY` vs `ADD`**

Both `COPY` and `ADD` are used to copy files from the host machine into the Docker container, but they have some subtle differences in behavior.

#### **`COPY`**
- **Purpose**: The `COPY` instruction is the **simplest** way to copy files from the **host** to the **container**.
- **Behavior**: It **only copies files and directories**.
- **Use Case**: Use `COPY` when you simply want to copy files or directories from the build context into the image.
  
**Syntax**:
```dockerfile
COPY <src> <dest>
```

- **Example**:
  ```dockerfile
  COPY ./localfile.txt /usr/src/app/
  ```

  This will copy the file `localfile.txt` from the build context into the `/usr/src/app/` directory of the container.

#### **`ADD`**
- **Purpose**: The `ADD` instruction does everything `COPY` does, but with additional functionality.
- **Behavior**:
  - **Extracts tar files**: If you `ADD` a `.tar`, `.tar.gz`, or `.tar.bz2` file, it will be **automatically extracted** into the container's destination directory.
  - **Supports remote URLs**: You can provide a **URL** as the source and `ADD` will download the file into the container.
  
**Syntax**:
```dockerfile
ADD <src> <dest>
```

- **Example**:
  ```dockerfile
  ADD ./localfile.txt /usr/src/app/
  ```

  Like `COPY`, this will copy the `localfile.txt` from the build context to the container.

  However:
  ```dockerfile
  ADD http://example.com/file.tar.gz /usr/src/app/
  ```

  In this case, `ADD` will download the file from the provided URL and store it in `/usr/src/app/`. It will also **automatically extract** the contents if it’s a `.tar.gz`, `.tar.bz2`, or `.tar.xz` file.

#### **Difference in behavior**:
- **`COPY`**: Best used when you just need to copy files from one place to another without additional functionality.
- **`ADD`**: Use it when you need the extra features of downloading from URLs or automatically extracting compressed files.

---

### Key Differences Recap:

| **Feature**        | **`CMD`**                               | **`ENTRYPOINT`**                         | **`COPY`**                                   | **`ADD`**                                         |
|--------------------|-----------------------------------------|------------------------------------------|---------------------------------------------|--------------------------------------------------|
| **Purpose**        | Default command to run in the container | Always runs a specific command          | Copy files from host to container           | Copy files + support URL & auto-extract tar files |
| **Overridable**    | Yes, by providing a new command in `docker run` | No, unless combined with `CMD`           | No additional behavior                     | Automatically extracts tar files, supports URLs  |
| **Use Case**       | Default command or arguments           | Main application or process to run      | Simply copying files                       | Copying files, extracting tar files, downloading |
| **Override Example** | `docker run <image> <new-command>`     | `docker run <image> <new-command>` will not override it unless `CMD` is used | -                                           | -                                                |

---

### Conclusion:
- **Use `CMD`** when you want to define a **default command** that can be easily overridden.
- **Use `ENTRYPOINT`** when you need the container to always run a specific command.
- **Use `COPY`** for a simple file copy operation.
- **Use `ADD`** when you need extra functionality like automatic tar file extraction or the ability to fetch files from remote URLs.
