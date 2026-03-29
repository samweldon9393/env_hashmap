# env-hashmap

A lightweight C library that loads a process's environment variables into a thread-safe hash table for fast O(1) key lookups.

## How it works

When a C program receives its environment as a `char **env` array, each entry is a single string in the form `KEY=VALUE`. This library copies those strings, splits them on `=` using `strtok`, and inserts the resulting key/value pairs into a chained hash table (100 buckets, polynomial rolling hash). All operations are protected by a `pthread` mutex, making the map safe to read from multiple threads.

## API

```c
// Load the process environment into a new heap-allocated hashmap
hashmap *get_env_hm(char **env);

// Look up a variable — returns NULL if not found
const char *hm_get(hashmap *hm, const char *key);

// Insert or overwrite a key/value pair
void hm_put(hashmap *hm, char *key, char *value);

// Free all memory and destroy the mutex
void destroymap(hashmap *hm);
```

## Building

```bash
make          # builds env.a (static library) and env_driver (demo binary)
make clean    # remove build artifacts
```

Requires GCC and pthreads (standard on Linux/macOS).

## Usage example

```c
#include "env.h"

int main(int argc, char **argv, char **env) {
    hashmap *hm = get_env_hm(env);

    printf("Shell: %s\n", hm_get(hm, "SHELL"));
    printf("User:  %s\n", hm_get(hm, "LOGNAME"));

    destroymap(hm);
    return 0;
}
```

Link against the static library when compiling your own program:

```bash
gcc -o myprogram myprogram.c env.a -lpthread
```

## Files

| File | Description |
|------|-------------|
| `env.h` | Public header — types and function declarations |
| `env.c` | Hash table implementation |
| `env_driver.c` | Demo program |
| `Makefile` | Build rules |
