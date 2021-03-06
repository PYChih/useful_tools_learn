TARGET=hello
SRCS=hello.c
CFLAGS=-g -Wall -O3
CC=c99

$(TARGET): $(SRCS)
	$(CC) -o $@ $(CFLAGS) $^

.PHONY: clean
clean:
	rm -f $(TARGET)