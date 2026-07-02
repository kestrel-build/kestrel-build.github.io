# Control Flow

## If / else

Parentheses around the condition are required:

```kestrel
if (x > 0) {
    printf("positive\n")
} else if (x < 0) {
    printf("negative\n")
} else {
    printf("zero\n")
}
```

## Ternary

```kestrel
int32 abs_x = if (x >= 0) x else -x
```

## For loops

```kestrel
// inclusive range (0 through 9)
for (i in 0..9) {
    printf("{i}\n")
}

// exclusive range (0 through 9, not 10)
for (i in 0..<10) {
    printf("{i}\n")
}
```

## While

```kestrel
while (counter < 100) {
    counter = counter + 1
}
```

## Do-while

```kestrel
do {
    counter = counter + 1
} while (counter < 100)
```

## Repeat

```kestrel
repeat (5) {
    printf("hello\n")
}
```

## Match

```kestrel
match (status) {
    case 200 {
        printf("OK\n")
    }
    case 404 {
        printf("Not found\n")
    }
    case 500..599 {
        printf("Server error\n")
    }
    case _ {
        printf("Unknown: {status}\n")
    }
}
```

## Break and continue

```kestrel
for (i in 0..<100) {
    if (i == 50) {
        break
    }
    if (i % 2 == 0) {
        continue
    }
    printf("{i}\n")
}
```
