# Bash


## Loops

### While loop

While true with several commands:
```sh
while true; do echo 3; echo 4; done
```

While true with a command in background:
```sh
while true; do echo 3 & echo 4; done
```

## Pipes

### stdin alternatives

The following commands are equivalent:

```sh
echo "Hello" | cat
cat < <(echo "Hello")
cat <(echo "Hello")
```

The last two are specially useful in context of programs that invoke bash like gdb.

### stdin: First bytes from program and then keyboard

```sh
(echo "Hello"; cat -) | cat
cat <(echo "Hello") - | cat
```
